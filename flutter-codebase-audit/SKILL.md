---
name: flutter-codebase-audit
license: MIT
description: Review Flutter or Dart code for concrete security, correctness, async/state, performance, and maintainability issues. Use for audits and production-readiness reviews; implement fixes when the user also asks to fix them.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter Codebase Audit

Find demonstrated risks, including small user-visible defects. Review-only requests produce findings; audit-and-fix requests include scoped implementation and checking affected callers. Do not stop at a report when fixes are authorized.

Do not launch apps, servers, emulators, or device sessions unless explicitly requested. Focused static checks/tests are allowed subject to project instructions. Finish with physical-device verification steps and checks not performed.

## Map before judging

Read project instructions, SDK constraints, pubspec/lockfile, analyzer configuration, tests, and representative features. Identify navigation, state owners, DI lifetimes, backend boundaries, caches, and sessions. Assess actual contracts rather than imposing `flutter-architecture` on existing apps.

For cross-module/security tracing, follow the user's code-intelligence instructions when present. Verify navigation hints against source; do not silently create an index.

## Trace behavior and interleavings

Read the relevant sections of [the checklist](references/flutter-checklist.md). Trace each trigger through state owner, repository, I/O, completion, cache write, UI effect, and teardown; inspect callers too.

For async features in scope, explicitly consider:

- A starts, B starts, B finishes, A finishes: can A overwrite B's data, error, busy flag, or cache?
- Inputs/route change, logout, account switch, close/dispose, or reconnect occurs while pending: can old work still write or navigate?
- Two mutations, pagination plus refresh, or snapshot plus HTTP response overlap: can updates be lost, duplicated, or wrongly rolled back?
- Equality, in-place mutation, provider dependencies, or builder/listener filtering hide a valid change: can the UI stay stale despite changed data?

Use `flutter-async-state-safety` if available for deeper diagnosis/fixes. Otherwise document the timeline, intended ordering, and guard at each side-effect boundary. Mounted/isClosed checks, debounce, and transformers alone do not establish correctness.

Include small defects: stuck busy flags, nullable copyWith fields that cannot clear, repeated toasts/navigation, date boundaries, retry cursor gaps, unsafe parsing, and resources with the wrong owner. Follow evidence; do not manufacture a fixed number of findings.

## Evidence and priority

Every finding needs a real file/line, trigger/event order, concrete consequence, and suggested fix. Distinguish confirmed behavior from a hypothesis and name missing evidence. Never print credential values.

Order by actual severity/likelihood: security/data loss, crashes/correctness (including stale state), maintainability, performance, then optional style. A small correctness bug can outrank a large architecture concern.

These alone are not defects: long files, missing const/use-case/sealed state, no mocking library, public client keys, or absence of a particular image-cache package. Identify a demonstrated failure or label the point a convention suggestion. Missing coverage is a verification gap, not proof of a bug. Do not launch profiling without a request; label unmeasured performance hypotheses.

## Fix and verify when authorized

1. Group symptoms by root cause; fix the shared owner/boundary once. Preserve interfaces, architecture, validation, and successful behavior.
2. For non-trivial fixes, add a focused regression failing on old behavior. Control completion order with fakes/completers, not sleeps or live services. Cosmetic changes need no artificial tests.
3. Run narrow allowed checks. Prefer existing coverage/focused tests; use full `flutter test --coverage` when coverage analysis is relevant and permitted. Do not install packages just to make an audit appear complete.
4. Report fixed findings, remaining findings, and verification gaps separately. Missing backend rules or untested device behavior remain unverified.

Give specific manual steps and expected outcomes: repeated taps, changed filters mid-request, back navigation during load, airplane mode/reconnect, or account switch as relevant.

## Maintenance

Check official docs against installed SDK/package versions when semantics matter. Update guidance from verified behavior or official changes; keep personal conventions separate from correctness rules.
