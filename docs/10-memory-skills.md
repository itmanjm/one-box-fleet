# 10 — Memory & Skills

> The difference between an agent that repeats onboarding every session and one that compounds: persistent memory for facts, a skills library for procedures.

## Two stores, two jobs

| Store | Holds | Lifetime |
|---|---|---|
| **Memory** | who the user is, environment facts, standing conventions — declarative sentences ("User prefers verdict-first updates") | always injected, hard character budget |
| **Skills** | *how* to do recurring work: playbooks, exact commands, pitfalls, templates | loaded on demand when relevant |

The split rule: **facts that apply everywhere go in memory; procedures that apply to a task type go in skills.** A lesson learned mid-task ("promotion PRs conflict when main advances — resolve on a clean worktree, deletions win") belongs in the skill it came from, where it loads next time that work appears.

## Skills format

A skill is a directory with a `SKILL.md`: YAML frontmatter (`name`, a trigger-class `description` starting with "Use when…", version, license) and a body with Overview → When to Use → the actual procedure → Common Pitfalls → Verification Checklist.

```markdown
---
name: staging-deploys
description: Use when deploying any service to staging. Verifies the gate order and post-deploy checks.
version: 1.0.0
---

## Overview
...
## Common Pitfalls
1. ...
## Verification Checklist
- [ ] ...
```

## Compounding discipline

- **After every difficult/iterative task, write the lesson back.** If a skill was missing a step, had a wrong command, or hit a new pitfall — patch the skill before finishing the task. This is the single highest-leverage habit in the whole stack.
- **Skills are redacted.** No tokens, no private IPs, no client names in skills that might get shared.
- **Share the good ones.** A polished, self-contained skill publishes to GitHub as-is (see our [frontend-mobile-ui-design](https://github.com/itmanjm/frontend-mobile-ui-design) for the format).

## Memory hygiene

- Entries are **declarative facts, not instructions** — "Dre prefers numbered QA lists" ✓, "Always use numbered lists" ✗ (imperatives read as directives later and override current requests).
- **Budget is finite:** when full, consolidate stale entries rather than skipping saves.
- **Facts that change get corrected, not duplicated** — keep the history, supersede the value.
- Anything stale within a week (task progress, TODO state) stays in session history, not memory.

## Why this matters more than model choice

A mid-tier model with a sharp skills library and clean memory outperforms a frontier model re-deriving your conventions from zero every session. The knowledge layer is the moat.
