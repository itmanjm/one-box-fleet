# 14 — Scheduled Jobs (the Cron Layer)

> Digests, scans, sweeps, and maintenance run on schedules. The pattern here keeps them from being the loudest silent failure in the stack.

## Job classes

| Class | Examples | Failure tolerance |
|---|---|---|
| **Reporting** | daily ops digest, weekly review | low — a missed digest is a dead-man's-switch event |
| **Sweeping** | stale-claim release, label hygiene, backup rotation | medium — retries safe, runs idempotent |
| **Scanning** | EOL/dependency scans, code-quality refresh, price monitors | low priority — never let these eat a rate budget needed by real work |
| **Maintenance** | container updates (watchtower), cert checks, disk pruning | careful — update the wrong service and you *are* the incident |

## The rules

1. **Idempotent or nothing.** Every job must be safe to run twice. A digest job that double-sends after a retry is worse than one that skips a day. Design: read state → compute → write, with the write deduped by a natural key (date + target).
2. **Catch-up after downtime.** If the box was off for 12 hours, does the 03:00 job run on boot? (systemd timers: `Persistent=true`. Cron: you'll miss it.) Decide per job — some reports *should* skip, some sweeps *must* run.
3. **Rate-limit awareness.** A tool-heavy scan job on a free-tier model lane will blow per-minute caps and either fail (silent gap) or starve interactive work. Match job weight to lane: heavy scans get the cheapest lane with sufficient quota, or get throttled/fanned out deliberately.
4. **Deliver results somewhere a human looks.** Job output routes to the team chat channel ([03](03-chat-gateways.md)) — a cron that writes to a log file nobody reads is theater.
5. **Evidence in the result.** "Scan done" is noise; "scan done: 0 EOL deps, 2 outdated (listed), 0 failures" is a report. Jobs attach the numbers.
6. **Dead-man's-switch.** The daily digest is also a heartbeat: if it *stops arriving*, the box is unhappy even if nothing alerted. Someone should notice absence within a day.

## Scheduling mechanics

- **systemd timers over cron**: journal logs, dependency ordering, `Persistent=true` catch-up, and `systemctl list-timers` as the one-view status page.
- Secrets come from the same `.env`/credential files as everything else ([12](12-security-hardening.md)) — never inline in the unit file.
- Every job logs its **exit code and duration**; a job that's been slowly getting slower (growing dataset, shrinking disk) shows up as a duration trend before it shows up as a failure.

## The audit

Quarterly: list every timer, and for each answer — what does it do, who reads its output, what does it cost (rate budget), and what breaks if it silently dies? A job that fails all four answers gets deleted.
