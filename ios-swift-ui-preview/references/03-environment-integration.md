# SwiftUI Preview: Environment Integration

## Environment Setup
Use SwiftUI's `@Entry` macro on `EnvironmentValues` to inject services. 
**Crucial Rule:** The default value must always be the `.unimplemented` variant. Do not use optionals (`nil`).

```swift
extension EnvironmentValues {
    @Entry var landmarkService: LandmarkService = .unimplemented
}
```

Provide a view modifier for ergonomic injection:
```swift
extension View {
    func landmarkService(_ service: LandmarkService) -> some View {
        environment(\.landmarkService, service)
    }
}
```

## Service Containers
For apps with multiple services, group them in a `Services` struct. This container must also have `.live`, `.mock`, `.preview`, and `.unimplemented` static variants.

```swift
struct Services: Sendable {
    let landmarks: LandmarkService
    let analytics: AnalyticsService
}

extension View {
    func services(_ services: Services) -> some View {
        environment(\.services, services)
            .environment(\.landmarkService, services.landmarks)
            .environment(\.analyticsService, services.analytics)
    }
}
```

Inject at the root App level:
```swift
@main
struct LandmarksApp: App {
    let appServices: Services = .live(baseURL: URL(string: "https://api.example.com")!)

    var body: some Scene {
        WindowGroup {
            ContentView()
                .services(appServices)
        }
    }
}
```

## Views Observe the Store Directly
Views must access the service from the environment, grab the store via a private computed property, and switch on the `loadingState`. 

There is no View Model. Data transformation logic belongs in the Store or Domain Model, and operations belong in the Service.

```swift
struct LandmarkListView: View {
    @Environment(\.landmarkService) private var landmarkService

    private var store: LandmarkStore { landmarkService.store }

    var body: some View {
        Group {
            switch store.loadingState {
            case .idle, .loading:
                ProgressView("Loading landmarks...")
            case .failed(let error):
                ContentUnavailableView(
                    "Unable to Load",
                    systemImage: "exclamationmark.triangle",
                    description: Text(error.localizedDescription)
                )
            case .loaded:
                landmarksList
            }
        }
        .navigationTitle("Landmarks")
    }
    
    // ... UI implementation mapping over `store.landmarks` or `store.featuredLandmarks`
}
```

## Previews
Because of the setup, previews require zero mock setup boilerplate. Just inject `.preview`:

```swift
#Preview {
    NavigationStack {
        LandmarkListView()
    }
    .services(.preview)
}
```
