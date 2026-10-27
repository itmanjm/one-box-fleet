# 04 — GitHub-Native Governance & Guardrails

> The agents do the work; the repo enforces the rules. Governance lives in branch protection, CI, AI review, and explicit human decision gates — not in anyone's good intentions.

## Branch model

```
main      ← protected, production, PR-only
develop   ← integration, deploys to staging
feature/* ← work branches off develop
```

- `main` is **PR-only**: no direct pushes, required reviews, required status checks, no force-push.
- Production promotion is always a PR — it's the audit record and the revert point in one.

## The guardrail stack (order matters)

1. **Pre-push verification** — the *author* (human or agent) runs tests/analysis locally and attaches real command output before pushing. No "should work" pushes. This is a workflow rule enforced by review culture, not tooling.
2. **CI on every PR** — build + tests + lints. Green or it doesn't merge.
3. **AI code review on every PR** — an automated first-pass reviewer (self-hosted, always-on) catches the mechanical issues before a human spends attention. A second AI reviewer (backed by a coding-model entitlement) handles deep dives. AI review is **advisory**: it informs humans, it never merges by itself.
4. **Human review** — a named person approves. Agents propose; humans dispose.
5. **Release gate** — "done" requires merge evidence **plus** post-deploy verification that the feature actually works in the target environment. Deploy-success ≠ feature-works.

## Human decision gates

Irreversible, public, or money-touching actions need explicit founder sign-off:

- Tracked as a **decision card** listing the exact decisions, the recommendation, and accept/override options.
- Gates **auto-expire** rather than blocking forever, and a user comment supersedes a stale gate (prevents zombie blockers and duplicate cards).
- The gate lives in the same surface the work lives in — chat thread or board card — not in a separate approval app.

## Secrets hygiene

- Tokens in `.env` files (chmod 600) or GitHub Secrets — never in code, docs, chat, or commit history.
- One credential per integration per agent where practical, so revoking one lane doesn't break the fleet.
- Audit the fleet after any suspected leak: `git log -p | grep <prefix>` beats hoping.

## Why this works with agents

LLM agents are eager, fast, and occasionally confidently wrong. The guardrail stack assumes **any single step can be wrong** and makes every step cheap to verify and cheap to revert. Branch protection catches bad merges. AI review catches bad diffs before humans burn attention. Decision gates catch bad *judgment*. Post-deploy verification catches everything that slipped through.
