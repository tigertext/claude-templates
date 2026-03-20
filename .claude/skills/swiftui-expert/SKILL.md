---

name: swiftui-expert
description: Write, review, or improve SwiftUI code following best practices for state management, view composition, performance. Use when building new SwiftUI features, refactoring existing views, reviewing code quality, or adopting modern SwiftUI patterns.

---

You are a SwiftUI expert for this iOS codebase. Use this skill to build, review, or improve SwiftUI features with correct state management, optimal view composition. Prioritize native APIs, Apple design guidance, performance-conscious patterns, `@Observable` on ViewModels, UIKit/SwiftUI bridging, and navigation patterns.

---

## General Guidelines

### State Management

- @State must be private; use for internal view state
- @Binding only when a child needs to modify parent state
- @StateObject when view creates the object; @ObservedObject when injected
- Use @State with @Observable classes; use @Bindable for injected observables needing bindings
- Use let for read-only values; var + .onChange() for reactive reads
- Never pass values into @State or @StateObject — they only accept initial values
- Nested ObservableObject doesn't propagate changes — pass nested objects directly; @Observable handles nesting fine

## ViewModel Declaration

- Use the `@Observable` macro with `@MainActor`:
- Use `@ObservationIgnored` for:
    - Stored `Task` references (async bags)
    - Callback closures (`onNavigate`, `Handlers`)
- Inject and bundle dependencies via a lightweight `struct Environment` with a `static func production()` method.
- Callbacks — navigation and analytics — are injected after construction as a `struct Handlers`.
    - The parent view assigns `Handlers` right after creating the ViewModel

## SwiftUI Patterns

### SwiftUI Views

- Use the `ViewState` enum pattern when views have a finite number of states (loading -> loaded -> error). Drive content from `viewModel.state`
- Take the ViewModel as a `let` constant when a parent object already holds a strong reference. 
    - If the View owns the ViewModel, declare it as `@State` and provide an injectable initializer for tests. 
    - The `@Observable` macro handles observation automatically.
- Extract repeated UI or delimited components with state (e.g. screens, rows, buttons) into named sub-views (structs) to prevent unnecessary full-tree re-renders.
- Extract UI into `computed properties` only if: 
  - The view's layout is trivial
  - Does not need state
  - Is a single-liner
- Pass the ViewModel down to sub-views by reference (it is a `final class`).
- Keep local `@State` only for UI-only concerns (e.g., expansion toggles, scroll offsets).

### UIKit ↔ SwiftUI Bridge

The standard pattern for embedding a SwiftUI view inside UIKit:

1. Create a `UIHostingController` subclass.
2. Inject the ViewModel through an initializer.
3. Call `super.init(rootView:)` and forward the ViewModel to the view.

The ViewController is the composition root: it owns the ViewModel, wires `Handlers`, and controls lifecycle.

## Navigation

### Pure SwiftUI Flows

- Use `NavigationStack` with `NavigationPath` and value-based navigation.
- Keep the `NavigationStack` and route enum at the feature root; child views do not need to know about the full route space.

### UIKit-Hosted Flows (Route + Navigator + Closure)

- Use the Route + Navigator pattern.
- Define a Route enum with possible routes and the data they need.
- The Navigator takes in a route and a navigationController, builds the next ViewController, and defines how the navigation is performed.
- Route and Navigator are feature-specific.
- Create a `UIHostingViewController` and bind it to the ViewModel to react to navigation intents. The ViewController will use the Navigator and pass down its `navigationController`. 


## Additional Resources

- For examples of ViewState, see [views.md](references/views.md)
- For examples of navigation patterns, see [navigation.md](references/navigation.md)