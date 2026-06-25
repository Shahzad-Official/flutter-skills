---
name: flutter-codebase-audit
description: Use when reviewing, auditing, or doing a senior-level code review of an existing Flutter/Dart codebase or repository — including phrases like "audit this repo", "review this Flutter project", "is this code production-ready", "find issues in this codebase", or "what would a senior dev flag here". Produces a prioritized, actionable report rather than scattered comments.
---

# Flutter Codebase Audit

A repeatable senior-dev-level review process for existing Flutter codebases. Goal: surface real risks (bugs, security, performance, maintainability) in priority order — not style nitpicks.

## How It Works

1. **Map the repo first** — before judging anything, understand structure:
   - Folder structure (feature-first vs layer-first vs flat)
   - State management approach in use (Bloc, Riverpod, Provider, GetX, setState)
   - Navigation approach (Navigator 1.0, go_router, auto_route)
   - DI approach (get_it, riverpod providers, manual)
   - Backend integration pattern (REST client, Firebase SDK direct calls, repository layer)
   - Read `pubspec.yaml` to understand the real dependency footprint and Dart/Flutter SDK constraints

2. **Check separation of concerns**
   - Are widgets doing business logic / API calls directly inside `build()` or button callbacks?
   - Is there a repository/data layer, or are Firestore/Dio calls scattered across UI files?
   - Are models doing serialization themselves, or is that leaking into UI/business logic?

3. **Scan for known Flutter risk patterns** (see `references/flutter-checklist.md` for the full list) — prioritize:
   - Memory leaks: undisposed `AnimationController`, `StreamSubscription`, `TextEditingController`
   - `setState` after dispose / unmounted widget errors
   - Missing `const` constructors causing avoidable rebuilds
   - Unbounded rebuilds: whole-screen `Consumer`/`BlocBuilder` instead of scoped widgets
   - Hardcoded API keys, Stripe secret keys, or Firebase service account data committed in code
   - No error/loading/empty state handling on async UI (sealed/freezed state classes missing)
   - Null-safety footguns: excessive `!` bang operators instead of proper null handling
   - God widgets/files (500+ line widgets, files mixing 3+ responsibilities)

4. **Check testability and test coverage**
   - Is business logic (Blocs/Cubits/notifiers, repositories, use cases) unit-testable in isolation, or tightly coupled to Flutter widgets/BuildContext?
   - Run `flutter test --coverage` if a test suite exists; note critical paths (payments, auth, data sync) with zero coverage
   - Flag any business logic that can't be tested without spinning up a widget tree

5. **Check performance basics**
   - `ListView.builder`/`GridView.builder` used for long lists, not `ListView(children: [...])`
   - Images: are they sized/cached (`cached_network_image`) or loaded raw and full-resolution?
   - Heavy work (JSON parsing, image processing) on the main thread vs `compute()`/isolates

6. **Produce the report** in this exact priority order, every time:
   1. 🔴 Security / data-loss risks (leaked keys, unguarded payment flows, unsafe storage of tokens)
   2. 🟠 Bugs / crash risks (memory leaks, null-safety issues, race conditions)
   3. 🟡 Architecture/maintainability debt (god widgets, missing layering, tight coupling)
   4. 🟢 Performance optimizations
   5. ⚪ Style/convention nitpicks — keep this section short, link don't elaborate

   For each finding: **file + line reference**, **why it matters** (1 sentence, concrete consequence not theory), **suggested fix**. Don't pad findings with generic advice — every line should be specific to what's actually in the code.

## Usage

Trigger this skill, then work through the steps above against the actual repo before writing anything. Do not generate findings from memory of "common Flutter mistakes" without having actually located them in the code — every finding must point to a real file/line.

See `references/flutter-checklist.md` for the full pattern list to scan against.
