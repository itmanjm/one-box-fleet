# 15 — App & Mobile Release Pipeline

> The product-shipping path on top of the CI/CD base ([05](05-ci-cd.md)): web apps on Firebase, mobile apps through signed builds to the store.

## Web apps (Firebase)

- **Two projects, hard-split:** `app-staging` and `app` (prod). Separate Auth, Firestore, Functions, Hosting. A staging deploy can never touch prod data — different project, not different site.
- **Hosting sites pre-created.** Newer Firebase projects must create Hosting sites explicitly before first deploy. The bootstrap guardrail: `hosting:sites:list` → `create` only if absent → verify → deploy. Idempotent, one site per `firebase.json` target, full real project ID (never an alias).
- Deploys target **named sites** in CI — never "the default site".
- Auth/Firestore changes follow the same gates as code: rules changes are diffs, reviewed, staged, verified before promotion.

## Mobile apps (Android)

```
feature → PR (CI + AI review + human) → merge develop
  → staging APK build (self-hosted runner) → UAT install → verify
  → release build: version-bumped, signed with the release keystore
  → internal testing track → verify on store build → promote to production
```

- **Versioning is a gate:** every release build gets an explicit version bump (no two store uploads with the same versionCode). CI fails the duplicate.
- **Signing:** the release keystore lives off-repo in encrypted backup ([08](08-backups.md)) with its own credential handling. CI gets it via secrets at build time; it is never in the repo, and losing it is losing the app — test the keystore restore path.
- **Self-hosted runners** do the heavy Gradle work ([05](05-ci-cd.md)); distribution artifacts (APK/AAB) upload to the store track — or to Drive with link-sharing for UAT when a store round-trip isn't needed.
- **Distribute-and-forget is not delivery.** An APK link is "done" when a real device has installed and exercised the build — evidence: version string from the installed app, the flow exercised on screen.

## The verification gate (both platforms)

Same rule as everywhere: **deploy-success ≠ feature-works.**

- Web: hit the live endpoints post-deploy, check status codes + body, attach to the release card.
- Mobile: install the store/internal build on a physical device, exercise the changed flow, screenshot the version + the working feature.

## Promotion hygiene

- The release card carries: merge evidence, staging verification, artifact (build ID/version), post-deploy verification. Card closes with all four ([06](06-work-lifecycle.md) gates 7–9).
- Store releases follow the store's own track discipline (internal → closed → production) — each track is itself a gate with the same verify-live rule.
