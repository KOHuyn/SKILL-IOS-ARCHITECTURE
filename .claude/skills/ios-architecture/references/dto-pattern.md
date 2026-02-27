# DTO Pattern

DTOs (Data Transfer Objects) model external API responses.

## Location
`Data/DTOs/{Name}DTOs.swift`

## Structure
```swift
import Foundation

// MARK: - Voice DTOs

struct VoicesResponseDTO: Codable {
    let voices: [VoiceDTO]
}

struct VoiceDTO: Codable {
    let voice_id: String
    let name: String
    let category: String?
    let description: String?
    let preview_url: String?
    let labels: Labels?

    struct Labels: Codable {
        let accent: String?
        let age: String?
        let gender: String?
        let use_case: String?
    }
}
```

## Rules
1. **Codable** - For JSON serialization
2. **snake_case** - Match API field names (use CodingKeys if needed)
3. **Optional fields** - API may omit fields
4. **Nested structs** - For nested JSON objects
5. **Group in single file** - Related DTOs together

## CodingKeys for camelCase
```swift
struct SynthesizeRequestDTO: Codable {
    let text: String
    let modelId: String
    let voiceSettings: VoiceSettings

    struct VoiceSettings: Codable {
        let stability: Double
        let similarityBoost: Double

        enum CodingKeys: String, CodingKey {
            case stability
            case similarityBoost = "similarity_boost"
        }
    }

    enum CodingKeys: String, CodingKey {
        case text
        case modelId = "model_id"
        case voiceSettings = "voice_settings"
    }
}
```

## Request vs Response DTOs
```swift
// Data/DTOs/TTSRouterDTOs.swift

// MARK: - Request DTOs

struct SynthesizeRequestDTO: Codable {
    let text: String
    let voiceId: String
    let speed: Double?

    enum CodingKeys: String, CodingKey {
        case text
        case voiceId = "voice_id"
        case speed
    }
}

// MARK: - Response DTOs

struct SynthesizeResponseDTO: Codable {
    let audioUrl: String
    let duration: Double?

    enum CodingKeys: String, CodingKey {
        case audioUrl = "audio_url"
        case duration
    }
}

struct ErrorResponseDTO: Codable {
    let error: String
    let message: String?
    let code: Int?
}
```

## Usage in API Client
```swift
final class TTSAPIClient {
    func synthesize(request: SynthesizeRequestDTO) async throws -> Data {
        var urlRequest = URLRequest(url: endpoint)
        urlRequest.httpBody = try JSONEncoder().encode(request)

        let (data, response) = try await URLSession.shared.data(for: urlRequest)

        // Handle error response
        if let httpResponse = response as? HTTPURLResponse,
           httpResponse.statusCode >= 400 {
            let error = try JSONDecoder().decode(ErrorResponseDTO.self, from: data)
            throw APIError.serverError(error.message ?? error.error)
        }

        return data
    }

    func fetchVoices() async throws -> [VoiceDTO] {
        let (data, _) = try await URLSession.shared.data(from: voicesEndpoint)
        let response = try JSONDecoder().decode(VoicesResponseDTO.self, from: data)
        return response.voices
    }
}
```

## Mapper Integration
```swift
// In Repository
func getVoices() async throws -> [VoiceEntity] {
    let dtos = try await apiClient.fetchVoices()
    return VoiceMapper.toDomain(dtos)  // DTO → Entity
}

func synthesize(text: String, voiceId: String) async throws -> Data {
    let request = SynthesizeRequestDTO(
        text: text,
        voiceId: voiceId,
        speed: nil
    )
    return try await apiClient.synthesize(request: request)
}
```

## Avoid
- NO business logic in DTOs
- NO methods (pure data)
- NO references to Entities
- NO UI-related types
