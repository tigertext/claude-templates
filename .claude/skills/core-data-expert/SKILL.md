---
name: core-data-expert
description: Use when working with Core Data in this iOS codebase — migrating context usage, fixing merge conflicts, modernizing the stack, or reviewing Core Data patterns.
---

# Core Data Expert

## General Guidelines

- Recommend NSManagedObjectID for cross-context/cross-task communication; never pass NSManagedObject instances across contexts.
- Avoid leaking Core Data concerns into ViewModels or Views.
- Confirm the context matches the work (UI vs background).
- Use `saveWithBlock(_:completion:)` to avoid blocking the calling thread.

## Current Stack

Stack lives in `TCCoreData/Sources/Core Data Stack/`. Central class: `TTCoreDataStack` (`@objc`).

### Context Hierarchy (legacy — still in use)

```
NSPersistentStoreCoordinator
    │
readOnlyRootContext    ← private-queue; PSC-connected; the persisting root (name is misleading — not read-only)
    │
mainQueueContext       ← main-queue child; used by FetchedResultsControllers and most read callers
    │
"Worker contexts" (n)  ← private-queue children created on demand via newChildContext(); destroyed after each block
```

Key APIs on `TTCoreDataStack`:
- `newChildContext()` — creates a new private-queue child of `mainQueueContext` on every call
- `contextForThread()` — caches one child context per thread description (`"\(Thread.current)"`); keys are unstable
- `saveWithBlockAndWait(_:)` / `saveWithBlock(_:)` — create an ephemeral child, run a block, save up the parent chain
- `dataOperationQueue` — serial `OperationQueue` for SSE/socket writes
- `persistedAPICallsOperationQueue` — unlimited `OperationQueue` for persisted API requests
- Enqueue methods: `enqueueDataBlock`, `enqueueSSEDataBlock`, `enqueueHighPriorityDataBlock`, `enqueueVeryLowPriorityDataBlock`, `enqueuePersistedAPIOperationBlock`

### Entity Inheritance (Party hierarchy → single SQLite table Z_PARTY)

```
Party (Abstract)
  ├── User → Patient, PatientContact, Role
  └── Group → RoleGroup
```

All subclasses share one table. Concurrent writes from any context collide at the row level — the primary source of merge conflicts.

Heavy-write non-inherited entities: `Message`, `RosterEntry`, `MessageStatus`, `UploadItem`, `DownloadItem`.

---

## Reference Files

- [current-architecture.md](references/current-architecture.md) — full architecture detail, pain points, call-site inventory
- [entity-model.md](references/entity-model.md) — entity inheritance tree, model versions, migration notes
