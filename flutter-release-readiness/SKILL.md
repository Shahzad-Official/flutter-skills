---
name: flutter-release-readiness
description: Review or prepare Flutter Android/iOS release configuration, flavors, permissions, signing references, and device verification. Use for release/build readiness work, not as a mandatory checklist for every feature edit.
metadata:
  last-reviewed: '2026-10-28'
---

# Flutter Release Readiness

Inspect the target platform, SDK/toolchain constraints, pubspec/lockfile, Android/iOS configuration, flavors, and existing CI/release workflow. Limit work to the requested release/configuration scope; do not rotate credentials, change application IDs, or upgrade the toolchain as a side effect.

Do not launch apps, emulators, servers, or device sessions without an explicit request. Do not build, sign, upload, publish, or deploy merely because readiness is being reviewed. If those actions are explicitly requested, perform them within the specified scope and environment; otherwise give commands/steps for the user.

## Static release checks

- Match version/build numbers, application/bundle identifiers, flavor entrypoints, environment-specific API/Firebase config, URL schemes, and entitlement references to the intended distribution. Avoid silently targeting production from a development flavor.
- Inspect signing configuration references and ignore rules without printing private credentials. Verify release is not wired to debug signing where production signing is required; missing local signing material is a stated gap, not a reason to invent a key.
- Check required permissions/usage descriptions against actual feature use and platform requirements. Include denial/permanent-denial recovery and settings return behavior for changed permission flows.
- Trace deep links, notification taps, and auth callbacks through cold/warm start and session initialization where relevant. Deduplicate repeated delivery and validate destinations/parameters.
- Review environment/logging flags, bundled secrets/assets, insecure transport exceptions, error reporting redaction, and platform config required by installed plugins.
- Inspect existing release handling for obfuscation/symbol files and source maps when used. Do not claim symbols/upload/signing were verified from configuration alone.

Check current official platform/store requirements when they affect the task; do not hardcode changing target-SDK, Xcode, privacy, or submission deadlines into this skill.

## Validation and handoff

Run only permitted focused static checks/tests. If a build is requested, use the established flavor/entrypoint/signing path and report the exact artifact and limitations; a successful build does not prove device behavior or store acceptance.

Give a physical-device release verification sequence using the existing distribution workflow: fresh install and upgrade from a prior build (without deleting real user data), launch, login/logout, permission denial, background/resume, offline/reconnect, and affected deep-link/payment/notification flows. Use test accounts and sandbox payments where applicable. State expected behavior and anything requiring another platform/device.

Report ready items, concrete blockers, and checks not performed. Do not call a release production-ready when critical device/backend/signing evidence is missing.

## Maintenance

Recheck official requirements for each actual release task. Reference: [Flutter deployment](https://docs.flutter.dev/deployment), [Android release](https://docs.flutter.dev/deployment/android), [iOS release](https://docs.flutter.dev/deployment/ios). Update this review date only after reviewing guidance; do not imply a release was tested.
