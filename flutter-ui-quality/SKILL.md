---
name: flutter-ui-quality
description: Build or fix Flutter screen behavior, responsive layouts, forms, keyboard/focus handling, accessibility, localization, and loading/error/empty states. Use for UI work and concrete UI defects, reusing the app's design system.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter UI Quality

Read the screen, neighboring components, theme/tokens, localization, navigation, and state contracts first. Reuse existing controls and dependencies; fix the relevant layout/state owner rather than introducing a second design system.

Do not launch apps, emulators, servers, or device sessions without an explicit request. Use focused analyzer/widget checks when allowed. End with device steps and disclose visual behavior not inspected.

## Implement complete interactions

- Handle initial loading, empty, content, error/retry, refresh, pagination, and submission only where relevant. Preserve useful content on refresh failure and avoid a single busy flag blocking unrelated actions.
- Bind controls to the actual state owner. Prevent duplicate submissions at the operation boundary as well as the button, while retaining a clear retry path.
- For forms, validate before submission, show actionable field errors, manage focus/keyboard actions, preserve entered values across recoverable failures, and make nullable clearing intentional.
- Keep requests/subscriptions out of build and other repeatable render paths. Replace subscriptions/controllers correctly when inputs change; dispose only resources this widget owns.
- Navigation, dialogs, and toasts should follow the existing side-effect mechanism, check the exact context lifetime after awaits, and avoid repeated delivery on rebuild or route return.
- When symptoms involve stale state, trace equality, mutable models, selectors, and async identity before adding keys, force-refresh calls, or setState workarounds. Use `flutter-async-state-safety` when available.

## Layout and access

- Design from available constraints (including split screen), not a guessed phone model. Avoid fixed heights around dynamic text; use scrolling/wrapping/flex intentionally and keep long lists lazy when needed.
- Account for safe areas, keyboard insets, notches, landscape, and nested-scroll/unbounded constraints. Keep the primary action reachable with the keyboard open.
- Verify large text/display scaling, long localized labels, RTL/directional padding/icons, locale-aware dates/numbers, and light/dark themes supported by the app. Do not hardcode English into an existing localization flow.
- Preserve semantics, meaningful labels for icon-only actions, focus traversal, contrast, and accessible interaction sizes. Decorative assets should not add screen-reader noise. Avoid communicating status by color alone.
- Size decoded images appropriately, provide placeholders/error recovery, and preserve aspect ratio. Reuse existing caching; a new image package is not an automatic requirement.

## Verification

Use focused widget tests for non-trivial changed behavior: form validation/retry, navigation effect once, explicit field clearing, large-text/RTL layout, or accessible interaction. Avoid broad golden suites for trivial spacing/copy changes unless the project requires them.

Give the user a physical-device checklist suited to the edit: smallest supported window, large text, keyboard open, rotation, supported RTL locale, TalkBack/VoiceOver, and failure/retry. State expected outcomes, such as no overflow, reachable submit, preserved input, and one action per tap sequence. Do not claim accessibility or appearance was verified solely from source.

## Maintenance

Check SDK/package APIs against the project versions. References: [adaptive/responsive Flutter UI](https://docs.flutter.dev/ui/adaptive-responsive), [accessibility](https://docs.flutter.dev/ui/accessibility), [accessibility testing](https://docs.flutter.dev/ui/accessibility/accessibility-testing). Update for relevant verified changes without rewriting the app's components.
