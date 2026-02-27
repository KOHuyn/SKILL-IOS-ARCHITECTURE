# MVI Pattern

Model-View-Intent with unidirectional data flow. For simpler MVVM, see `viewmodel-pattern.md`.

## Data Flow

```
User Action → Intent → Reducer → UiState → View re-renders
```

## Location
`Presentation/{Feature}/{Feature}Store.swift`

## UiState (Immutable)

```swift
struct FeatureUiState: Equatable {
    var items: [Item] = []
    var selectedItem: Item?
    var isLoading: Bool = false
    var error: String?

    var hasItems: Bool { !items.isEmpty }
    var showError: Bool { error != nil }
}
```

## Intent (User Actions)

```swift
enum FeatureIntent {
    case loadItems
    case loadItemsSuccess([Item])
    case loadItemsFailure(String)
    case selectItem(Item)
    case deleteItem(id: String)
    case clearError
}
```

## Reducer (Pure Function)

```swift
func featureReducer(state: FeatureUiState, intent: FeatureIntent) -> FeatureUiState {
    var newState = state
    switch intent {
    case .loadItems:
        newState.isLoading = true
        newState.error = nil
    case .loadItemsSuccess(let items):
        newState.items = items
        newState.isLoading = false
    case .loadItemsFailure(let error):
        newState.error = error
        newState.isLoading = false
    case .selectItem(let item):
        newState.selectedItem = item
    case .deleteItem(let id):
        newState.items.removeAll { $0.id == id }
        if newState.selectedItem?.id == id {
            newState.selectedItem = nil
        }
    case .clearError:
        newState.error = nil
    }
    return newState
}
```

## Store (Coordinator)

```swift
@MainActor
final class FeatureStore: ObservableObject {
    @Published private(set) var uiState = FeatureUiState()

    private let getItemsUseCase: GetItemsUseCaseProtocol
    private let deleteItemUseCase: DeleteItemUseCaseProtocol

    init(
        getItemsUseCase: GetItemsUseCaseProtocol,
        deleteItemUseCase: DeleteItemUseCaseProtocol
    ) {
        self.getItemsUseCase = getItemsUseCase
        self.deleteItemUseCase = deleteItemUseCase
    }

    func send(_ intent: FeatureIntent) {
        uiState = featureReducer(state: uiState, intent: intent)
        handleSideEffects(intent)
    }

    private func handleSideEffects(_ intent: FeatureIntent) {
        switch intent {
        case .loadItems:
            Task {
                do {
                    let items = try await getItemsUseCase.execute()
                    send(.loadItemsSuccess(items))
                } catch {
                    send(.loadItemsFailure(error.localizedDescription))
                }
            }
        case .deleteItem(let id):
            Task { try? await deleteItemUseCase.execute(id: id) }
        default: break
        }
    }
}
```

## View

```swift
struct FeatureView: View {
    @StateObject private var store: FeatureStore

    init(store: FeatureStore) {
        _store = StateObject(wrappedValue: store)
    }

    var body: some View {
        Group {
            if store.uiState.isLoading {
                ProgressView()
            } else {
                List(store.uiState.items) { item in
                    Text(item.name)
                        .onTapGesture { store.send(.selectItem(item)) }
                }
            }
        }
        .alert("Error", isPresented: .constant(store.uiState.showError)) {
            Button("OK") { store.send(.clearError) }
        } message: {
            Text(store.uiState.error ?? "")
        }
        .task { store.send(.loadItems) }
    }
}
```

## Rules

1. **Pure Reducer** - Same state + intent = same result
2. **Single UiState** - All UI state in one Equatable struct
3. **Intent Pipeline** - All actions via `send()`
4. **Side Effects Isolation** - Async in `handleSideEffects` only
5. **Use Case Injection** - Never inject repositories directly
