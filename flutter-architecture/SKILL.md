---
name: flutter-architecture
description: Use when starting a new Flutter project or feature, deciding folder structure, choosing state management, setting up dependency injection, or asked to "set up the architecture", "scaffold a new feature", "structure this project", or "how should this be organized". Applies a consistent, proven architecture instead of improvising per project.
---

# Flutter Architecture Template

A single consistent architecture to apply across all Flutter client projects, so every new app/feature starts from the same proven structure instead of being designed from scratch each time.

## Layers (top to bottom)

1. **Presentation** — Widgets + state management (Bloc/Cubit or Riverpod notifiers). No direct API/Firestore calls here. Only calls into the domain layer.
2. **Domain** — Use cases / business logic. Pure Dart, no Flutter imports. This is what unit tests target directly.
3. **Data** — Repositories (interface + implementation) + data sources (Firestore, REST via Dio, Stripe SDK). Repository implementations depend on data sources; domain depends only on repository *interfaces*. Repository methods return `Result<T>` (see Error Handling Convention below) — never throw.
4. **Core** — Shared utilities: error types, network client setup, constants, theming, DI setup.

Dependency direction is always: Presentation → Domain → Data (interfaces only) ← Data (implementation). Never let Data or Core import from Presentation.

## Folder Structure (feature-first)

```
lib/
  core/
    error/            # Failure/Exception types
    network/           # Dio client, interceptors
    constants/
    theme/
    di/                 # get_it / riverpod provider setup
  features/
    <feature_name>/
      data/
        models/         # DTOs with toJson/fromJson
        repositories/   # Repository implementation
        datasources/    # Firestore/Dio/Stripe calls live here ONLY
      domain/
        entities/        # Plain Dart objects used by UI/logic
        repositories/     # Abstract repository interface
        usecases/          # One class per business operation
      presentation/
        bloc/ (or notifiers/)
        pages/
        widgets/
  app.dart
  main.dart
```

Rule of thumb: if you're about to write a Firestore/Dio call inside a widget or a Bloc event handler directly, stop — it belongs in `datasources/`, exposed through a repository.

## State Management Default

- **Default to Bloc/Cubit** for anything with multi-step async flows (auth, checkout, onboarding) — explicit states are worth the boilerplate for these.
- **Riverpod notifiers** are fine for simpler, more local UI state where Bloc's ceremony isn't worth it.
- Never mix more than one approach within a single feature. Mixing per-project is fine; mixing per-feature isn't.

## Error Handling Convention

No `dartz`/`Either`. Use a **custom sealed `Result<T>` type** with a `fold` method that takes named callbacks — never throw raw exceptions out of repositories or use cases.

### `Result<T>` (core/result/result.dart)

```dart
sealed class Result<T> {
  const Result();

  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(AppFailure failure) = ResultFailure<T>;

  R fold<R>({
    required R Function(T data) onSuccess,
    required R Function(AppFailure failure) onFailure,
  }) {
    return switch (this) {
      Success<T>(data: final data) => onSuccess(data),
      ResultFailure<T>(failure: final failure) => onFailure(failure),
    };
  }

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

Usage at every call site (repository, use case, Bloc) — always via `fold`, never pattern-matching ad hoc or null-checking:

```dart
final result = await getListingsUseCase();
result.fold(
  onSuccess: (listings) => emit(ListingsLoaded(listings)),
  onFailure: (failure) => emit(ListingsError(failure.message)),
);
```

### `AppFailure` — single sealed hierarchy, used app-wide

One definition, in `core/error/app_failure.dart`, reused by every feature — never redefine failure types per feature.

```dart
sealed class AppFailure {
  final String message;
  const AppFailure(this.message);
}

final class NetworkFailure extends AppFailure {
  const NetworkFailure([super.message = 'No internet connection']);
}

final class ServerFailure extends AppFailure {
  final int? statusCode;
  const ServerFailure(super.message, {this.statusCode});
}

final class CacheFailure extends AppFailure {
  const CacheFailure([super.message = 'Local cache error']);
}

final class AuthFailure extends AppFailure {
  const AuthFailure(super.message);
}

final class ValidationFailure extends AppFailure {
  const ValidationFailure(super.message);
}

final class PaymentFailure extends AppFailure {
  const PaymentFailure(super.message);
}

final class UnknownFailure extends AppFailure {
  const UnknownFailure([super.message = 'Something went wrong']);
}
```

Rules:
- Every data source/repository catch block maps the caught exception to one specific `AppFailure` subtype — never a bare `UnknownFailure` unless the exception genuinely doesn't match a known case.
- Add new `AppFailure` subtypes here, in this one file, as new failure categories appear (e.g. `RateLimitFailure`). Never declare a one-off failure class inside a feature folder.
- UI layer never sees raw exceptions — Blocs/Notifiers only ever receive `Result<T>` from use cases and `fold` it into UI state.
- UI state itself is still modeled as a sealed class (Loading / Loaded / Error / Empty), with the `Error` state holding the `AppFailure` (or just its `message`) — never inferred from nullable fields.

## Dependency Injection

- Use `get_it` registered in `core/di/injection.dart`, with `injectable` codegen if the project is large enough to justify it.
- Repositories and data sources are registered as singletons; use cases as factories.

## Backend Integration Notes (Firebase/Stripe specific)

- Firestore queries live in datasource classes, never inline in Blocs or widgets — makes swapping/mocking trivial for tests.
- Stripe: client only collects payment method/confirms intents created server-side (NestJS) — client never computes charge amounts.
- Cloud Functions calls go through a dedicated `CloudFunctionsDataSource`, not scattered `FirebaseFunctions.instance.httpsCallable(...)` calls.

## When Scaffolding a New Feature

1. Create the four folders (data/domain/presentation under core) even if some start nearly empty
2. Define the domain entity and repository interface first
3. Then the use case(s)
4. Then the data source + repository implementation
5. Then the Bloc/notifier
6. Then the widgets
7. This order matters for TDD — see the `flutter-tdd-workflow` skill, which assumes this same layering.
