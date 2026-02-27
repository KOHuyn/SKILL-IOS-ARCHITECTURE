# Entity Pattern

Entities are pure domain models with NO framework dependencies.

## Location
`Domain/Entities/{Name}Entity.swift`

## Structure
```swift
import Foundation  // Only Foundation allowed

struct BookEntity: Identifiable, Equatable {
    let id: UUID
    let title: String
    let author: String
    let coverImageData: Data?
    let documentData: Data
    let documentType: DocumentType
    let pageCount: Int
    let dateAdded: Date
    var lastReadDate: Date?
    var progress: ReadingProgressEntity?

    init(
        id: UUID = UUID(),
        title: String,
        author: String,
        coverImageData: Data? = nil,
        documentData: Data,
        documentType: DocumentType,
        pageCount: Int,
        dateAdded: Date = Date(),
        lastReadDate: Date? = nil,
        progress: ReadingProgressEntity? = nil
    ) {
        self.id = id
        self.title = title
        self.author = author
        self.coverImageData = coverImageData
        self.documentData = documentData
        self.documentType = documentType
        self.pageCount = pageCount
        self.dateAdded = dateAdded
        self.lastReadDate = lastReadDate
        self.progress = progress
    }
}
```

## Rules
1. **Pure structs** - No classes unless reference semantics needed
2. **No UIKit/SwiftUI** - Foundation only
3. **Identifiable** - Always include `id: UUID`
4. **Equatable** - For comparison and SwiftUI diffing
5. **Immutable by default** - Use `let`, `var` only for mutable state
6. **Default values** - Provide sensible defaults in init

## Nested/Related Entities
Group related entities in same file:
```swift
// BookEntity.swift
struct BookEntity: Identifiable, Equatable {
    // ... book properties
}

struct ReadingProgressEntity: Identifiable, Equatable {
    let id: UUID
    let currentPage: Int
    let currentCharacterIndex: Int
    let totalCharacters: Int
    var isCompleted: Bool
    let lastUpdated: Date

    var progressPercentage: Double {
        guard totalCharacters > 0 else { return 0 }
        return Double(currentCharacterIndex) / Double(totalCharacters)
    }
}

struct PageEntity: Identifiable, Equatable {
    let id: Int  // pageNumber as id
    let pageNumber: Int
    let text: String
    let qualityScore: Double?
}
```

## Enums in Domain
```swift
// DocumentType.swift in Domain/Protocols/
enum DocumentType: String, CaseIterable, Codable {
    case pdf, epub, docx, txt

    var extensions: [String] {
        switch self {
        case .pdf: return ["pdf"]
        case .epub: return ["epub"]
        case .docx: return ["docx", "doc"]
        case .txt: return ["txt", "text"]
        }
    }

    static func from(extension ext: String) -> DocumentType? {
        allCases.first { $0.extensions.contains(ext.lowercased()) }
    }
}
```

## Avoid
- NO `@Model` (SwiftData) - That's Data layer
- NO `Codable` unless needed for serialization
- NO computed properties with side effects
- NO dependencies on repositories or services
