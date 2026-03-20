## Cancellation Pattern

When a Task can be invoked multiple times in within a short span of time:
1. Store the task in the class as a property
2. Cancel the previous `Task` before launching a new one. 
3. Check `Task.isCancelled` after every `await`

```swift
private func search(with query: String, continuation: String? = nil) {
    cancellableTask?.cancel()
    state = .loading
    cancellableTask = Task { @MainActor in
        guard !Task.isCancelled else { return }

        let result = await env.searchRepository.inboxSearch(request)

        guard !Task.isCancelled, searchTerm == query else { return }

        switch result {
        case .success(let value): ...
        case .failure: state = .error(message: ...)
        }
    }
}
```

Cancel all tasks in `deinit`:

```swift
deinit {
    cancellableTask?.cancel()
    debounceTask?.cancel()
}
```

---

## Debounced Task Pattern

For input-driven searches, cancel the previous debounce `Task` before sleeping:

```swift
private func scheduleSearch() {
    debounceTask?.cancel()
    debounceTask = Task { @MainActor [weak self] in
        try? await Task.sleep(for: .milliseconds(400))
        guard !Task.isCancelled, let self else { return }
        search(with: self.searchTerm)
    }
}
```

---

## withThrowingTaskGroup for Bulk Fetches

```swift
private func bulkRoleLookup(ids: Set<String>, orgToken: String) async throws -> [TCAccount] {
    try await withThrowingTaskGroup(of: TCAccount.self, returning: [TCAccount].self) { group in
        for id in ids {
            group.addTask {
                try await fetchFromRemote(id: id, orgToken: orgToken, isRole: true)
            }
        }
        return try await group.reduce(into: []) { $0.append($1) }
    }
}
```

---

## for-await and Capturing Self

On long-running Tasks that are awaiting values produced from an `AsyncStream`, make sure to capture `weak self` and check Task cancellation within the body of the loop to be able to break out of it.

```swift
final class ViewModel {
    init() {
        Task {
            for await value in stream {
                handle(value) // ❌ memory-leak, self is captured and the task will run indefinitely
            }
        }
    }
}
```

```swift
final class ViewModel {
    init() {
        Task { [weak self, stream] in
            for await value in stream {
                guard let self else { break }
                handle(value) // ✅ no memory-leak, we break out of the look if self is nil by the next time the stream produces a value.
            }
        }
    }
}
```