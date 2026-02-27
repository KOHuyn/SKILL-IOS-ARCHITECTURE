---
name: ios-architecture
description: iOS Clean Architecture with MVI/MVVM for SwiftUI. Use for layers, use cases, repositories, unidirectional state, DI.
version: 1.0.0
---

# iOS Architecture

Build iOS/SwiftUI apps with Clean Architecture + MVI (or MVVM) pattern.

## When to Use

- Creating new iOS/SwiftUI features
- Structuring iOS projects with layered architecture
- Implementing unidirectional state management
- Adding use cases, repositories, entities
- Refactoring to testable architecture

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation                         │
│   View ←→ Store (MVI) or ViewModel (MVVM)              │
└────────────────────────┬────────────────────────────────┘
                         │ Use Cases
┌────────────────────────▼────────────────────────────────┐
│                      Domain                             │
│   Entities │ Use Cases │ Repository Protocols           │
└────────────────────────┬────────────────────────────────┘
                         │ Implementations
┌────────────────────────▼────────────────────────────────┐
│                       Data                              │
│   Repositories │ DTOs │ Mappers │ Services              │
└─────────────────────────────────────────────────────────┘
```

## Directory Structure

```
{App}/
├── Domain/                 # Business logic (no UI dependencies)
│   ├── Entities/          # Pure data models
│   ├── UseCases/          # Business operations
│   └── Repositories/      # Repository protocols
├── Data/                   # Implementation layer
│   ├── Repositories/      # Protocol implementations
│   ├── DTOs/              # API response models
│   ├── Mappers/           # Entity ↔ DTO conversion
│   └── Services/          # External service clients
├── Presentation/           # UI layer
│   └── {Feature}/
│       ├── {Feature}View.swift
│       ├── {Feature}Store.swift   # MVI
│       └── {Feature}ViewModel.swift # MVVM
└── Core/
    ├── DI/                # Dependency injection
    └── Extensions/
```

## Pattern Selection

| Pattern | When to Use |
|---------|-------------|
| **MVI** | Complex state, multiple transitions, high testability needed |
| **MVVM** | Simpler screens, quick implementation, familiar team |

## References

### Clean Architecture
- `references/clean-architecture-layers.md` - Layer responsibilities
- `references/naming-conventions.md` - File/class naming
- `references/entity-pattern.md` - Domain entities
- `references/use-case-pattern.md` - Use case implementation
- `references/repository-pattern.md` - Repository pattern
- `references/dependency-injection.md` - DI container setup

### Presentation Patterns
- `references/mvi-pattern.md` - MVI with Store, Intent, Reducer
- `references/viewmodel-pattern.md` - MVVM ViewModel
- `references/testing-patterns.md` - Unit testing strategies

### Data Layer
- `references/dto-pattern.md` - Data Transfer Objects
- `references/mapper-pattern.md` - Entity ↔ DTO mapping

## Quick Start

1. **Define Entity** in `Domain/Entities/`
2. **Define Repository Protocol** in `Domain/Repositories/`
3. **Implement Use Case** in `Domain/UseCases/`
4. **Create DTO + Mapper** in `Data/`
5. **Implement Repository** in `Data/Repositories/`
6. **Create Store (MVI) or ViewModel (MVVM)** in `Presentation/{Feature}/`
7. **Create View** in `Presentation/{Feature}/`
8. **Register in DI Container**

## Key Principles

- Domain layer has NO dependencies on other layers
- All dependencies flow inward (Presentation → Domain ← Data)
- Use protocols for abstraction and testability
- Inject dependencies via DI Container
