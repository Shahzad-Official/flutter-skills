---
name: flutter-architecture
description: Design or scaffold Flutter projects and features, layer boundaries, state management, and dependency injection. Use for architecture decisions; preserve existing project conventions unless a migration is requested.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter Architecture

Use these defaults for new projects. In existing apps, read project instructions, SDK constraints, pubspec/lockfile, and a neighboring feature first. Reuse its state management, DI, serialization, error contract, and components. A feature or bug fix is not an architecture migration.

Do not launch apps, servers, emulators, or device sessions unless explicitly requested. Use focused static checks/tests when allowed; finish with physical-device verification steps and disclose checks not performed.

## Boundaries

- Presentation owns widgets and Bloc/Cubit or Riverpod notifiers. Keep business rules and external I/O out of widgets.
- Domain owns pure Dart entities, repository interfaces, and use cases where business logic warrants them. No Flutter, Dio, or Firebase SDK imports.
- Data owns repository implementations, DTO mapping, and data sources wrapping APIs, SDKs, and storage.
- Core owns genuinely shared contracts/infrastructure. Feature code belongs inside its feature.

Source dependencies: `Presentation → Domain ← Data`. Repository interfaces belong in Domain; Data implements them. Domain can depend on pure-Dart Core contracts, never Flutter-dependent utilities. Data/Core must not import Presentation.

```text
lib/
  core/
    error/           # AppFailure
    result/          # Result<T>
    network/
    constants/
    theme/
    di/
  features/
    <feature_name>/
      data/
        models/
        repositories/
        datasources/
      domain/
        entities/
        repositories/
        usecases/
      presentation/
        bloc/        # or notifiers/, following the project
        pages/
        widgets/
  app.dart
  main.dart
```

Create folders when they contain real code. Use cases earn their place through business rules, orchestration, or reuse; avoid mechanical pass-through classes and duplicate DTO/entity types. Preserve equivalent boundaries in a simpler established architecture.

## State and ownership

- Default to Bloc/Cubit for new projects with multi-step async flows. Riverpod is an alternative chosen for the project, not a reason to add another state framework for a small feature.
- Local widget state, focus, controllers, and animations can coexist with business state. Keep one owner for each business-state value.
- Model loading/success/empty/failure explicitly. New Bloc flows default to sealed states; preserve sound existing representations such as AsyncValue or immutable status/data states.
- Separate initial load, refresh, pagination, and submission when they can overlap. Preserve usable content during refresh failures when appropriate.
- Keep state and nested collections immutable; equality must include fields affecting rendering.
- Choose latest-wins, queued, ignore-duplicate, or independent behavior before adding async work. Check operation identity as well as lifecycle before publishing or persisting results. A mounted/closed check alone cannot prevent stale data.
- Scope listeners, cache keys, and pending work to user/tenant/query. Invalidate old work on logout, account switch, and changed inputs.

For overlapping requests/stale state, use `flutter-async-state-safety` if available; otherwise trace starts, awaits, completions, state/cache writes, and teardown with these invariants.

## Error contract for new projects

Preserve an established app's contract. New projects default to a custom sealed `Result<T>` and shared `AppFailure`, without adding dartz/Either. The sealed-class/pattern examples require Dart 3; do not upgrade an older SDK merely to copy this template.

`core/result/result.dart`:

```dart
import '../error/app_failure.dart';

sealed class Result<T> {
  const Result();
  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(AppFailure failure) = ResultFailure<T>;

  R fold<R>({
    required R Function(T data) onSuccess,
    required R Function(AppFailure failure) onFailure,
  }) => switch (this) {
    Success<T>(data: final data) => onSuccess(data),
    ResultFailure<T>(failure: final failure) => onFailure(failure),
  };

  bool get isSuccess => this is Success<T>;
  bool get isFailure => this is ResultFailure<T>;
}

final class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

final class ResultFailure<T> extends Result<T> {
  final AppFailure failure;
  const ResultFailure(this.failure);
}
```

- Data sources may throw typed transport/SDK/parsing exceptions. Repositories map expected operational exceptions once into `AppFailure`; repository/use-case callers receive `Result<T>`. Do not repeatedly wrap failures or swallow programming errors with blanket catches.
- Define the sealed `AppFailure` hierarchy in `core/error/app_failure.dart`, with a user-safe `message`. Reuse NetworkFailure, ServerFailure (optional statusCode), CacheFailure, AuthFailure, ValidationFailure, PaymentFailure, and UnknownFailure as needed. Add categories only for actionable distinctions, in the shared hierarchy.
- Treat cancellation as superseded work rather than a visible network error. Preserve diagnostic cause/stack through existing reporting without logging secrets or exposing raw server messages.
- Prefer named `fold(onSuccess: ..., onFailure: ...)` callbacks for consistency. Await the returned future when callbacks are async; do not silently lose their failures.

```dart
// Members of the Cubit; invalidate this generation on session/input changes too.
int _listingsGeneration = 0;

Future<void> loadListings() async {
  final generation = ++_listingsGeneration;
  final result = await getListingsUseCase();
  if (isClosed || generation != _listingsGeneration) return;
  result.fold(
    onSuccess: (listings) => emit(ListingsLoaded(listings)),
    onFailure: (failure) => emit(ListingsError(failure.message)),
  );
}
```

This read example guards Cubit emissions; any repository cache/persistence writes need their own identity boundary. It is not a write-cancellation or backend-idempotency mechanism.

## DI and backend boundaries

- New Bloc projects default to get_it in `core/di/injection.dart`; reuse Riverpod providers when they already own DI. Add injectable only when codegen has a concrete benefit.
- Choose singleton/session/route/factory lifetimes by ownership. Shared clients may be singletons; user caches/listeners need reset or disposal on logout. Do not make every repository a permanent singleton.
- Firebase/REST/Stripe calls belong behind data sources/repositories. Cloud Functions use a dedicated data source.
- Payment amount/currency/authorization/final status are server-controlled. The client collects/confirms using server-created intents; never ship server credentials or fulfill orders based only on client callbacks.

## Scaffolding and checks

Define contracts and state transitions, then implement the required interfaces, data layer, state owner, and widgets. When TDD is requested, test each behavior before implementation; use `flutter-tdd-workflow` if available. Neither skill requires empty layers.

Check affected callers, imports, DI registrations, cleanup, generated-code requirements, and error paths. Report SDK/generated-code limitations honestly.

## Maintenance

On an upgrade request or demonstrated incompatibility, compare pinned versions with official migration notes. Record reviewed versions in the task report; update this review date only after checking the guidance. Do not upgrade packages solely because a newer release exists.

Reference: [Flutter architecture recommendations](https://docs.flutter.dev/app-architecture/recommendations). The Bloc/Result/layer defaults here are personal conventions, not universal Flutter requirements.
