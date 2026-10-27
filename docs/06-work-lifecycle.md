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
