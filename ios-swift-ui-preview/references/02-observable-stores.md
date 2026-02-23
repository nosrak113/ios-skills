# SwiftUI Preview: Observable Stores & State Management

## The View Model Replacement
We **do not use View Models**. The Store *is* the view model. It holds observable state that views react to. Views read data from the Store, but only the Store's methods can modify it.

## Store Architecture Rules
1. Must be annotated with `@MainActor` (UI state lives on the main thread).
2. Must be annotated with the `@Observable` macro.
3. Must be a `final class`.
4. Properties must be `private(set)`.
5. Expose computed properties for data derivations (e.g., filtering features).
6. Expose mutating functions for the Service to call.

```swift
@MainActor
@Observable
final class LandmarkStore {
    private(set) var landmarks: [Landmark] = []
    private(set) var loadingState: LoadingState<[Landmark]> = .idle

    var featuredLandmarks: [Landmark] {
        landmarks.filter { $0.isFeatured }
    }

    func landmarks(for category: Category) -> [Landmark] {
        landmarks.filter { $0.category == category }
    }

    func setLoading() { loadingState = .loading }
    
    func setLandmarks(_ landmarks: [Landmark]) {
        self.landmarks = landmarks
        loadingState = .loaded(landmarks)
    }
    
    func setError(_ error: Error) { loadingState = .failed(error) }
}
```

## The Loading State Enum
All async operations must use the standard `LoadingState` enum. This generic enum handles the full async lifecycle. Include the exact computed properties below for easy view access:

```swift
enum LoadingState<T: Sendable>: Sendable {
    case idle
    case loading
    case loaded(T)
    case failed(Error)

    var value: T? {
        if case .loaded(let value) = self { return value }
        return nil
    }

    var isLoading: Bool {
        if case .loading = self { return true }
        return false
    }

    var error: Error? {
        if case .failed(let error) = self { return error }
        return nil
    }
}
```
