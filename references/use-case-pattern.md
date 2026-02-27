# Use Case Pattern

Use Cases encapsulate single business operations.

## Location
`Domain/UseCases/{Feature}UseCases.swift`

## Structure
```swift
import Foundation

// MARK: - Protocol

protocol ImportBookUseCaseProtocol {
    func execute(documentData: Data, fileExtension: String) async throws -> BookEntity
}

// MARK: - Implementation

final class ImportBookUseCase: ImportBookUseCaseProtocol {
    private let bookRepository: BookRepositoryProtocol

    init(bookRepository: BookRepositoryProtocol) {
        self.bookRepository = bookRepository
    }

    func execute(documentData: Data, fileExtension: String) async throws -> BookEntity {
        guard let documentType = DocumentType.from(extension: fileExtension) else {
            throw DocumentParserError.unsupportedFormat(fileExtension)
        }

        let metadata = bookRepository.extractMetadata(from: documentData, type: documentType)
        let coverImage = bookRepository.generateCoverImage(
            from: documentData,
            type: documentType,
            size: CGSize(width: 200, height: 280)
        )

        let book = BookEntity(
            title: metadata.title ?? "Untitled",
            author: metadata.author ?? "Unknown",
            coverImageData: coverImage,
            documentData: documentData,
            documentType: documentType,
            pageCount: metadata.pageCount
        )

        try await bookRepository.saveBook(book)
        return book
    }
}
```

## Rules
1. **Single responsibility** - One use case = one operation
2. **Protocol-based** - Define protocol, then implementation
3. **Constructor injection** - Inject repositories via init
4. **execute() method** - Standard entry point
5. **async throws** - Use Swift concurrency
6. **Return entities** - Never return DTOs or Data models

## Grouping Use Cases
Group related use cases in single file:
```swift
// BookUseCases.swift

// MARK: - Import Book
protocol ImportBookUseCaseProtocol {
    func execute(documentData: Data, fileExtension: String) async throws -> BookEntity
}
final class ImportBookUseCase: ImportBookUseCaseProtocol { ... }

// MARK: - Get Books
protocol GetBooksUseCaseProtocol {
    func execute() async throws -> [BookEntity]
    func execute(query: String) async throws -> [BookEntity]
}
final class GetBooksUseCase: GetBooksUseCaseProtocol { ... }

// MARK: - Delete Book
protocol DeleteBookUseCaseProtocol {
    func execute(bookId: UUID) async throws
}
final class DeleteBookUseCase: DeleteBookUseCaseProtocol { ... }
```

## Multiple Repositories
```swift
final class SpeakTextUseCase: SpeakTextUseCaseProtocol {
    private let ttsRepository: TTSRepositoryProtocol
    private let audioRepository: AudioRepositoryProtocol

    init(
        ttsRepository: TTSRepositoryProtocol,
        audioRepository: AudioRepositoryProtocol
    ) {
        self.ttsRepository = ttsRepository
        self.audioRepository = audioRepository
    }

    func execute(text: String) async throws {
        let audioData = try await ttsRepository.synthesize(text: text)
        try await audioRepository.play(data: audioData)
    }
}
```

## Overloaded execute()
```swift
protocol GetBooksUseCaseProtocol {
    func execute() async throws -> [BookEntity]
    func execute(query: String) async throws -> [BookEntity]
}

final class GetBooksUseCase: GetBooksUseCaseProtocol {
    func execute() async throws -> [BookEntity] {
        try await bookRepository.getAllBooks()
    }

    func execute(query: String) async throws -> [BookEntity] {
        if query.isEmpty {
            return try await execute()
        }
        return try await bookRepository.searchBooks(query: query)
    }
}
```
