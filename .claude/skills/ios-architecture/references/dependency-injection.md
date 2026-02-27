# Dependency Injection

Centralized DI Container for all dependencies.

## Location
`Core/DI/DependencyContainer.swift`

## Structure
```swift
import Foundation
import SwiftData
import Combine

final class DependencyContainer: ObservableObject {
    // MARK: - Singleton
    static let shared = DependencyContainer()

    // MARK: - Infrastructure
    private(set) var modelContainer: ModelContainer!

    // MARK: - Repositories
    private(set) var bookRepository: BookRepositoryProtocol!
    private(set) var ttsRepository: TTSRepositoryProtocol!
    private(set) var audioRepository: AudioRepositoryProtocol!

    // MARK: - Use Cases (Book)
    private(set) var importBookUseCase: ImportBookUseCaseProtocol!
    private(set) var getBooksUseCase: GetBooksUseCaseProtocol!
    private(set) var getBookUseCase: GetBookUseCaseProtocol!
    private(set) var deleteBookUseCase: DeleteBookUseCaseProtocol!

    // MARK: - Use Cases (Reader)
    private(set) var extractPagesUseCase: ExtractPagesUseCaseProtocol!
    private(set) var saveProgressUseCase: SaveProgressUseCaseProtocol!

    // MARK: - Use Cases (TTS)
    private(set) var speakTextUseCase: SpeakTextUseCaseProtocol!
    private(set) var getVoicesUseCase: GetVoicesUseCaseProtocol!

    // MARK: - Private Init
    private init() {}

    // MARK: - Setup
    func setup(with modelContainer: ModelContainer) {
        self.modelContainer = modelContainer
        setupRepositories()
        setupUseCases()
    }

    private func setupRepositories() {
        let modelContext = modelContainer.mainContext

        bookRepository = BookRepository(modelContext: modelContext)
        ttsRepository = TTSEngineManager.shared
        audioRepository = AudioRepository()
    }

    private func setupUseCases() {
        // Book
        importBookUseCase = ImportBookUseCase(bookRepository: bookRepository)
        getBooksUseCase = GetBooksUseCase(bookRepository: bookRepository)
        getBookUseCase = GetBookUseCase(bookRepository: bookRepository)
        deleteBookUseCase = DeleteBookUseCase(bookRepository: bookRepository)

        // Reader
        extractPagesUseCase = ExtractPagesUseCase(bookRepository: bookRepository)
        saveProgressUseCase = SaveProgressUseCase(bookRepository: bookRepository)

        // TTS
        speakTextUseCase = SpeakTextUseCase(ttsRepository: ttsRepository)
        getVoicesUseCase = GetVoicesUseCase(ttsRepository: ttsRepository)
    }

    // MARK: - ViewModel Factory Methods
    func makeLibraryViewModel() -> LibraryViewModel {
        LibraryViewModel(
            getBooksUseCase: getBooksUseCase,
            importBookUseCase: importBookUseCase,
            deleteBookUseCase: deleteBookUseCase
        )
    }

    func makeReaderViewModel(book: BookEntity) -> ReaderViewModel {
        ReaderViewModel(
            book: book,
            extractPagesUseCase: extractPagesUseCase,
            saveProgressUseCase: saveProgressUseCase,
            speakTextUseCase: speakTextUseCase,
            getVoicesUseCase: getVoicesUseCase
        )
    }

    func makeSettingsViewModel() -> SettingsViewModel {
        SettingsViewModel(
            getVoicesUseCase: getVoicesUseCase
        )
    }
}
```

## App Entry Point
```swift
// BookcastApp.swift
import SwiftUI
import SwiftData

@main
struct BookcastApp: App {
    let modelContainer: ModelContainer

    init() {
        do {
            let schema = Schema([Book.self, ReadingProgress.self])
            let config = ModelConfiguration(schema: schema)
            modelContainer = try ModelContainer(for: schema, configurations: [config])

            // Setup DI Container
            DependencyContainer.shared.setup(with: modelContainer)
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(modelContainer)
    }
}
```

## Usage in Views
```swift
// Option 1: Factory method
struct LibraryView: View {
    @StateObject private var viewModel = DependencyContainer.shared.makeLibraryViewModel()
}

// Option 2: Environment injection
struct ContentView: View {
    @StateObject private var container = DependencyContainer.shared

    var body: some View {
        LibraryView()
            .environmentObject(container)
    }
}
```

## Testing with Mock Dependencies
```swift
// In tests, create container with mocks
class MockBookRepository: BookRepositoryProtocol { ... }

let mockRepo = MockBookRepository()
let useCase = GetBooksUseCase(bookRepository: mockRepo)
let viewModel = LibraryViewModel(
    getBooksUseCase: useCase,
    importBookUseCase: MockImportUseCase(),
    deleteBookUseCase: MockDeleteUseCase()
)
```
