# Flutter Audit Checklist (reference)

Loaded on demand by flutter-codebase-audit. Use as a scan list, not a script to read aloud.

## Security
- API keys, Stripe secret keys, Firebase service-account JSON, or Agora app certificates hardcoded in Dart files
- `.env` files or secrets committed to git history (check `.gitignore`)
- Tokens/credentials stored in plain `SharedPreferences` instead of `flutter_secure_storage`
- Firestore/Realtime DB rules not reviewed alongside client code that assumes server-side validation
- Stripe payment flow trusting client-calculated amounts instead of server-confirmed amounts

## Crash / correctness risks
- `StreamSubscription`, `AnimationController`, `TextEditingController`, `ScrollController` not disposed in `dispose()`
- `setState()` or `context` used after async gap without `if (mounted)` check
- Force-unwrap (`!`) on nullable values from API responses without a null-check path
- `late` variables that can plausibly be accessed before initialization
- Catch blocks that swallow errors silently (`catch (e) {}`) with no logging/reporting
- Repository/data source methods that throw instead of returning `Result<T>` (per the `flutter-architecture` skill's error convention)
- New failure classes declared inside a feature folder instead of added to the single app-wide `AppFailure` hierarchy in `core/error/`
- Race conditions in concurrent Firestore writes or optimistic UI updates without rollback

## Architecture / maintainability
- Widgets directly calling Firestore/Dio/Stripe instead of going through a repository layer
- No clear boundary between data models (DTOs) and domain/UI models
- Business logic embedded inside widget callbacks instead of Bloc/Cubit/Notifier
- Files exceeding ~400-500 lines or widgets nested 5+ levels with inline logic
- Inconsistent state management approach across features (mixing setState, Provider, Bloc randomly)
- Magic strings/numbers for Firestore collection names, route names, API endpoints instead of constants

## Performance
- `ListView`/`Column` with unbounded children instead of `.builder` constructors
- Missing `const` on widgets that never change
- Whole-screen rebuild via top-level `BlocBuilder`/`Consumer` instead of scoped builders
- Unoptimized images (no `cacheWidth`/`cacheHeight`, no `cached_network_image`)
- Heavy JSON parsing or image manipulation on the UI thread instead of `compute()`

## Testability
- Business logic (Cubit/Bloc/Notifier, repositories, use-cases) requiring `BuildContext` or widget tree to test
- Zero tests on payment, auth, or data-sync logic specifically
- No mocking layer (mocktail/mockito) around Firebase/Dio dependencies, making unit tests impossible without network
