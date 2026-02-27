# Mapper Pattern

Mappers convert between Domain Entities and Data models.

## Location
`Data/Mappers/{Name}Mapper.swift`

## Structure (enum with static methods)
```swift
import Foundation

enum BookMapper {
    // MARK: - Data → Domain
    static func toDomain(_ data: Book) -> BookEntity {
        BookEntity(
            id: data.id,
            title: data.title,
            author: data.author,
            coverImageData: data.coverImageData,
            documentData: data.documentData,
            documentType: data.documentType,
            pageCount: data.pageCount,
            dateAdded: data.dateAdded,
            lastReadDate: data.lastReadDate,
            progress: data.progress.map { ProgressMapper.toDomain($0) }
        )
    }

    // MARK: - Domain → Data
    static func toData(_ entity: BookEntity) -> Book {
        let book = Book(
            id: entity.id,
            title: entity.title,
            author: entity.author,
            coverImageData: entity.coverImageData,
            documentData: entity.documentData,
            documentType: entity.documentType,
            pageCount: entity.pageCount,
            dateAdded: entity.dateAdded
        )
        book.lastReadDate = entity.lastReadDate
        return book
    }

    // MARK: - Update existing Data model
    static func update(_ data: Book, with entity: BookEntity) {
        data.title = entity.title
        data.author = entity.author
        data.coverImageData = entity.coverImageData
        data.lastReadDate = entity.lastReadDate
        // Note: Don't update id, documentData, dateAdded (immutable)
    }
}
```

## DTO → Entity Mapper
```swift
// For API response models
enum VoiceMapper {
    static func toDomain(_ dto: VoiceDTO) -> VoiceEntity {
        VoiceEntity(
            id: dto.voice_id,
            name: dto.name,
            category: dto.category,
            description: dto.description,
            previewURL: dto.preview_url.flatMap { URL(string: $0) },
            labels: mapLabels(dto.labels)
        )
    }

    static func toDomain(_ dtos: [VoiceDTO]) -> [VoiceEntity] {
        dtos.map { toDomain($0) }
    }

    private static func mapLabels(_ labels: VoiceDTO.Labels?) -> VoiceEntity.Labels {
        VoiceEntity.Labels(
            accent: labels?.accent,
            age: labels?.age,
            gender: labels?.gender,
            useCase: labels?.use_case
        )
    }
}
```

## Rules
1. **Enum, not struct/class** - No state, pure transformations
2. **Static methods** - `toDomain()`, `toData()`, `update()`
3. **Bidirectional** - Support both directions when needed
4. **Nested mappers** - For related entities (ProgressMapper)
5. **Batch mapping** - Provide array versions

## Related Entity Mapper
```swift
enum ProgressMapper {
    static func toDomain(_ data: ReadingProgress) -> ReadingProgressEntity {
        ReadingProgressEntity(
            id: data.id,
            currentPage: data.currentPage,
            currentCharacterIndex: data.currentCharacterIndex,
            totalCharacters: data.totalCharacters,
            isCompleted: data.isCompleted,
            lastUpdated: data.lastUpdated
        )
    }

    static func toData(_ entity: ReadingProgressEntity, for book: Book) -> ReadingProgress {
        ReadingProgress(
            id: entity.id,
            currentPage: entity.currentPage,
            currentCharacterIndex: entity.currentCharacterIndex,
            totalCharacters: entity.totalCharacters,
            isCompleted: entity.isCompleted,
            lastUpdated: entity.lastUpdated,
            book: book
        )
    }

    static func update(_ data: ReadingProgress, with entity: ReadingProgressEntity) {
        data.currentPage = entity.currentPage
        data.currentCharacterIndex = entity.currentCharacterIndex
        data.totalCharacters = entity.totalCharacters
        data.isCompleted = entity.isCompleted
        data.lastUpdated = entity.lastUpdated
    }
}
```

## Usage in Repository
```swift
final class BookRepository: BookRepositoryProtocol {
    func getAllBooks() async throws -> [BookEntity] {
        let books = try modelContext.fetch(descriptor)
        return books.map { BookMapper.toDomain($0) }  // Map to entities
    }

    func saveBook(_ book: BookEntity) async throws {
        let dataBook = BookMapper.toData(book)  // Map to data model
        modelContext.insert(dataBook)
        try modelContext.save()
    }
}
```
