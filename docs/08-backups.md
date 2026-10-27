# 08 — Backups & Recovery

> If restoring is untested, you don't have backups — you have hope. Nightly automated backups, scheduled verification, and a one-command restore.

## What gets backed up

| Asset | Method | Why |
|---|---|---|
| Agent profiles (config, memory, skills, cron) | nightly archive | the fleet's brain |
| `.env` / credential files | nightly, encrypted, chmod 600 | everything else is rebuildable except these |
| Docker volumes (databases) | `docker run --rm -v ... tar` dumps | SonarQube, agent-memory pgvector, etc. |
| Compose files & infra scripts | git repo | infrastructure as code |
| Off-box copy | synced to second machine / cloud storage | a backup on the same disk is not a backup |

## The pattern

```bash
# nightly: archive + timestamp + off-box sync
tar -czf /backups/hermes-$(date +%F).tar.gz ~/.hermes/profiles/
rclone sync /backups remote:ops-backups --max-age 30d
```

Run as a systemd timer (not cron) so you get journal logs, dependency ordering, and `Persistent=true` catch-up after downtime.

## Restore discipline

- **Test restores quarterly:** pick one backup, restore to a scratch directory, diff against live. A restore you've never run is a rumor.
- **One-command restore:** a single script that takes a date and rebuilds profiles + volumes + services. If recovery needs tribal knowledge, it will fail at 2am.
- **Credential rotation on restore:** if you're restoring because of a compromise, rotate every secret that was in the backup *after* the restore, not before.

## The separate failure domain

The backup target must not share a failure domain with the primary: different machine at minimum, different region/cloud ideally. Our layout: ops box → personal-assistant box (different building) → cloud object storage.

## What we don't back up

Rebuildable-from-source state (git repos have origins, container images have registries). Back up *state*, reference *code*.
