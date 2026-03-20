# Claude Project Instructions – iOS App

This repository contains an iOS application written in Swift and Objective-C.  
It includes an application target, multiple Swift Packages (Swift and Objective-C), Fastlane automation, and CircleCI for CI/CD.

Claude should follow the guidelines below when assisting with code, configuration, or documentation in this repo.

---

## Tech Stack

- **Languages:** Swift, Objective-C
- **Platform:** iOS
- **Minimum Deployment Target:** iOS 17
- **UI Framework:** UIKit (primary)
- **Architecture:** MVVM with Repository pattern
- **Dependency Management:** Swift Package Manager (SPM)
- **Linting:** SwiftLint (custom configuration)
- **CI:** CircleCI
- **Automation:** Fastlane

---

## UI Framework Rules

- The app is **UIKit-first**
- SwiftUI may exist for:
  - Previews
  - Isolated screens
  - New features *only if already established*
- **Do not introduce SwiftUI into UIKit flows** without explicit request
- Match existing UI patterns, navigation, and layout conventions

---

## Architecture Overview

The app follows **MVVM with Repositories and Data Sources**.

### Layers

- **View / ViewController**
  - UIKit views and view controllers
  - Bind to ViewModels
  - No business logic
  - No direct data access

- **ViewModel**
  - Exposes state and actions for the view
  - Transforms domain models into UI-ready data
  - Coordinates with repositories
  - No UIKit dependencies

- **Repository**
  - Acts as the single source of truth
  - Decides whether data comes from:
    - Local data source
    - Network data source
  - Hides implementation details from ViewModels

- **Data Sources**
  - **Network:** API clients, DTOs, request/response mapping
  - **Local:** Persistence (e.g. UserDefaults, CoreData, files, cache)
  - Data sources should not contain business logic

- **Models**
  - Prefer immutable value types
  - Separate:
    - Network models
    - Domain models
    - Persistence models
  - Avoid leaking transport-layer models into UI

---

## Architecture Documentation

The canonical description of the app’s architecture lives in:

- `docs/AppArchitecture.md`

Claude should treat this document as the **source of truth** for:
- Architectural decisions
- Layer responsibilities
- Data flow
- Naming conventions

If there is any conflict between this file and `docs/AppArchitecture.md`, **`docs/AppArchitecture.md` takes precedence**.
Before making any architectural decision, read `docs/AppArchitecture.md` first.

## Project Structure

### App Target

- Contains:
  - App lifecycle
  - UIKit UI
  - Composition root / dependency wiring
- May contain Objective-C code
- Avoid placing reusable or testable logic directly in the app target if it belongs in a package

### Swift Packages

- Used for:
  - Domain logic
  - Networking
  - Persistence
  - Shared utilities
- Workspace includes:
  - Swift packages
  - At least one **Objective-C Swift Package**
- Respect language boundaries:
  - Avoid unnecessary Swift ↔ Objective-C bridging
  - Keep public APIs stable and intentional

---

## Objective-C Guidelines

- Existing Objective-C code should be respected and maintained
- Do not rewrite Objective-C to Swift unless explicitly requested
- When interacting with Objective-C:
  - Keep APIs simple and well-annotated (`NS_ASSUME_NONNULL`, generics, etc.)
  - Be mindful of nullability
- Avoid introducing new Objective-C unless required by existing patterns

### Objective-C and Swift Concurrency

When exposing Objective-C APIs to Swift, the goal is to make them compatible with
Swift Concurrency (`async` / `await`) wherever possible.

Claude should follow Apple’s official guidance when:
- Designing new Objective-C APIs consumed by Swift
- Modifying existing Objective-C APIs
- Annotating Objective-C code for better Swift interoperability

Reference:
- Apple Documentation: *Calling Objective-C APIs Asynchronously*  
  https://developer.apple.com/documentation/swift/calling-objective-c-apis-asynchronously

Key expectations:
- Prefer completion-handler–based Objective-C APIs that can be imported as `async`
- Use correct nullability annotations (`nullable`, `nonnull`)
- Use `NSError **` patterns that bridge cleanly to `throws`
- Avoid blocking or synchronous APIs when async behavior is intended

---

## Swift Package Guidelines

When modifying or adding Swift Packages:

- Keep package boundaries clear
- Minimize inter-package dependencies
- Update `Package.swift` carefully
- Prefer internal targets over large public APIs
- Ensure packages build independently when possible

---

## Linting (SwiftLint)

- SwiftLint is enforced using a **custom `.swiftlint.yml`**
- Claude should:
  - Follow existing lint rules strictly
  - Avoid introducing new violations
  - Match existing formatting and naming conventions
- If a rule forces awkward code:
  - Prefer refactoring over disabling rules
  - Do not add `// swiftlint:disable` without justification

---

## Localization

- Always use the `.localized` / `.localized(bundle:)` extension from `String+Localized.swift`
- **Never** use the verbose `NSLocalizedString(_:tableName:bundle:value:comment:)` form
- For strings in a Swift Package module, use `"KEY".localized(bundle: .module)`
- For strings in the main app target, use `"KEY".localized`

```swift
// ✅ correct
label.text = "OfflineModeBannerTitle".localized(bundle: .module)

// ❌ avoid
label.text = NSLocalizedString("OfflineModeBannerTitle", tableName: "Localizable", bundle: Bundle.module, value: "...", comment: "...")
```

---

## Coding Guidelines

- Follow **Swift API Design Guidelines**
- Prefer value types (`struct`) unless reference semantics are required
- Avoid force unwraps (`!`) unless provably safe
- Write code that is:
  - Testable
  - Modular
  - Readable
- Match existing naming, file organization, and access control patterns

---

## Testing

- Use XCTest
- Add or update tests alongside production changes
- Keep tests:
  - Fast
  - Deterministic
  - Independent of network or external services
- Mock repositories and data sources where appropriate

---

## CI / Fastlane Rules

- Builds run in a **clean CI environment**
- CI configuration lives in `.circleci/config.yml`
- Fastlane is the primary automation tool:
  - build
  - test
  - beta / release
  - versioning
- Do not hardcode secrets
- Assume certificates, provisioning, and credentials come from environment variables
- Prefer Fastlane lanes over raw `xcodebuild` commands

---

## What Claude Should Do

Claude is encouraged to:
- Maintain architectural consistency
- Improve clarity and separation of concerns
- Suggest MVVM-appropriate refactors
- Explain tradeoffs when proposing changes
- Ask for clarification when changes impact:
  - Architecture
  - Public APIs
  - CI/CD or release behavior

Claude should **not**:
- Introduce new architectures or UI frameworks
- Mix UIKit and SwiftUI patterns arbitrarily
- Rewrite Objective-C code without request
- Add dependencies casually
- Modify CI or Fastlane without clear explanation

---

## Assumptions

- UIKit is the primary UI framework
- MVVM + Repository is the enforced architecture
- Swift Packages are preferred for reusable code
- CI is the source of truth for correctness
- Consistency beats novelty

---

End of instructions.