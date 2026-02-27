# Repository Pattern

Repository protocols in Domain, implementations in Data.

## Protocol Location
`Domain/Repositories/{Name}RepositoryProtocol.swift`

## Implementation Location
`Data/Repositories/{Name}Repository.swift`

## Protocol Structure
```swift
import Foundation

protocol BookRepositoryProtocol {
    // MARK: - CRUD Operations
    func getAllBooks() async throws -> [BookEntity]
    func getBook(by id: UUID) async throws -> BookEntity?
    func saveBook(_ book: BookEntity) async throws
    func updateBook(_ book: BookEntity) async throws
    func deleteBook(by id: UUID) async throws
    func searchBooks(query: String) async throws -> [BookEntity]

    // MARK: - Business Operations
    func updateProgress(bookId: UUID, progress: ReadingProgressEntity) async throws

    // MARK: - Document Operations
    func extractPages(from documentData: Data, type: DocumentType) async -> [PageEntity]
    func extractMetadata(from documentData: Data, type: DocumentType) -> DocumentMetadata
    func generateCoverImage(from documentData: Data, type: DocumentType, size: CGSize) -> Data?
}
```

## Implementation Structure
```swift
import Foundation
import SwiftData  // Framework dependency OK in Data layer

final class BookRepository: BookRepositoryProtocol {
    private let modelContext: ModelContext
    private let parserFactory: DocumentParserFactory

    init(
        modelContext: ModelContext,
        parserFactory: DocumentParserFactory = .shared
    ) {
        self.modelContext = modelContext
        self.parserFactory = parserFactory
    }

    // MARK: - CRUD Operations

    func getAllBooks() async throws -> [BookEntity] {
        let descriptor = FetchDescriptor<Book>(
            sortBy: [SortDescriptor(\.dateAdded, order: .reverse)]
        )
        let books = try modelContext.fetch(descriptor)
        return books.map { BookMapper.toDomain($0) }
    }

    func getBook(by id: UUID) async throws -> BookEntity? {
        let predicate = #Predicate<Book> { $0.id == id }
        let descriptor = FetchDescriptor<Book>(predicate: predicate)
        guard let book = try modelContext.fetch(descriptor).first else {
            return nil
        }
        return BookMapper.toDomain(book)
    }

    func saveBook(_ book: BookEntity) async throws {
        let dataBook = BookMapper.toData(book)
        modelContext.insert(dataBook)
        try modelContext.save()
    }

    func deleteBook(by id: UUID) async throws {
        let predicate = #Predicate<Book> { $0.id == id }
        let descriptor = FetchDescriptor<Book>(predicate: predicate)
        guard let book = try modelContext.fetch(descriptor).first else {
            throw AppError.bookNotFound
        }
        modelContext.delete(book)
        try modelContext.save()
    }

    // MARK: - Document Operations (delegate to parsers)

    func extractPages(from documentData: Data, type: DocumentType) async -> [PageEntity] {
        let parser = parserFactory.parser(for: type)
        return await parser.extractAllPages(from: documentData)
    }
}
```

## Rules
1. **Protocol in Domain** - No framework imports
2. **Implementation in Data** - Framework imports OK
3. **Return Entities** - Map from Data models using Mappers
4. **Async/throws** - Use Swift concurrency
5. **Inject dependencies** - ModelContext, services via init
6. **Delegate to services** - Parser, API clients, etc.

## Multiple Data Sources
```swift
final class TTSRepository: TTSRepositoryProtocol {
    private let apiClient: TTSAPIClient
    private let cacheService: AudioCacheService

    func synthesize(text: String) async throws -> Data {
        // Check cache first
        if let cached = cacheService.get(for: text) {
            return cached
        }

        // Fetch from API
        let audio = try await apiClient.synthesize(text: text)
        cacheService.store(audio, for: text)
        return audio
    }
}
```
