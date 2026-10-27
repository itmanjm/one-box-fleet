# 06 — The Work Lifecycle

> Every piece of work — from a one-line fix to a new product — follows the same loop. The loop is the org chart.

## The loop

```
1. INTAKE & VETTING   research before build: market scan, feasibility, license check.
                      go/no-go written down — never assumed.
2. PRODUCT DEFINITION approved ideas become a spec/PRD: verifiable acceptance criteria,
                      decision rules with named owners. Ambiguity resolved HERE, not mid-build.
3. DECOMPOSITION      spec → cards on the board. Board = source of truth.
                      Each card: one lane owner (product / engineering / governance).
4. BUILD              branch off develop. Pre-push evidence gate (real command output).
5. REVIEW             PR: CI + AI review + human review. Nothing merges red.
6. STAGING VERIFY     merged code deploys to staging; the requester verifies it live.
7. PROMOTE            PR-only to main → deploy prod → post-deploy verification.
8. CLOSE WITH EVIDENCE card closes only with merge + deploy + verification evidence.
9. POST-LIVE LOOP     monitoring, digests, incident triage. Lessons written back into
                      the playbooks/skills so the next cycle starts smarter.
```

Two constants across every step: **staging-first** (never experiment in prod) and **evidence over claims** (nothing is done until verified live).

## How the lifecycle is gated

Between every stage sits a **gate**: a concrete, checkable condition that must be satisfied before work advances. Gates make the loop enforceable — an agent (or a rushed human) cannot skip a stage, because the next stage's tooling refuses unmet preconditions.

### The gate ladder

| # | Gate | Blocks | Enforced by | Who clears it |
|---|---|---|---|---|
| 1 | **Vetting gate** | idea → spec | a written go/no-go verdict with research citations exists | founder (or delegated owner) |
| 2 | **Spec gate** | spec → cards | numbered, verifiable acceptance criteria + named decision owners | product lane |
| 3 | **Pre-push gate** | code → PR | local tests/analysis run, real command output attached to the card | author (agent or human) |
| 4 | **CI gate** | PR → merge | required status checks green (branch protection refuses the merge button) | nobody — it's mechanical |
| 5 | **AI review gate** | PR → human review | automated reviewer comments triaged; blocking findings resolved | author |
| 6 | **Human review gate** | PR → merge | a named approver signs off | reviewer (never the author) |
| 7 | **Staging gate** | merge → promote | the requester verified the feature live in staging | the human who asked for it |
| 8 | **Promotion gate** | staging → prod | PR to `main` (protected; PR-only path, so this gate can't be bypassed) | maintainer merges |
| 9 | **Evidence gate** | deploy → done | post-deploy verification output (status codes, screenshots) attached to the card | author, witnessed by requester |
| 10 | **Decision gates** | any irreversible/public/paid action | explicit founder accept/override recorded | founder only |

### Decision-gate mechanics (the founder sign-off)

Stages 1, 7, and any irreversible action use **decision gates** — and their ergonomics decide whether governance actually gets used or gets routed around:

- **Surface brief.** A gate renders as a one-line ask plus optional detail — glanceable, so the founder sees *what* is being asked without reading a wall of text. Long context goes in a collapsible details section, never in the prompt line.
- **Accept = ratify the recommendation.** Gates carry the owner's recommendation (e.g. "Accept = all four options as recommended"), so a green light is one action, not four.
- **Override requires a reason.** Rejecting forces a written reason — it becomes the spec correction, not just a veto.
- **Auto-expiry, not zombie blocking.** A gate nobody answers expires after a window (the work parks as `waiting`, visible on the board) instead of silently blocking the lane forever.
- **Comments supersede.** A founder reply ("do D3 differently") retires the pending gate — a new card is stamped with the amended state. Nobody re-asks an already-answered question.
- **Re-arm on change.** If the underlying spec is revised while a gate is pending, the old gate is superseded by a new one matching the new revision — an accept of stale wording can't clear new content.

### Where gate state lives

The board is the gate dashboard. In a GitHub-native setup: issues are the cards, the project board's statuses are the lifecycle stages (`Intake → Specified → In Progress → In Review → Staging → Promotion → Done`), and gate state rides as **labels and fields**:

- `gate:founder` / `gate:staging` labels mark what's blocked and on whom
- a `waiting-days` counter field makes aging gates visible before they rot
- `blocked` + reason label for anything parked

Anyone glancing at the board sees exactly which lane is waiting on which human — that's the point. Governance you can't see gets bypassed; governance that's visible at a glance gets used.

### What happens on failure

- **Mechanical gate fails (CI/evidence):** work stops in place; the author fixes and re-runs. No human needed for the gate itself.
- **Review gate fails:** findings become the next iteration's card content — the review comments are the spec.
- **Decision gate expires/rejected:** work parks or redirects per the reason. Nothing advances on silence.
- **A gate was wrong:** the correction is written into the gate definition (the skill/playbook), so the same wrong gate doesn't fire twice. Gates learn too.

## What each stage prevents

| Stage | Prevents |
|---|---|
| 1 Vetting | building things nobody wants / can't legally ship |
| 2 PRD | the "that's not what I asked for" rebuild |
| 3 Cards | orphaned work with no owner |
| 4 Pre-push | broken main, review noise |
| 5 Review | bad diffs reaching users |
| 6 Staging | "works on my machine" releases |
| 7 Promote | untracked prod changes, unrevertable state |
| 8 Evidence | zombie tickets that are "probably fine" |
| 9 Post-live | repeating the same mistake |

## New-product example (the loop at full size)

1. **Intake:** founder sees a viral tool → trace the repo, check license (AGPL means: unmodified engine + proprietary wrapper is compliant; a closed fork is not), scan competitors.
2. **Definition:** PRD with numbered, verifiable acceptance criteria; founder decisions (pricing, brand, timing) explicit; a field-interview kit for validation.
3. **Cards:** proof build to engineering lane; interviews to product lane; decision gate to founder.
4–8. **Build → live** exactly as above, with the proof artifact doubling as the customer prop.
9. **Post-live:** market feedback re-enters intake as the next cycle's input.

## Scaling note

The loop is the same for a 1-person company and a 10-agent fleet — what changes is how many cards are in flight. Keep the board honest: one card, one owner, one verifiable outcome.
