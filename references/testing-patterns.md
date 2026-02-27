# Testing Patterns

Unit testing for MVI and MVVM architectures.

## Reducer Testing (MVI)

Pure reducers = easiest to test.

```swift
import XCTest
@testable import YourApp

final class FeatureReducerTests: XCTestCase {

    func testLoadItems_setsLoadingTrue() {
        let initial = FeatureState()
        let result = featureReducer(state: initial, intent: .loadItems)

        XCTAssertTrue(result.isLoading)
        XCTAssertNil(result.error)
    }

    func testLoadItemsSuccess_populatesItems() {
        let initial = FeatureState(isLoading: true)
        let items = [Item(id: "1", name: "Test")]

        let result = featureReducer(state: initial, intent: .loadItemsSuccess(items))

        XCTAssertEqual(result.items, items)
        XCTAssertFalse(result.isLoading)
    }

    func testDeleteItem_removesFromList() {
        let items = [Item(id: "1", name: "A"), Item(id: "2", name: "B")]
        let initial = FeatureState(items: items)

        let result = featureReducer(state: initial, intent: .deleteItem(id: "1"))

        XCTAssertEqual(result.items.count, 1)
    }
}
```

## Store/ViewModel Testing

Mock use cases, verify state changes.

```swift
@MainActor
final class FeatureStoreTests: XCTestCase {

    func testLoadItems_success() async {
        let mock = MockUseCase()
        mock.itemsToReturn = [Item(id: "1", name: "Test")]
        let store = FeatureStore(useCase: mock)

        store.send(.loadItems)
        try? await Task.sleep(nanoseconds: 100_000_000)

        XCTAssertEqual(store.state.items.count, 1)
        XCTAssertFalse(store.state.isLoading)
    }

    func testLoadItems_failure() async {
        let mock = MockUseCase()
        mock.shouldFail = true
        let store = FeatureStore(useCase: mock)

        store.send(.loadItems)
        try? await Task.sleep(nanoseconds: 100_000_000)

        XCTAssertNotNil(store.state.error)
    }
}
```

## Mock Pattern

```swift
final class MockUseCase: FeatureUseCaseProtocol {
    var itemsToReturn: [Item] = []
    var shouldFail = false

    func execute() async throws -> [Item] {
        if shouldFail { throw NSError(domain: "Test", code: 1) }
        return itemsToReturn
    }
}
```

## State Sequence Testing

```swift
func testIntentSequence() {
    var state = FeatureState()

    state = featureReducer(state: state, intent: .loadItems)
    XCTAssertTrue(state.isLoading)

    let items = [Item(id: "1", name: "A")]
    state = featureReducer(state: state, intent: .loadItemsSuccess(items))
    XCTAssertEqual(state.items.count, 1)

    state = featureReducer(state: state, intent: .deleteItem(id: "1"))
    XCTAssertTrue(state.items.isEmpty)
}
```

## Test Naming

```
test{Action}_{expectedBehavior}
test{Scenario}_{expectedOutcome}
```

Examples:
- `testLoadItems_setsLoadingTrue`
- `testDeleteLastItem_clearsSelection`
- `testNetworkError_showsErrorState`
