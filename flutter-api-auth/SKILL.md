---
name: flutter-api-auth
license: MIT
description: Implement or fix Flutter REST/API integration, authentication sessions, token refresh, request retries, cancellation, pagination, and cache isolation. Use for network/session behavior; preserve the project's HTTP client and auth provider.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter API and Auth

Inspect the actual auth protocol, installed HTTP client, interceptors, storage, session owner, and repository contract before editing. Reuse them; do not replace Firebase-managed authentication with a custom token system or add Dio when another client already serves the project.

Do not launch apps, servers, emulators, device sessions, or live network tests unless explicitly requested. Use focused fakes/static checks/tests where allowed; end with physical-device steps and verification gaps.

## Trace and implement the boundary

- Keep transport/DTO parsing in the data layer and expose the existing repository error contract. Validate external payloads, absent fields, and typed status/error codes. Do not expose raw server messages or credentials in UI/logs.
- Preserve timeouts, request cancellation, headers, query encoding, and TLS validation. Do not disable certificate checks to fix connectivity.
- Scope requests and caches to all relevant user/tenant/query inputs. Check identity again before accepting results or storing data. Cancellation alone cannot prevent late cache writes or undo server writes.
- Reuse the existing secure-storage/session strategy, and examine backup/logging exposure for sensitive tokens. Do not assume client storage can replace server authorization.

## Token refresh invariants

When the protocol uses refresh tokens:

1. One owner manages session generation and token version. Coalesce concurrent refreshes into one in-flight future for that generation; clear it on success and failure so waiters settle and future retries remain possible.
2. Exclude login/refresh endpoints from recursive refresh. Distinguish authentication failure from forbidden/validation/offline errors using the actual protocol; not every 401 or 403 should initiate refresh.
3. An old request's 401 may arrive after a newer token was installed. Compare the request's token/session version before starting another refresh; replay with the current token where appropriate.
4. Before storing refreshed credentials or replaying, verify the session still matches. Logout/account switch invalidates pending refresh, cancels or detaches old work, clears owned sensitive caches/headers/listeners, and prevents delayed storage writes from restoring credentials.
5. Bound replay (normally once for the auth recovery) and settle each original request exactly once. Refresh failure must not deadlock queued requests or create repeated logout/navigation effects.

A queued interceptor is a tool, not proof of single-flight refresh or session safety. Inspect the actual request/error queue interactions and avoid waiting on a refresh request blocked behind itself.

## Retry and write semantics

- Use bounded retries/backoff only for the API's retryable cases, honoring Retry-After where relevant. Do not retry cancellation, validation failures, or arbitrary auth failures indefinitely.
- A timeout can follow a successful server write. Payments/orders/uploads and other non-idempotent operations need the server's idempotency/status-reconciliation contract before replay. Reuse the same logical operation key; do not generate a fresh key for each retry.
- Replaying bodies may require rebuilding a consumed stream or multipart body. Verify replayability before retry; never silently send a partial upload.
- Keep pagination cursor and deduplication tied to the current filter/refresh generation. Separate refresh, page-load, and mutation state so one failure cannot lock unrelated work.

Use `flutter-async-state-safety` if available for broader interleaving diagnosis; the invariants here still apply independently.

## Verification and maintenance

Select focused regressions: concurrent expired-token requests, delayed refresh after logout, refresh failure, stale 401 after token replacement, offline/timeout, cancellation, malformed payload, bounded replay, and cursor retry. Assert effects and settled waiters using fakes rather than live credentials.

Manual checks: expire a test session, rapidly trigger affected requests, switch account during loading, reconnect, and retry a failed action. Expect one coherent session, recoverable errors, and no duplicate committed write. Do not claim backend idempotency is verified from client tests alone.

Check APIs against locked versions before adopting helpers. References: [Dio cancellation/interceptors](https://pub.dev/packages/dio), [Stripe idempotent requests](https://docs.stripe.com/api/idempotent_requests). Review on relevant migrations; do not automatically upgrade packages.
