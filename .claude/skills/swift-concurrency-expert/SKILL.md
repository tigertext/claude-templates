---

name: swift-concurrency-expert
description: Swift Concurrency review and remediation for Swift 6+. Use when asked to review Swift Concurrency usage, improve concurrency compliance, or fix Swift concurrency compiler errors in a feature or file.

---

# Swift Concurrency Expert

## Rules

1. Analyze Package.swift or .pbxproj to determine Swift language mode (5.x vs 6) and toolchain before giving advice.
2. Before proposing fixes, identify the isolation boundary: `@MainActor`, custom actor, actor instance isolation, or nonisolated.
3. Do not recommend `@MainActor` as a blanket fix. Justify why main-actor isolation is correct for the code.
4. Prefer structured concurrency (child tasks, task groups) over unstructured tasks. Use `Task.detached` only with a clear reason.
5. If recommending `@preconcurrency`, `@unchecked Sendable`, or `nonisolated(unsafe)`, require:
   - a documented safety invariant
   - a follow-up ticket to remove or migrate it

## Triage Checklist (Before Advising)

- Capture the exact compiler diagnostics and the offending symbol(s).
- Identify the current isolation boundary and module defaults (`@MainActor`, custom actor, default isolation).
- Confirm whether the code is UI-bound or intended to run off the main actor.

## Swift Concurrency Safety Review Checklist

- Are `async`/`await` call sites correctly structured?
- Are `Task` lifetimes managed (no detached tasks that outlive their context unintentionally)?
- Is `@MainActor` used correctly for UI updates?
- Are actors used where shared mutable state requires protection?
- Are there data races or shared mutable state accessed from multiple concurrency domains?
- Are `AsyncSequence` or Combine pipelines used correctly?
- When bridging Objective-C APIs: are completion-handler–based APIs annotated for async import? Do nullability annotations bridge cleanly?

## Additional Resources

- For examples of preferred patterns, see [examples.md](references/examples.md)
- For task creation, cancellation, debouncing, see [task-management.md](references/task-management.md)
- For using concurrency with SwiftUI, see [swiftui-concurrency.md](references/swiftui-concurrency.md)