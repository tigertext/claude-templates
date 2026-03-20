## Method Signatures

- Prefer `async throws` over `Result`

```swift
protocol Repository {
    func get(userId: String) async throws -> TCAccount // <-- use this ✅
    func get(userId: String) async -> Result<TCAccount, Error> // <-- avoid this ❌
}
```

---

## @MainActor on ViewModels

Declare `@MainActor` at the view model's class level so all stored properties and methods are isolated by default:

```swift
@Observable
@MainActor
public final class TCDelegationBannerViewModel { ... }
```

---

## actor for Shared Mutable State

Repositories with shared mutable state across callers are declared as `actor`:

```swift
public final actor TCPresenceRepository {
    nonisolated public static func shared() -> TCPresenceRepository { ... }
    nonisolated public let publisher = PassthroughSubject<TCPresence, Never>()
}
```

Use `nonisolated` only for:
- Static factory methods
- Combine `Publisher` properties (must be `Sendable`)

---

## Notification Observation

Use `NotificationCenter.default.publisher(for:).sink` with Combine. Bridge to async code with `Task { }` inside the sink closure:

```swift
private func observeChanges(in group: TCGroup) {
    NotificationCenter
        .default
        .publisher(for: .ttKitGroupDidUpdateChange)
        .sink { [weak self] notification in
            guard let token = notification.userInfo?["groupToken"] as? String,
                  token == group.token
            else { return }

            self?.setupObservation(for: .group(id: group.token))
        }
        .store(in: &cancellables)
}
```

For actor-isolated repositories, bridge via `Task`:

```swift
NotificationCenter
    .default
    .publisher(for: .ttKitDoNotDisturbSettingChanged)
    .sink { [weak self] _ in
        Task(priority: .medium) {
            await self?.handleLocalUserDndChange()
        }
    }
    .store(in: &cancellables)
```

Store all cancellables in `Set<AnyCancellable>` and clear in `reset()` / `deinit`.

---