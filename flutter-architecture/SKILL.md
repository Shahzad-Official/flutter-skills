---
name: flutter-architecture
license: MIT
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

Source dependencies for application logic: `Presentation → Domain ← Data`. Repository interfaces belong in Domain; Data implements them. Domain can depend on pure-Dart Core contracts, never Flutter-dependent utilities. Data and shared Core infrastructure must not import Presentation. Composition-only DI modules are the explicit exception: they may import concrete implementations and presentation state owners to wire dependencies, without containing business logic.

```text
lib/
  app/
    di/
      injection.dart # app composition root: coordinates registrations
  core/
    error/           # AppFailure
    result/          # Result<T>
    network/
    constants/
    theme/
    di/
      core_module.dart # shared infrastructure registrations only
  features/
    <feature_name>/
      di/
        feature_module.dart # this feature's registrations
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

### App composition and feature-level DI

- New Bloc projects default to get_it with an app composition root in `app/di/injection.dart`. It coordinates shared registrations from `core/di/core_module.dart` and each feature's `features/<feature>/di/<feature>_module.dart`. Register dependencies in the required order and complete necessary async initialization before resolving consumers.
- Each feature owns registration of its data sources, repository interface-to-implementation bindings, use cases, and Bloc/Cubit factories. Its DI module may import that feature's Data, Domain, and Presentation types for wiring. Keep business logic out of registration files and inject dependencies into the classes being constructed.
- `core/di` registers shared infrastructure; it does not become a registry of feature state owners. App/feature DI modules sit outside the business-layer dependency direction. Domain, data sources, and repositories must not import those modules or resolve their own dependencies from a global locator. Feature modules must not call each other to build hidden dependency cycles; app composition coordinates shared contracts.
- Existing projects may keep their composition root at `core/di/injection.dart` or another established location. Mark that file (and any dedicated composition-only helpers) as wiring code allowed to import feature modules or presentation state owners. This exception does not permit ordinary Core/Data code to depend on Presentation; do not move files solely to match the new-project template.
- Feature-level registration is code organization, not a requirement for an independent container per feature. Reuse the app locator; add session/route scopes only when ownership requires them. Register each binding once per intended scope and release that scope's owned resources when it ends.
- Choose singleton/session/route/factory lifetimes by ownership. Shared clients may be singletons; user caches/listeners need reset or disposal on logout, with old async work invalidated before it can repopulate them. Route-owned Blocs/Cubits usually use factories. `BlocProvider(create: ...)` owns closing a newly created instance; `BlocProvider.value` does not own closing an existing one. Avoid shared instances closed by a borrower or duplicate disposal by DI and UI.
- Reuse Riverpod providers when they already own DI; preserve their feature organization and lifetimes instead of adding a get_it container beside them. Add injectable only when codegen has a concrete benefit.

### Backend boundaries

- Firebase/REST/Stripe calls belong behind data sources/repositories. Cloud Functions use a dedicated data source.
- Payment amount/currency/authorization/final status are server-controlled. The client collects/confirms using server-created intents; never ship server credentials or fulfill orders based only on client callbacks.

## Scaffolding and checks

Define contracts and state transitions, then implement the required interfaces, data layer, state owner, and widgets. When TDD is requested, test each behavior before implementation; use `flutter-tdd-workflow` if available. Neither skill requires empty layers.

Check affected callers, imports, the app root's feature registration calls, duplicate/missing bindings, dependency readiness, ownership/disposal, generated-code requirements, and error paths. Report SDK/generated-code limitations honestly.

## Maintenance

On an upgrade request or demonstrated incompatibility, compare pinned versions with official migration notes. Record reviewed versions in the task report; update this review date only after checking the guidance. Do not upgrade packages solely because a newer release exists.

Reference: [Flutter architecture recommendations](https://docs.flutter.dev/app-architecture/recommendations). The Bloc/Result/layer defaults here are personal conventions, not universal Flutter requirements.

DI API references: [get_it registrations and scopes](https://pub.dev/packages/get_it), [BlocProvider ownership](https://pub.dev/documentation/flutter_bloc/latest/flutter_bloc/BlocProvider-class.html). Check the project's pinned versions; the module layout above is a personal organization preference.
