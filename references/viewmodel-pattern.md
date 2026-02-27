# ViewModel Pattern

MVVM with UiState pattern. For MVI unidirectional flow, see `mvi-pattern.md`.

## Location
`Presentation/{Feature}/{Feature}ViewModel.swift`

## UiState

```swift
struct FeatureUiState: Equatable {
    var items: [Item] = []
    var isLoading: Bool = false
    var error: String?
    var selectedItem: Item?

    var hasItems: Bool { !items.isEmpty }
    var showError: Bool { error != nil }
}
```

## ViewModel Structure

```swift
@MainActor
final class FeatureViewModel: ObservableObject {
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

    func loadItems() async {
        uiState.isLoading = true
        uiState.error = nil

        do {
            uiState.items = try await getItemsUseCase.execute()
        } catch {
            uiState.error = error.localizedDescription
        }

        uiState.isLoading = false
    }

    func selectItem(_ item: Item) {
        uiState.selectedItem = item
    }

    func deleteItem(_ item: Item) async {
        do {
            try await deleteItemUseCase.execute(id: item.id)
            uiState.items.removeAll { $0.id == item.id }
            if uiState.selectedItem?.id == item.id {
                uiState.selectedItem = nil
            }
        } catch {
            uiState.error = error.localizedDescription
        }
    }

    func clearError() {
        uiState.error = nil
    }
}
```

## View Binding

```swift
struct FeatureView: View {
    @StateObject private var viewModel: FeatureViewModel

    init(viewModel: FeatureViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }

    var body: some View {
        Group {
            if viewModel.uiState.isLoading {
                ProgressView()
            } else {
                List(viewModel.uiState.items) { item in
                    Text(item.name)
                        .onTapGesture { viewModel.selectItem(item) }
                }
            }
        }
        .alert("Error", isPresented: .constant(viewModel.uiState.showError)) {
            Button("OK") { viewModel.clearError() }
        } message: {
            Text(viewModel.uiState.error ?? "")
        }
        .task { await viewModel.loadItems() }
    }
}
```

## Rules

1. **Single UiState** - All UI state in one `@Published` struct
2. **Equatable UiState** - For efficient SwiftUI updates
3. **@MainActor** - All state on main thread
4. **private(set)** - Read-only from View
5. **Use Case injection** - Never inject repositories directly
