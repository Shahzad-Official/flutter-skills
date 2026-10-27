---
name: flutter-tdd-workflow
license: MIT
description: Implement Flutter or Dart behavior with red-green-refactor when the user explicitly requests TDD or the project requires test-first development. Includes deterministic async race and stale-state regressions without imposing TDD on unrelated tasks.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter TDD Workflow

Apply strict test-first discipline when TDD is selected, not automatically for every feature request. Respect explicit skip-tests instructions unless higher-priority rules require tests; disclose the gap.

Do not launch apps, servers, emulators, or device sessions without an explicit request. Focused unit/widget tests and static checks may run when allowed. Do not start a backend emulator just to satisfy tests; use deterministic fakes or explain the integration gap.

## Red → green → refactor

1. Read the contract, callers, and existing tests. Write the smallest behavior test before implementation.
2. Run it and establish the intended failure. A missing new API can initially fail compilation; fix unrelated import/setup problems and use a minimal skeleton to reach a behavior assertion where possible. An accidental compile error is not a reproduced bug.
3. Implement the smallest correct fix, preserving error handling and integrity boundaries. Run the focused test.
4. Refactor while green. Verify a regression would fail on old behavior without reverting unrelated user work.

If execution is blocked by SDK/dependencies/permissions, report that red/green was not observed. Never claim TDD completion from unrun tests or silently upgrade tooling.

## Choose the actual layer

- Domain/business rules: test behavior against repository fakes/mocks. Do not invent use-case classes solely to fit a test template.
- Repositories: fake/mock data sources and test mapping. Under the custom Result contract, sources may throw; repositories return ResultFailure. Example: `returns AuthFailure when the data source reports 401`.
- Bloc/Cubit: reuse project testing tools, including bloc_test when installed. Assert meaningful state sequences/final data and control pending work.
- Riverpod: fresh container per test, dependency overrides, and a listener keeping auto-disposed providers alive while exercising pending work. Use ProviderContainer.test only when supported by the pinned version; otherwise create/dispose explicitly. Prefer observable behavior over mocking notifier internals.
- Widgets: test important interactions, navigation effects, focus/semantics, and recovery UI. UI-only behavior can start with a widget test; other layers need not be exhausted first.

Reuse installed test tools. Mocktail is a new-project preference when mocking helps, not a reason to replace existing mockito/fakes or add dependencies for simple doubles. Mirror relevant lib paths; name tests by behavior.

## Deterministic async regressions

Use completers, controlled stream controllers, and existing fake-clock utilities. Avoid arbitrary sleeps/live services and pumpAndSettle against intentionally unresolved work. Advance only to the required checkpoint.

Select the cases relevant to the fix:

- Start A then B; finish B then A. B must remain current after A succeeds or fails, including busy/error/cache state.
- Dispose, change query, or change user while pending; old completions must not emit, persist stale data, restore credentials, or navigate.
- Overlap refresh/pagination or optimistic mutations; verify stable identity, cursor ownership, and version-aware rollback.
- Several requests need token refresh; assert one refresh per session generation, bounded replay, and no session restoration after logout.
- Clear nullable state or update nested data through public behavior; assert the correct immutable value reaches listeners/UI.

Use `flutter-async-state-safety` if available for deeper scenario selection. Test the necessary invariant, not every permutation for every class. Teardown must resolve/cancel owned work and release resources.

## Completion

Report the observed red/green result, focused checks, and gaps. Cover empty/error/malformed/unauthorized cases where the changed behavior handles them, without irrelevant tests for pass-through code.

End with feature-specific physical-device steps and expected outcomes. Manual device checks supplement deterministic regressions.

## Maintenance

Check helpers against the project's locked versions. Reference: [Riverpod testing](https://riverpod.dev/docs/how_to/testing). Update for verified breaking changes without forcing package upgrades.
