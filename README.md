# Flutter Skills

A personal collection of AI coding-agent skills for building, reviewing, and maintaining Flutter apps.

> **These skills reflect my personal engineering preferences and working methods. They are not a universal Flutter standard. Read the skills before using them. If the approach fits how you work, use the relevant parts and adapt them to your app.**

Different developers and teams make different architecture decisions for good reasons. Project size, business requirements, existing code, and team experience all affect what makes sense. You do not need to adopt my entire stack or agree with every choice to find a skill useful.

Have an improvement in mind? [Contributions and feedback are welcome](#contributing), including approaches different from my own.

## Contents

- [What these skills target](#what-these-skills-target)
- [My preferred structure and defaults](#my-preferred-structure-and-defaults)
- [Available skills](#available-skills)
- [Before using them in your app](#before-using-them-in-your-app)
- [Quick start](#quick-start)
- [Making the skills available to your agent](#making-the-skills-available-to-your-agent)
- [Example requests](#example-requests)
- [How review and verification should work](#how-review-and-verification-should-work)
- [Keeping the collection current](#keeping-the-collection-current)
- [Contributing](#contributing)
- [License](#license)

## What these skills target

The main focus is backend-connected Android and iOS product apps, such as:

- Booking and service apps.
- E-commerce and marketplace apps.
- Business tools, customer portals, and inventory apps.
- Content and community apps.
- Apps with authentication, forms, search, pagination, uploads, and realtime updates.

Some guidance also applies to smaller apps, Flutter web, and desktop, but this collection does not provide specialized workflows for every platform or product type. Games, graphics-heavy tools, hardware integrations, and complex offline-first systems may need additional guidance.

## My preferred structure and defaults

For new projects, I prefer a feature-first structure with clear responsibilities:

```text
lib/
  app/
    di/injection.dart      # app composition root
  core/                    # shared contracts, networking, theme
    di/core_module.dart    # shared infrastructure registrations
  features/
    auth/
      di/auth_module.dart  # feature-owned registrations
      data/                # data sources, DTOs, repository implementations
      domain/              # entities, repository interfaces, business rules
      presentation/        # screens, widgets, state management
    bookings/
    profile/
```

The source dependency direction is `Presentation → Domain ← Data`. Use cases and separate models should earn their place; a small feature does not need empty folders or layers that only forward calls.

DI wiring sits outside that business-layer dependency direction. Each feature registers its own dependencies; the app composition root coordinates those modules and shared infrastructure. Wiring modules can import the concrete classes they construct, including Blocs/Cubits. This does not allow ordinary Domain/Data/Core infrastructure to depend on Presentation. Existing projects can keep a clearly identified composition root under `core/di`; relocating it is not required. Feature modules can share one app locator, with lifetimes/scopes chosen by ownership.

| Area | My default |
| --- | --- |
| State management | Bloc/Cubit for new multi-step async flows; Riverpod is an alternative when chosen for the project. |
| Local UI state | Keep focus, animations, controllers, and simple widget state local where appropriate. |
| Error handling | A custom sealed `Result<T>` and shared `AppFailure` hierarchy for new projects. The examples require Dart 3. |
| Dependency injection | `get_it` with feature-owned registrations and app-level composition for new Bloc projects; existing Riverpod DI stays in place. |
| Backend access | Keep external I/O behind data sources and repositories. Use Firebase only where the app actually uses it. |
| Existing apps | Reuse established components, dependencies, interfaces, and sound conventions. Migrations are separate work. |
| Implementation | Prefer the smallest complete fix, with validation and data integrity preserved. |

These are defaults, not reasons to rewrite a working app. A different state manager, error representation, or folder structure is not automatically a defect.

## Available skills

Each folder contains a `SKILL.md` entrypoint. Some include references for more detailed checks.

| Skill | Use it for |
| --- | --- |
| [flutter-architecture](flutter-architecture/SKILL.md) | New project/feature structure, layer boundaries, state ownership, DI, and error contracts. |
| [flutter-codebase-audit](flutter-codebase-audit/SKILL.md) | Evidence-based security, correctness, async/state, maintainability, and performance reviews; scoped fixes when requested. |
| [flutter-async-state-safety](flutter-async-state-safety/SKILL.md) | Race conditions, stale UI/caches, overlapping requests, lifecycle problems, and optimistic-update bugs. |
| [flutter-tdd-workflow](flutter-tdd-workflow/SKILL.md) | Red-green-refactor when TDD is requested or required, including deterministic async regressions. |
| [flutter-api-auth](flutter-api-auth/SKILL.md) | API boundaries, token refresh, retries, cancellation, pagination, and session isolation. |
| [flutter-ui-quality](flutter-ui-quality/SKILL.md) | Responsive layouts, forms, keyboard/focus handling, accessibility, RTL, and recovery states. |
| [flutter-firebase-workflows](flutter-firebase-workflows/SKILL.md) | Firebase Auth, Firestore, Storage, rules boundaries, offline behavior, listeners, and concurrent writes. |
| [flutter-release-readiness](flutter-release-readiness/SKILL.md) | Android/iOS release configuration, flavors, permissions, signing references, and device verification. |

The UI skill focuses on implementation quality and usability. Visual identity, art direction, and a complete design system still depend on the app's design requirements.

## Before using them in your app

1. **Read the relevant skill.** Check its assumptions and any linked references, not just its name.
2. **Compare it with your project.** Look at your architecture, SDK/package versions, backend, team conventions, and project instructions.
3. **Choose what fits.** You can use the audit or async workflow without adopting the architecture template. Firebase and release guidance apply when those areas are in scope.
4. **Adapt personal preferences.** Keep the correctness checks, but adjust stack and organizational choices to your app. Record project-specific decisions in your own instructions.
5. **Try a focused task first.** Review the agent's reasoning, diff, and verification before using the workflow more broadly.

A skill should support your project and the task you requested. It should not become an excuse for unrelated refactoring, dependency changes, or a larger architecture migration.

## Quick start

### 1. Get a local copy

```bash
git clone https://github.com/Shahzad-Official/flutter-skills.git
cd flutter-skills
```

If you already have a local copy, use it instead. Review incoming changes and preserve your local adaptations before updating it.

### 2. Read and select a skill

Start with the [available skills](#available-skills) table, then open the selected `SKILL.md` and its relevant references. Pick skills for the task at hand; you do not need to enable the whole collection.

For example, a stale-search-results bug may need `flutter-async-state-safety` without the architecture or release skills.

### 3. Point your agent at the selected instructions

If your agent can read local files, give it the actual path to the skill along with your app and task. Replace the example paths below with your own:

```text
Project: /path/to/my_flutter_app

Read /path/to/flutter-skills/flutter-async-state-safety/SKILL.md
and any relevant references before investigating this bug:
older search results sometimes replace results for my latest query.

Follow this project's instructions and existing conventions.
Do not launch the app, servers, emulators, or device sessions.
Explain the cause, make the smallest complete fix, run permitted focused
checks, and give me manual verification steps and expected outcomes.
```

This is an explicit instruction to read a file. For native skill discovery, follow the installation guidance below. If your agent cannot access the local repository, use its supported skill import mechanism instead.

### 4. Review the result before adopting it more broadly

Check that the agent used the intended skill, stayed within the task, preserved your project's conventions, and explained what it verified. Review the diff and test the affected behavior in your own environment.

## Making the skills available to your agent

Use your agent's supported skill installation/discovery mechanism, or explicitly ask it to read a selected `SKILL.md` from this repository when it supports local-file instructions.

Keep the complete skill folder, including its `references/` directory where present. Cloning this repository alone does not guarantee that your agent will discover or activate its skills; check your agent's documentation and confirm which instructions it loaded.

These are agent instructions, not Flutter packages. They do not belong in `pubspec.yaml` and do not add a runtime dependency to your app.

### Codex: project-local installation

Codex discovers repository skills in `.agents/skills`. For a first-time installation on macOS/Linux, run this from your Flutter project root after reviewing the selected skill. Replace the source path with your local clone; this example assumes the destination skill folder does not already exist. Compare any existing copy before replacing it. [Official OpenAI documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)

```bash
mkdir -p .agents/skills
cp -R "/path/to/flutter-skills/flutter-async-state-safety" .agents/skills/
cp "/path/to/flutter-skills/LICENSE" .agents/skills/flutter-async-state-safety/LICENSE
```

The installed entrypoint is `.agents/skills/flutter-async-state-safety/SKILL.md`, with its references and license alongside it. Copy other selected skill folders the same way. For personal use across repositories, Codex also supports `~/.agents/skills`; choose the intended scope rather than installing duplicate names at both locations.

Codex detects skill changes automatically; restart it if a new skill does not appear. Verify it is listed in the skill selector, then explicitly invoke it with a focused request:

```text
$flutter-async-state-safety Investigate and fix stale search results in this
project. Preserve existing conventions and do not launch the app.
```

This documents the supported discovery layout; it does not install skills into your projects automatically. Copied skills are snapshots, so review and apply upstream changes deliberately.

## Example requests

After making the relevant skills available:

**Architecture**

> Use flutter-architecture to structure this new booking feature. Read the neighboring features first and preserve the project's existing conventions.

**Audit and fix**

> Use flutter-codebase-audit to review this feature and fix confirmed issues. Include file/line evidence, focused regression checks, and anything that remains unverified.

**Race conditions and stale state**

> Use flutter-async-state-safety to investigate this search flow. An older request sometimes replaces newer results. Check success, failure, loading flags, cache writes, and what happens after navigation or logout.

**TDD**

> Use flutter-tdd-workflow to implement this behavior test-first. Control async completion order in the tests instead of relying on real delays.

For my usual execution preferences, include:

> Do not launch the app, servers, emulators, or device sessions. Run permitted focused static checks and tests, then give me physical-device verification steps and expected outcomes.

## How review and verification should work

The skills ask the agent to inspect real code, trace affected callers, identify a concrete failure, and fix its root cause when fixes are authorized. A review-only request should remain a review.

Async checks go beyond `mounted` or `isClosed`: they cover older results and errors, stale `finally` blocks, session changes, pagination/refresh overlap, and optimistic rollback. Smaller defects such as stuck loading, nullable fields that cannot clear, repeated navigation, and incomplete equality are included too.

Non-trivial fixes should have focused behavior checks where allowed. TDD is a separate choice; it is not forced on every task. App launches, live integrations, builds, and releases follow the task's explicit scope and project instructions.

**Skills guide an agent; they do not guarantee correct code, complete coverage, security, or production readiness.** Review generated changes and verify the important behavior in your own environment. Fake-based tests do not prove deployed backend rules or real device behavior. If a check was not run, the agent should say so and provide practical manual steps.

## Keeping the collection current

Each skill records a `last-reviewed` date and includes maintenance guidance. Relevant skills link to official documentation.

- Check APIs and migration guidance against the versions your project actually uses.
- Update skills when a verified change or a real failure shows that their guidance needs correction.
- Keep personal conventions separate from framework requirements and demonstrated bugs.
- Review local adaptations when pulling updates into your own copy.

A review date is not a compatibility guarantee for every SDK/package combination. This repository does not automatically monitor releases, update installed copies, or upgrade your app's dependencies.

## Contributing

You're very welcome to contribute if you think a skill can be improved, updated, or expanded. These started with my personal preferences, but I'd like to learn from other Flutter developers and the problems they encounter in real apps.

You do not have to share all my architecture choices to contribute. Different approaches are welcome when they explain the use case, trade-offs, and how they fit alongside the existing guidance.

### Ways to help

- **Report a gap or mistake:** a missed race condition, stale-state bug, unclear instruction, or workflow that led an agent to make the wrong change.
- **Update outdated guidance:** include the affected SDK/package versions and relevant official migration notes or documentation.
- **Improve an existing skill:** clearer triggers, better diagnosis, more useful regression scenarios, or fewer unnecessary steps.
- **Suggest a new skill:** describe a distinct, recurring Flutter workflow and why the current skills do not cover it well.
- **Share practical feedback:** what worked in your app, what you adapted, and where the instructions fell short. Documentation fixes and small corrections are welcome too.

### Opening an issue or pull request

Start with an [issue](https://github.com/Shahzad-Official/flutter-skills/issues) for feedback, a bug report, or an idea. If you already have a focused improvement, you can [open a pull request](https://github.com/Shahzad-Official/flutter-skills/pulls).

For a useful report or PR:

1. Name the affected skill and explain the problem or proposed improvement. A small, sanitized example of expected versus actual behavior is especially helpful; leave out credentials, private code, and personal data.
2. Explain the reasoning and any relevant version constraints. Separate personal preferences from framework requirements or demonstrated correctness issues.
3. Keep the change focused, update related references when needed, and say how you checked it. If something was not tested, say so. Check skill metadata and relative links for changed or new skills.

For a larger change to the collection's defaults or scope, an issue explaining the proposal first helps others discuss the trade-offs before implementation.

If this collection is useful to you, a star or sharing it with another Flutter developer helps others find it. Thanks for trying it and helping make it better.

## License

This collection is licensed under the [MIT License](LICENSE). Include the license and copyright notice when redistributing the collection or individual skills.
