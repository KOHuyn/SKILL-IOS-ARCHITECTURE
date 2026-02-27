# Clean Architecture Layers

Layer responsibilities and dependency rules.

## Dependency Rule

```
Presentation → Domain ← Data
```

- Domain has NO external dependencies
- Data depends ONLY on Domain
- Presentation depends on Domain (via Use Cases)

## Domain Layer

**Purpose**: Business logic, pure Swift, no UI dependencies.

### Entities
```swift
// Domain/Entities/Book.swift
struct Book: Identifiable, Equatable {
    let id: String
    let title: String
    let author: String
    let coverURL: URL?
}
```

### Repository Protocols
```swift
// Domain/Repositories/BookRepositoryProtocol.swift
protocol BookRepositoryProtocol {
    func fetchBooks() async throws -> [Book]
    func getBook(id: String) async throws -> Book
    func saveBook(_ book: Book) async throws
}
```

### Use Cases
```swift
// Domain/UseCases/GetBooksUseCase.swift
protocol GetBooksUseCaseProtocol {
    func execute() async throws -> [Book]
}

final class GetBooksUseCase: GetBooksUseCaseProtocol {
    private let repository: BookRepositoryProtocol

    init(repository: BookRepositoryProtocol) {
        self.repository = repository
    }

    func execute() async throws -> [Book] {
        try await repository.fetchBooks()
    }
}
```

## Data Layer

**Purpose**: External data implementation (API, DB, files).

### DTOs
```swift
// Data/DTOs/BookDTO.swift
struct BookDTO: Codable {
    let id: String
    let title: String
    let author: String
    let cover_url: String?
}
```

### Mappers
```swift
// Data/Mappers/BookMapper.swift
enum BookMapper {
    static func toDomain(_ dto: BookDTO) -> Book {
        Book(
            id: dto.id,
            title: dto.title,
            author: dto.author,
            coverURL: dto.cover_url.flatMap { URL(string: $0) }
        )
    }
}
```

### Repository Implementation
```swift
// Data/Repositories/BookRepository.swift
final class BookRepository: BookRepositoryProtocol {
    private let apiService: APIServiceProtocol

    init(apiService: APIServiceProtocol) {
        self.apiService = apiService
    }

    func fetchBooks() async throws -> [Book] {
        let dtos: [BookDTO] = try await apiService.get("/books")
        return dtos.map(BookMapper.toDomain)
    }
}
```

## Presentation Layer

**Purpose**: UI components, state management (MVI/MVVM).

### Store (MVI) or ViewModel (MVVM)
```swift
// Presentation/Library/LibraryStore.swift
@MainActor
final class LibraryStore: ObservableObject {
    @Published private(set) var state: LibraryState

    private let getBooksUseCase: GetBooksUseCaseProtocol

    init(getBooksUseCase: GetBooksUseCaseProtocol) {
        self.state = LibraryState()
        self.getBooksUseCase = getBooksUseCase
    }
}
```

### View
```swift
// Presentation/Library/LibraryView.swift
struct LibraryView: View {
    @StateObject private var store: LibraryStore

    init(getBooksUseCase: GetBooksUseCaseProtocol) {
        _store = StateObject(wrappedValue: LibraryStore(getBooksUseCase: getBooksUseCase))
    }
}
```

## Key Benefits

- **Testability**: Mock protocols at each layer
- **Maintainability**: Changes isolated to single layer
- **Scalability**: Easy to add features without breaking existing code
