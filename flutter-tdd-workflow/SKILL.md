---
name: flutter-tdd-workflow
description: Use when implementing a new feature, use case, repository, or bug fix in Flutter and the user wants test-driven development — including phrases like "build this with TDD", "write the test first", "let's TDD this feature", or any new business-logic implementation where tests should drive the code, not follow it. Enforces strict red-green-refactor instead of writing implementation first.
---

# Flutter TDD Workflow

Enforces actual test-driven development discipline: a failing test must exist before any implementation code is written for that unit. This skill exists because agents (and devs under deadline pressure) default to writing code first and tests after — this skill blocks that shortcut.

## The Loop (non-negotiable order)

For every new unit of behavior (a use case, a repository method, a Bloc event handler):

1. **Red** — Write the test first, for the smallest piece of behavior. Run it. Confirm it fails, and confirm it fails *for the expected reason* (e.g. "method doesn't exist" or "returns null" — not a typo/import error).
2. **Green** — Write the *minimum* code to make that test pass. Resist adding anything not demanded by the test yet.
3. **Refactor** — With the test passing, clean up naming/structure. Re-run the test after every refactor step to confirm it still passes.
4. Repeat for the next smallest piece of behavior.

Never skip to step 2. If asked to "implement X", the first action is always writing `X_test.dart`, not `x.dart`.

## What to Test at Each Layer

(Matches the layering in the `flutter-architecture` skill.)

- **Domain/use cases**: pure unit tests, no widget tree needed. Mock the repository interface with `mocktail`. This is where most TDD value lives — fast, no Flutter bindings required.
- **Repositories**: unit test against mocked data sources (mock Dio/Firestore calls). Test both the `Success` path and the `ResultFailure` path — assert the correct `AppFailure` subtype is returned for each exception case (e.g. `DioException` timeout → `NetworkFailure`, 401 response → `AuthFailure`).
- **Bloc/Cubit**: use `bloc_test` package — given a state, when an event is added, expect this sequence of states. Mock the use case, not the repository, at this layer.
- **Widgets**: `flutter_test` + `WidgetTester`, test behavior (tap → state change → rendered text) not implementation details. These come *last*, after logic underneath is already covered.

## Test File Conventions

- Mirror the lib/ structure: `test/features/<feature>/domain/usecases/get_listings_test.dart` etc.
- One test file per class being tested.
- Use `mocktail` for mocking (preferred over `mockito` — no codegen needed).
- Group tests with `group('GetListingsUseCase', () { ... })`, one `test()` per behavior/edge case, named as a sentence: `test('returns Failure when repository throws', () { ... })`.

## Order of Implementation for a New Feature (TDD-driven)

1. Write failing test for the use case (mocking the repository interface) → implement use case
2. Write failing test for repository implementation (mocking the data source) → implement repository
3. Write failing test for the data source if it has any logic beyond a raw API call (e.g. response mapping) → implement
4. Write failing bloc_test for the Bloc/Cubit (mocking the use case) → implement Bloc
5. Only then build widgets/UI, with widget tests for key interactions

## Definition of Done for a Unit

A unit (use case/repository method/bloc handler) is not done until:
- A test existed and failed before the implementation did
- The test passes after implementation, for the right reason
- Edge cases are covered: empty data, network failure, malformed response, unauthorized/expired token (where relevant)

## When the User Says "Just Build It Fast, Skip Tests"

Acknowledge the trade-off plainly and proceed without enforcing this skill for that specific task — don't argue, just don't silently apply TDD discipline against an explicit instruction to skip it.
