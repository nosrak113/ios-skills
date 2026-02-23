# Closure-Based Services

## The Protocol Problem
Do not use protocols for dependency injection. Protocols require multiple conforming types (protocol + live implementation + mock implementation), creating unnecessary boilerplate and duplicating state management.

## Services as Structs with Closures
Services must be defined as a single `struct` conforming to `Sendable`. Each dependency/operation is a `@Sendable` closure property. The struct must own an instance of its corresponding `Store`.

```swift
struct LandmarkService: Sendable {
    let store: LandmarkStore

    var fetchLandmarks: @Sendable () async throws -> Void
    var fetchLandmark: @Sendable (String) async throws -> Landmark?
    var fetchLandmarksByCategory: @Sendable (Category) async throws -> [Landmark]
}
```

## Static Factory Implementations
All implementations (live, mock, preview, unimplemented) live as static properties or factory methods on the service struct.

### 1. Live Implementation
The `.live` factory maps API models to Domain models. It injects a network client (also a closure-based dependency) and updates the encapsulated store.
```swift
extension LandmarkService {
    static func live(client: NetworkClient, baseURL: URL) -> LandmarkService {
        let store = LandmarkStore()

        return LandmarkService(
            store: store,
            fetchLandmarks: {
                await store.setLoading()
                do {
                    let url = baseURL.appendingPathComponent("landmarks")
                    let response = try await client.fetch(LandmarkListApiResponse.self, from: url)
                    let landmarks = response.items.compactMap(\.domainModel)
                    await store.setLandmarks(landmarks)
                } catch {
                    await store.setError(error)
                    throw error
                }
            },
            // ... other closures
        )
    }
}
```

### 2. Mock Implementation
The `.mock` implementation must use `Task.sleep` to simulate realistic network delays for UI testing.
```swift
extension LandmarkService {
    static var mock: LandmarkService {
        let store = LandmarkStore()

        return LandmarkService(
            store: store,
            fetchLandmarks: {
                await store.setLoading()
                try await Task.sleep(for: .milliseconds(500))
                let landmarks = LandmarkApiModel.mockApiResponse.compactMap(\.domainModel)
                await store.setLandmarks(landmarks)
            },
            // ... other closures
        )
    }
}
```

### 3. Preview Implementation
For SwiftUI Previews, pre-populate the store immediately on the `@MainActor`. Fetch closures should be empty or instantly return local data.
```swift
extension LandmarkService {
    static var preview: LandmarkService {
        let store = LandmarkStore()
        let landmarks = LandmarkApiModel.mockApiResponse.compactMap(\.domainModel)

        // Pre-populate immediately
        Task { @MainActor in
            store.setLandmarks(landmarks)
        }

        return LandmarkService(
            store: store,
            fetchLandmarks: { },  // Already loaded, do nothing
            // ... other closures
        )
    }
}
```

### 4. Unimplemented Implementation (Required Default)
Every closure must call `fatalError()`. This guarantees a loud crash if a view calls an un-injected service method.
```swift
extension LandmarkService {
    static var unimplemented: LandmarkService {
        LandmarkService(
            store: LandmarkStore(),
            fetchLandmarks: { fatalError("fetchLandmarks not implemented") },
            fetchLandmark: { _ in fatalError("fetchLandmark not implemented") },
            fetchLandmarksByCategory: { _ in fatalError("fetchLandmarksByCategory not implemented") }
        )
    }
}
```
