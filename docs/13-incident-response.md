# 13 — Incident Response & Triage

> Doc [07](07-monitoring.md) tells you *when* something's wrong. This is what happens next: triage → duplicate check → root cause → fix → verify live → write back.

## The flow

```
ALERT / REPORT
  1. TRIAGE      what's broken, from the user's perspective? severity?
  2. DEDUPE      is this already known/tracked? close-or-merge duplicates fast.
  3. CONTAIN     is damage still happening? (rotate creds, stop the job, roll back)
  4. ROOT CAUSE  reproduce it deterministically before touching the fix.
  5. FIX         smallest change that removes the cause — staged, never hot in prod.
  6. VERIFY LIVE the fix works in the environment users touch. Evidence attached.
  7. WRITE BACK  the lesson lands in the playbook/skill so the class of incident dies.
```

## Triage rules

- **Severity = user impact × blast radius.** "Login broken for everyone" beats "report page renders slowly" regardless of how interesting the latter is. One severity scale, written down, no relitigating per incident.
- **Dedupe before debugging.** Check open issues/cards for the same symptom. Two agents (or an agent and a human) independently debugging the same bug is the classic multi-agent failure — the *claim* on the card prevents it (see [01](01-agent-harness.md)).
- **Contain before you understand.** If damage is ongoing (bad deploy live, credentials leaking, job in a mutation loop): roll back, revoke, stop. Root cause can wait an hour; the bleeding can't. Staging-first + PR-only prod means **rollback is always available**: revert the promotion PR, redeploy the previous known-good.

## Root-cause discipline

- **Deterministic repro or it didn't happen.** "It failed once in the logs" gets a repro attempt with the *actual* payload/conditions, not a theory. If you can't reproduce, you're guessing — instrument, don't patch.
- **Verify before claiming fixed.** The report isn't "fix pushed" — it's "retested now, works" with fresh evidence (status codes, screenshots, logs). Live-only fixes get "retest now"; anything needing a new build gets a new build (see [05](05-ci-cd.md)).
- **Check the boring causes first:** expired credentials, filled disk, exhausted quota, a service that watchtower auto-updated into an API change. Most "weird" incidents are one of these.

## Write-back (step 7 — the one everyone skips)

Every incident ends with one of:
- a **playbook/skill patch** so the failure class is prevented ("never write plausible audit IDs into FK'd tables")
- a **monitoring addition** so the next occurrence alerts instead of waiting for a user
- a **guardrail change** if a gate should have caught it (see [04](04-governance.md), [06](06-work-lifecycle.md))

No write-back, no close. The incident isn't over when it's fixed; it's over when it can't recur the same way.

## Post-incident review (for sev-1s only)

Five questions, thirty minutes, written down: what happened, what was the user impact, what was the root cause, what gates missed it, what changes. No blame — the system is the suspect.
