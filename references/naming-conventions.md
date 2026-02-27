# Naming Conventions

## File Naming (PascalCase)
| Type | Pattern | Example |
|------|---------|---------|
| Entity | `{Name}Entity.swift` | `BookEntity.swift` |
| Use Case | `{Name}UseCases.swift` | `BookUseCases.swift` |
| Repository Protocol | `{Name}RepositoryProtocol.swift` | `BookRepositoryProtocol.swift` |
| Repository | `{Name}Repository.swift` | `BookRepository.swift` |
| DTO | `{Name}DTOs.swift` | `TTSRouterDTOs.swift` |
| Mapper | `{Name}Mapper.swift` | `BookMapper.swift` |
| ViewModel | `{Name}ViewModel.swift` | `LibraryViewModel.swift` |
| View | `{Name}View.swift` | `LibraryView.swift` |
| Error | `{Name}Error.swift` | `DocumentParserError.swift` |

## Protocol Naming
```swift
// Repository protocols: {Name}RepositoryProtocol
protocol BookRepositoryProtocol { }

// Use case protocols: {Action}{Name}UseCaseProtocol
protocol ImportBookUseCaseProtocol { }
protocol GetBooksUseCaseProtocol { }
protocol DeleteBookUseCaseProtocol { }

// Other protocols: {Name}Protocol or {Capability}able
protocol DocumentParserProtocol { }
protocol TTSEngineProtocol { }
```

## Class/Struct Naming
```swift
// Entities (struct): {Name}Entity
struct BookEntity { }
struct VoiceEntity { }

// Use Cases (final class): {Action}{Name}UseCase
final class ImportBookUseCase: ImportBookUseCaseProtocol { }
final class GetBooksUseCase: GetBooksUseCaseProtocol { }

// Repositories (final class): {Name}Repository
final class BookRepository: BookRepositoryProtocol { }

// ViewModels (final class, ObservableObject): {Feature}ViewModel
final class LibraryViewModel: ObservableObject { }

// Mappers (enum with static methods): {Name}Mapper
enum BookMapper { }
```

## Method Naming
```swift
// Use Cases: execute() or execute(param:)
func execute(documentData: Data) async throws -> BookEntity
func execute() async throws -> [BookEntity]
func execute(bookId: UUID) async throws

// Repository CRUD: get/save/update/delete
func getAllBooks() async throws -> [BookEntity]
func getBook(by id: UUID) async throws -> BookEntity?
func saveBook(_ book: BookEntity) async throws
func updateBook(_ book: BookEntity) async throws
func deleteBook(by id: UUID) async throws

// Mappers: toDomain/toData/update
static func toDomain(_ data: Book) -> BookEntity
static func toData(_ entity: BookEntity) -> Book
static func update(_ data: Book, with entity: BookEntity)
```

## Folder Structure
```
Domain/
├── Entities/           # {Name}Entity.swift
├── UseCases/           # {Feature}UseCases.swift (groups related use cases)
├── Repositories/       # {Name}RepositoryProtocol.swift
├── Protocols/          # {Name}Protocol.swift (non-repository)
└── Errors/             # {Name}Error.swift

Data/
├── Repositories/       # {Name}Repository.swift
├── DTOs/               # {Name}DTOs.swift
├── Mappers/            # {Name}Mapper.swift
└── Services/           # {Name}Service.swift

Presentation/
└── {Feature}/
    ├── {Feature}View.swift
    ├── {Feature}ViewModel.swift
    └── Components/     # Feature-specific subviews
```
