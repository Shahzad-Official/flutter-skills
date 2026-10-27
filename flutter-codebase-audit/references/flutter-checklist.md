# Flutter Audit Checklist

Use relevant sections as hypotheses to verify against source and reachable paths. Pattern matches alone are not findings.

## Security and integrity

- Server secrets, service accounts, payment secret keys, signing credentials in source/history/build assets; redact values. Gitignore does not prove a secret was never committed.
- Distinguish public client identifiers from credentials. Firebase client keys are generally public; check restrictions, Security Rules, and App Check instead of flagging their mere existence.
- Sensitive token storage/backup exposure, credentials or personal data in logs/analytics, stale authorization headers after logout.
- Server ownership/field validation; hidden UI or query filters are not access control.
- Client-controlled payment amounts, unsafe write retries, fulfillment based only on a client callback.
- User/tenant data retained in shared caches, queued work, deep links, or resumed sessions after account changes.

## Races and stale state

- Older successes, errors, cache writes, or finally blocks overwrite newer work. Guards must cover side-effect boundaries.
- Debounce/handler cancellation without stale-result suppression or transport cancellation; client cancellation does not undo server writes.
- Bloc transformers cover one event registration while different types mutate shared state; Cubit methods overlap without event transformers.
- Pre-await snapshots restored after await, losing unrelated fields changed meanwhile.
- Concurrent token refreshes or refresh completion after logout restore retired credentials.
- Pagination races refresh/filter changes: duplicate items, stale cursors, skipped retries, or missing stable IDs.
- Failed optimistic mutation A rolls back successful B; local serialization assumed to protect multi-device writes.
- In-place nested mutation, incomplete equality, shallow copies, nullable copyWith fields that cannot explicitly clear.
- ref.read used where reactive dependencies are needed, incomplete family/cache keys, stale closures, buildWhen/listenWhen/select suppressing necessary updates.
- Duplicate timers/listeners, subscriptions not replaced when IDs change, navigation/toasts repeated from persistent states.

## Lifecycle and correctness

- Controllers/subscriptions/focus nodes/timers not released by their owner, or shared clients closed by a borrower.
- BuildContext/navigation/setState after await without checking that exact State/context lifetime; checking a different context is insufficient.
- Work completes after Bloc close/provider disposal; async callbacks or fire-and-forget futures lose failures.
- Busy flags remain stuck, or old completions clear them while newer work is pending.
- Unsafe unwraps/casts, missing JSON fields, invalid numbers/dates, reachable uninitialized late fields.
- Swallowed errors, raw server messages in UI, expected failures escaping the established repository contract. Data-source exceptions may be intentional.
- Timezone/DST/timestamp-unit mistakes, missing validation, numeric precision assumptions in money, duplicate submissions, empty/null conflation.

## Architecture and tests

- I/O in build/repeated on rebuild, hidden global dependencies, business rules coupled to widget lifetimes, DI/session scope leaks.
- Model boundaries causing concrete serialization/mutation bugs; duplicate DTO/entity types are not automatically required.
- Untested auth/payment/sync behavior. Fakes/manual injection/mocks are valid; a missing mocking package does not prove untestability.
- Real services or sleeps in ordering tests, shared provider/test state, tests mirroring implementation instead of behavior.

## UI and performance

- Unbounded scroll/layout constraints, keyboard-hidden actions, small-screen overflow, text-scale/RTL/focus/semantics failures.
- Missing recovery UI, or refresh errors unnecessarily discarding useful content.
- Large eager lists, unnecessarily large decoded images, repeated I/O/work in builders, or demonstrated excessive rebuilds. Short eager lists can be fine.
- Heavy parsing/work blocks interaction. Isolates/caching packages/const are contextual options, not unconditional requirements.

References: [Firebase API keys](https://firebase.google.com/docs/projects/api-keys), [BuildContext across async gaps](https://dart.dev/tools/linter-rules/use_build_context_synchronously).
