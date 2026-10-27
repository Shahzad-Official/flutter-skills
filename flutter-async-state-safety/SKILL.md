---
name: flutter-async-state-safety
description: Diagnose and fix Flutter/Dart race conditions, stale UI or caches, overlapping requests, optimistic updates, and lifecycle bugs in Bloc, Cubit, Riverpod, or widget state. Use when async ordering or state ownership is implicated, including during an audit-and-fix task.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter Async and State Safety

Follow the existing architecture and installed package versions. Analyze the reachable event order before choosing a fix. A single Dart isolate can still have logical races across awaits; adding a mounted check is not enough.

Do not launch apps, servers, emulators, or device sessions without an explicit request. Focused tests/static checks are allowed subject to project instructions. Finish with physical-device reproduction steps and any unverified behavior.

## Trace the complete operation

For each affected path, identify the trigger, state owner, shared fields, query/entity/user identity, external call, await points, state/cache writes, and UI side effects. Include repository caches and background listeners, not just visible state. Trace callers and teardown; use project code-intelligence guidance when applicable.

Write a concrete failing timeline, for example: `search A starts → search B starts → B finishes → A finishes → A replaces B`. State the invariant: only the current query/session may publish or cache these results. Locate every boundary where that invariant can be broken, including catch/finally and async work inside callbacks.

For stale UI without overlapping requests, inspect immutable state construction, nested collections, equality, nullable copyWith clearing, captured closures, provider dependencies/keys, selectors, buildWhen/listenWhen, and multiple sources of truth. Do not force a rebuild before finding why the intended state is not reaching the view.

## Choose ordering from product behavior

| Operation semantics | Minimal suitable approach | Limit to verify |
| --- | --- | --- |
| Search/filter/read where newest input wins | Request generation/key guard; cancellation when supported | Guard errors, finally, and cache writes too |
| Writes where every action matters | Serialize at the shared resource owner | Local queues do not protect against other devices |
| Duplicate submit while the same action is pending | Synchronous in-flight guard/coalescing | Do not drop distinct valid actions; backend idempotency still matters |
| Independent entity loads | Concurrency with per-entity state/keys | One global busy flag or generation can incorrectly cancel unrelated work |
| Shared refresh/token refresh | One in-flight future per session generation | Every waiter settles on failure; logout invalidates the result |
| Optimistic edits | Per-operation identity/version plus reconciliation | A failed old operation must not roll back a newer edit |

Use the smallest guard at the owner that all affected callers share. Do not install a global mutex or refactor state frameworks for a local bug.

## Framework-specific boundaries

- **Bloc:** inspect the installed version and each `on<Event>` registration. With bloc_concurrency available, restartable/sequential/droppable/concurrent can express ordering. Transformers are scoped to their registration, so two event types may still overlap. Handler cancellation does not necessarily cancel the HTTP future, repository cache write, or server side effect. Check `emit.isDone` where supported before emitting after async work; it does not replace operation identity.
- **Cubit:** event transformers do not apply to ordinary methods. Use method-level generations, keyed guards, or an owned queue as needed. `isClosed` protects lifetime only. Await async result callbacks and spawned work that belongs to the method.
- **Riverpod:** verify the installed major version. Use reactive dependencies and complete family keys for inputs. Clean up owned work with `ref.onDispose`; use `ref.mounted` only where that API exists, or an appropriate disposal/generation guard. Do not mutate other providers during disposal. Invalidating a provider does not undo external side effects.
- **Widgets:** after an async gap check the exact State/context being used. Release owned timers/listeners/controllers; replace subscriptions when widget inputs change. Disposal checks do not establish that a response still belongs to the current query.

## Repair every affected boundary

- Capture operation and session identity before starting. Validate it immediately before publishing/persisting; an async persistence operation may need its own guard or a user/generation-scoped destination so logout cannot be followed by an old write.
- Invalidate identity on replacement, logout, account/tenant change, and teardown. Guard both success and failure paths. An older `finally` must not clear a newer operation's busy flag.
- Do not restore a whole pre-await state snapshot over unrelated changes. Merge the relevant field into current immutable state **after** establishing that the operation is still valid.
- Cancellation is expected control flow for superseded work, not a user-visible failure. Do not mask genuine failures or assume cancellation rolls back committed writes.
- Scope cache keys to all response-affecting inputs (user/tenant/query/sort/page). Decide when cache data is usable and when a server result supersedes it; timestamps alone may not establish ordering across devices.
- For pagination, bind cursor/page requests to the filter/refresh generation. Deduplicate by stable identity, advance the cursor after accepted success, and retain retryability after failure.
- For optimistic updates, roll back only the failed operation's still-current effect, or reconcile from the authoritative source. Do not replace the whole collection with an old snapshot.
- Keep navigation/toasts in the established side-effect mechanism with a consumable identity/transition. Fix repeated delivery without suppressing later legitimate events.
- Counters, inventory, balances, and other shared writes need backend atomicity/version checks/transactions or idempotency; a Flutter lock cannot enforce cross-client consistency.

## Verify the invariant

Read [regression scenarios](references/regression-scenarios.md) and select those relevant to the bug. Use controlled completers/streams and fake time; complete operations in the harmful order. Assert observable state, cache, navigation, and external-call effects, not private generation counters. Add a focused regression for non-trivial fixes, unless tests are explicitly excluded; disclose that gap.

Report the proven timeline, root cause, fix, and checks performed. For physical-device testing, describe fast input/taps, navigation away, connectivity changes, and user switch as applicable, with expected outcomes. Manual timing cannot prove all interleavings are safe.

## Maintenance and official references

Check semantics against pinned versions before using newer APIs. These sources explain APIs; the timelines/invariants above are the diagnostic method:

- [Bloc concurrency transformers](https://pub.dev/packages/bloc_concurrency)
- [Bloc emitter completion](https://pub.dev/documentation/bloc/latest/bloc/Emitter/isDone.html)
- [Riverpod cancellation and disposal](https://riverpod.dev/docs/how_to/cancel)
- [BuildContext across async gaps](https://dart.dev/tools/linter-rules/use_build_context_synchronously)

Update the review date after checking behavior/migration guidance, without automatically upgrading project dependencies.
