# Current Architecture Detail

## Context Creation & Save Path

`newChildContext()` creates a fresh `privateQueueConcurrencyType` context parented to `mainQueueContext` every time it is called. Both save variants call `performSave(asynchronously:…)` in `TTNSManagedObjectContext+SaveHelpers.swift`:

1. `context.save()` on the worker (child) context
2. Recursive `performSave(…)` on `context.parent` (mainQueueContext)
3. Recursive save on `mainQueueContext.parent` (readOnlyRootContext) → flushed to PSC/disk

Synchronous path: `saveContextToStoreAndWait` (used by `saveWithBlockAndWait`).
Async path: `saveContext` (used by `saveWithBlock` and the `DidSave` notification handler on `mainQueueContext`).

## Thread-to-Context Mapping

`contextForThread()` in `TTCoreDataStack+Threading.swift` maintains a `[String: NSManagedObjectContext]` dictionary keyed by `"\(Thread.current)"`. GCD thread descriptions include the thread address, which is **not stable** — entries accumulate for threads that no longer exist. Called in 10+ places across `TTKitClass.m` and `TTKitInternal.m`.

## Contexts Created Outside the Operation Queue

Several places bypass the enqueue path entirely:

```objc
// TTKitInternal.m – local user search
dispatch_queue_t queue = dispatch_queue_create("com.tigertext.tigertext.localUserSearch", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_async(group, queue, ^{
    NSManagedObjectContext *context = [weakSelf.coreDataStack newChildContext];
    [context performBlockAndWait:^{ … }];
});
```

`TTAttachmentDownloadService.m` (line 381) also creates a raw `newChildContext()` outside any enqueue path.

## Socket Listener Protocol

Every socket listener implements:
```objc
- (void)processEvent:(TCAnySocketEvent *)event
           dbContext:(NSManagedObjectContext *)context
postContextSaveQueue:(NSOperationQueue *)postContextSaveQueue;
```
`dbContext` is a child context from the enqueue path. `postContextSaveQueue` is `TTSocketListenerManager.postOperationQueue` — an unlimited queue for post-save side effects. When migrating, `dbContext` becomes `backgroundContext` and `postContextSaveQueue` usage must be reviewed to ensure it does not touch Core Data objects after their context has been released.

## Pain Points

### Unbounded Context Proliferation
Every `saveWithBlockAndWait` call creates a new `NSManagedObjectContext`. The `contextForThread()` cache accumulates stale entries because thread descriptions include the memory address and GCD reuses threads.

### Inconsistent Enqueue Discipline
Many callers of `saveWithBlockAndWait` are **not** inside an `enqueueDataBlock` wrapper. API callbacks, socket listeners, and managers can write concurrently, causing:
- `NSMergeConflict` errors
- Crashes in `obtainPermanentIDs(for:)`
- `NSFetchedResultsController` delegate calls on the wrong thread

### Nested saveWithBlockAndWait
In `TTKitClass.m` (lines 1192–1196) and `TTKitInternal.m` (line 2090–2095), `saveWithBlockAndWait` is called inside another `saveWithBlockAndWait`. Each creates its own context; neither sees the other's changes, and the inner save propagates before the outer block finishes.

### mainQueueContext Used Off Main Thread
`contextForThread()` returns `mainQueueContext` when called on the main thread, but callers in `TTABContactsService.m` (lines 211, 222) pass it into `dispatch_async` blocks that may run off-main. `mainQueueContext` is `.mainQueueConcurrencyType` and must only be accessed via `perform`/`performAndWait` or directly on the main thread.

## Major Call-Site Inventory

| File | Pattern |
|---|---|
| `TTKit/Sources/Managers/TTKitInternal.m` | ~50 uses of `saveWithBlockAndWait`, `mainQueueContext`, `newChildContext`, `contextForThread` |
| `TTKit/Sources/Managers/TTKitClass.m` | ~40 uses; FRC creation via `mainQueueContext` |
| `TTKit/Sources/Managers/TTNetworkPersistenceManager.m` | Mixed enqueue + bare saves |
| `TTKit/Sources/Managers/TTUploadManager.m` | Enqueue + bare saves; `mainQueueContext` for state reads |
| `TTKit/Sources/Managers/API/TTAPI+Message.m` | 10+ `saveWithBlockAndWait` in API callbacks |
| `TTKit/Sources/Managers/API/TTAPI+Role.m` | 15+ `saveWithBlockAndWait`; cross-context `inContext:` calls |
| `TTKit/Sources/Managers/API/TTAPI+Roster.m` | 4 `saveWithBlockAndWait` |
| `TTKit/Sources/Managers/TTSocket/TTSocketListenerManager.m` | Owns `postOperationQueue`; routes SSE via `enqueueSSEDataBlock` |

## Test Infrastructure

- `TCTestShared/Sources/TTKitBaseTestCase.swift` — exposes `mainQueueContext` as `context` for test assertions
- `TCTestShared/Sources/coredata/NSManagedObjectContextSwizzling.swift` — swizzles `save()` for test isolation
- `TCTestShared/Sources/coredata/TTCoreDataStackTestConfig.swift` — configures in-memory store

When migrating contexts, update `TTKitBaseTestCase.context` to `backgroundContext` (write tests) or `viewContext` (display tests) as appropriate.
