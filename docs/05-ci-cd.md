# 05 — CI/CD Pipeline

> Staging-first, PR-only production, self-hosted runners where it counts, and a deploy isn't done until it's verified live.

## The pipeline

```
feature branch
  → pre-push checks (local: tests + analysis, evidence attached)
  → PR to develop: CI (build/test/lint) + AI review + human review
  → merge → auto-deploy to STAGING
  → human verifies the feature live in staging
  → PR to main (the promotion PR)
  → merge → deploy PRODUCTION
  → post-deploy verification against prod (feature actually works)
```

## Self-hosted runners (Actions Runner Controller)

For heavy pipelines (mobile builds especially), hosted-runner minutes get expensive fast.

- Run **ARC on a small Kubernetes cluster** (k3s is enough), targeting the repos/lanes that need it.
- A repo **variable** (e.g. `USE_SELF_HOSTED_RUNNERS`) flips pipelines between hosted and self-hosted lanes — fallback to GitHub-hosted is one variable change, which is your outage plan.
- Real numbers from our lane: mobile PR checks went **~25 billed minutes → ~8 minutes** wall-clock, at zero marginal cost.

## Staging discipline

- Staging mirrors production config as closely as possible (same infra, test data).
- The human who asked for the feature verifies it in staging — not the author alone.
- **Fix-reporting rule:** live-only fixes get "retest now"; anything needing a client/app update gets a new build. Never blur the two.
- Firebase-specific: staging and production are **separate projects**, deploys target named Hosting sites, and promotions go through the PR.

## Post-deploy verification

Deploy success is a *server* fact; feature-works is a *user* fact. Verify the latter:

```bash
# example: verify an API guard in prod right after deploy
curl -s -o /dev/null -w '%{http_code}' https://api.example.com/members   # expect 401 unauthenticated
curl -s -o /dev/null -w '%{http_code}' -H "Authorization: Bearer $TOKEN" ... # expect 200 authorized
```

Attach the actual status codes to the release card. If the check fails, the rollback decision happens immediately — not after users report it.

## Promotion mechanics that bite

- **Promotion PRs go stale fast:** when `main` advances, an open develop→main PR starts conflicting. Resolve by merging `main` into `develop` on a clean worktree, and when a conflict is a "delete vs edit", the ratified deletion wins.
- **Config gates re-arm:** if a decision gate expired while you waited, supersede it with the new state instead of resurrecting the old card.
