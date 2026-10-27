# Deterministic Async Regression Scenarios

Choose cases matching the changed contract. Use a fake dependency returning separately controlled futures or streams. Wait until the intended requests have actually started before resolving them. Do not add all cases to every class or rely on arbitrary sleeps.

## Request replacement

1. Start A with old inputs; leave its future pending.
2. Start B with current inputs; resolve B, then A.
3. Assert current data and cache still belong to B. Repeat with A failing instead of succeeding.
4. In a separate order, finish A while B remains pending. B's busy state must remain active; A must not set a stale error.

Include A → B → A when inputs can repeat; equality of query text alone must not let the first A replace a later operation.

## Lifecycle and identity

- Start a request; close/dispose the owner; finish success/failure. No invalid state access, leaked exception, or UI effect.
- Start as user A; logout/switch to B; finish A. No A data/credentials can appear in B's memory or persistent cache. Exercise delayed persistence too, not only delayed transport.
- Change a widget entity ID while an old subscription/request is pending. Only the new entity may update the screen; the old owned subscription is released.

## Shared state and writes

- Begin load, change an unrelated field, then finish load. The unrelated change survives; no pre-await whole-state restore.
- Trigger different Bloc event types or different Cubit methods touching the same resource. Verify the intended shared ordering, not only repeated instances of one event.
- Trigger a duplicate submission before the first completes. Assert the intended number of external writes and allow a legitimate retry after failure. Disabled UI alone does not test the shared method's guard.
- For ordered distinct writes, hold the first; verify the second cannot commit first where order matters. Cross-client correctness needs a backend-level test when available.

## Refresh, pagination, and caches

- Begin page N, refresh/change filter, finish old page N. It cannot append old items or advance the new cursor.
- Fail a page, retry, return an overlapping item set. No skipped cursor or duplicate stable IDs; retry flags recover.
- Let a cached response and server response arrive in either order. The chosen freshness policy holds; an old listener cannot resurrect deleted/stale content.

## Optimistic changes

Apply A, apply B, let B succeed, then A fail. B remains. Also exercise A failure while B is still pending. For deletion, ensure a rollback does not reinsert an entity superseded by a newer authoritative result.

## Refresh authentication

Make several requests observe expiration in the same session. Hold refresh, then finish success/failure; assert one refresh, bounded replay, and all waiters settled. Repeat with logout/account change during refresh and with an old 401 arriving after a newer token is installed.

## State propagation and side effects

- Change nested data without changing unrelated fields; assert listeners/rendered output update and old emitted state stays unchanged.
- Set then explicitly clear a nullable value through copyWith/public behavior; assert the field is null.
- Rebuild/resubscribe/return to a route after a navigation/toast event. It is not repeated; a later distinct event is still delivered.

Release listeners, fake streams, containers, and owned pending work in teardown. Report scenarios that need missing backend/device facilities instead of claiming they were verified.
