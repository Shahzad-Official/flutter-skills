---
name: flutter-firebase-workflows
license: MIT
description: Implement or fix Flutter Firebase Auth, Firestore, Storage, and callable-function behavior, including rules boundaries, listener lifetimes, offline data, and concurrent writes. Apply only when the project actually uses Firebase.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter Firebase Workflows

Inspect installed FlutterFire versions, project/environment configuration, auth/session owner, data sources, rules/indexes, and relevant server functions when available. Reuse the repository boundary and existing initialization. Do not add Firebase to an unrelated project.

Do not launch apps, servers, Firebase emulators, or device sessions unless explicitly requested. Do not deploy rules/functions or mutate remote data merely to validate a client change. Focused tests/static checks may run when allowed; report unavailable server evidence and physical-device steps.

## Authorization and mapping

- Firebase client keys identify the project and are generally public; verify restrictions and data access protections rather than treating them like service-account/payment secrets. Keep server credentials out of all client bundles/assets.
- Check ownership, tenant boundaries, allowed fields, and value validation in Security Rules for direct client access. Client-side filters/hidden actions are not authorization. Admin/server paths need their own authorization because rules do not protect Admin SDK access.
- App Check mitigates abuse; it does not replace user authorization. If rules/server code are missing, report that boundary as unverified.
- Parse missing/deleted documents, nullable server timestamps, numeric types, and malformed fields explicitly. Map expected Firebase errors through the existing contract; avoid exposing sensitive diagnostic payloads.
- Callable functions validate identity and input server-side, including price/payment-sensitive values. Do not trust a client success callback as proof of final fulfillment.

## Streams, offline data, and sessions

- Own each listener and its cancellation. Rebind on user/entity/query changes and ensure old emissions cannot update a newer owner or cache. Distinguish auth initialization, authenticated, and signed-out states to avoid premature redirects.
- Account for locally cached data and pending writes where product behavior depends on freshness. A local snapshot is not automatically server acknowledgment. Define the recovery UI when a write is rejected after appearing locally successful.
- Logout does not automatically erase every persistent SDK cache or reverse pending writes. Inspect actual platform/SDK behavior and choose supported session-isolation/cleanup semantics; do not promise erasure or call clearPersistence blindly while Firestore is active.
- Tie pagination to stable ordering/identity and the current query generation. A previous listener/page must not repopulate a refreshed or different account's list.

## Concurrent writes

- For dependent read-modify-write invariants, use a transaction or authoritative server operation; for independent grouped writes, assess an atomic batch. Atomic increments can fit simple counters but do not enforce every business invariant.
- Transaction callbacks may retry. Keep them free of UI mutation, charging, notifications, or other non-repeatable side effects. Perform intended effects after committed success through the appropriate authoritative workflow.
- Transactions need connectivity; offline synchronization and last-write-wins behavior need a separate conflict policy. An optimistic rollback must not revert a newer accepted update.
- Local locks and disabled buttons cannot prevent two devices from spending the same balance or reserving the same inventory. Enforce those invariants at the backend and make retryable function/payment operations idempotent.

Use `flutter-async-state-safety` when available for detailed interleavings; do not assume an SDK stream removes ownership races.

## Storage operations

- Scope object paths and Storage Rules to the intended user/tenant; validate file size/type and allowed metadata through the actual trust boundary. Client validation alone is insufficient.
- Own upload tasks and progress subscriptions. Bind progress/completion/download-URL updates to the current entity/session so a cancelled or replaced upload cannot overwrite a newer selection.
- Distinguish upload completion from the follow-up document update. If one succeeds and the other fails, provide a retry/reconciliation path; cleanup must target only the operation's orphaned object, never an unrelated or newer upload. Do not delete remote objects simply to test cleanup.

## Verify and maintain

Use focused fakes for mapping/lifetimes and controlled old/new emissions. Client fakes do not prove Security Rules, indexes, transactions, or deployed functions. Use existing rules/integration tests only with an already authorized environment; otherwise provide steps for later verification.

Physical-device checks: airplane mode/edit/reconnect, logout/account switch while loading, document deletion, permission rejection, and concurrent edits from two test clients where applicable. Expect no cross-user UI data, correct pending/rejected states, and preserved backend invariants.

Official references: [API keys](https://firebase.google.com/docs/projects/api-keys), [Security Rules conditions](https://firebase.google.com/docs/firestore/security/rules-conditions), [transactions and batches](https://firebase.google.com/docs/firestore/manage-data/transactions), [offline data](https://firebase.google.com/docs/firestore/manage-data/enable-offline), [Storage uploads](https://firebase.google.com/docs/storage/flutter/upload-files). Check installed SDK/platform migration notes on relevant upgrade tasks; do not upgrade/deploy automatically.
