---
name: swiftui-preview
description: Guidelines and best practices for creating effective and maintainable SwiftUI Previews.
---

# SwiftUI Preview Architecture

## Overview
When writing or refactoring SwiftUI code, you must strictly follow the Closure-Based Dependency Injection pattern. We **do not use MVVM**, nor do we use Protocol-based dependency injection (e.g., no `ServiceProtocol`, `LiveService`, `MockService` classes). 

Instead, we use structs with `@Sendable` closures for services, `@Observable` classes for state management (Stores), and SwiftUI's native Environment for dependency injection.

## Core Directives

### 1. No Protocols for Services
Never define a protocol for a service. Define a single `struct` marked `Sendable` where every operation is a `@Sendable` closure property. 
*Read `references/01-closure-services.md` for implementation details.*

### 2. No View Models (Anti-MVVM)
Never create a `ViewModel`. The view's state is managed by an `@Observable` Store class owned by the Service. Views observe the store directly.
*Read `references/02-observable-stores.md` for the Store and `LoadingState` implementation.*

### 3. Fail Fast, Fail Loud
Service implementations must be injected via SwiftUI's Environment using `@Entry`. The default value for any service in the environment must **always** be an `.unimplemented` instance that calls `fatalError()` to turn silent bugs into loud crashes.
*Read `references/03-environment-integration.md` for the wiring.*

## Workflow Decision Tree
1. **Creating a new Domain/Feature:** 
   - Create a `Store` (`@Observable`, `@MainActor`, `final`).
   - Create a `Service` (`struct`, `Sendable`, containing the `Store` and closures).
   - Create static variants: `.live`, `.mock`, `.preview`, `.unimplemented`.
2. **Handling Async State:** Use the standard `LoadingState<T>` enum.
3. **Building the UI:** Read state directly from the injected service's store. Switch on `store.loadingState`.
