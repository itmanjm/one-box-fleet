# 07 — Monitoring & Observability

> Know before your users tell you. Three layers: a container dashboard, uptime checks on public surfaces, and a scheduled digest that lands in chat.

## Layer 1 — Container dashboard (Portainer)

```bash
docker volume create portainer_data
docker run -d --name portainer --restart=unless-stopped \
  -p 100.x.y.z:9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:lts
```

- **Bind to the tailnet/private IP only** (`-p 100.x.y.z:9443:9443`, not `-p 9443:9443`) — the dashboard can control every container; it must never face the internet.
- Create the admin account promptly — Portainer's first-run window times out (restart the container to reopen it).
- You get: per-container health/stats, logs, consoles, image/volume views.

## Layer 2 — Uptime checks (Uptime Kuma)

- Self-hosted, one container, monitors every public URL and heartbeat endpoint.
- **Monitor the user path, not the process.** "Container running" ≠ "feature works." Check the actual URL an end user hits, and expect the status code you documented.
- Notifications go to the same chat channels the team already uses.

## Layer 3 — Scheduled digests

A cron job (per agent profile) that compiles state into a chat message on a schedule:

- What ran since last digest (cron jobs, deploys, merges)
- What failed and why (with the real error text, not "it errored")
- What's waiting on a human (decision gates, reviews)
- Disk/memory pressure trending

The digest is also a **dead-man's switch**: if the digest stops arriving, the box is unhappy even if nothing alerted.

## Verification discipline

- An alert's green check is configuration-proven, not delivery-proven: after setting up any monitor, **trigger the condition once** (or use a known-good/bad pair) and confirm the notification actually arrives in the actual channel.
- Probe both sides of an auth wall: an unauthenticated 302 proves the login page loads, not that the app behind it works.

## What we monitor

| Target | Check | Why |
|---|---|---|
| Public app URLs | HTTP 200 + content sniff | user path |
| Tunnel endpoints | HTTPS + body | edge is up ≠ origin is up |
| Cron jobs | completion + exit codes | silent failure is the default |
| Disk usage | threshold | Docker build cache fills disks quietly |
| Agent gateways | process + heartbeat | a dead gateway looks like silence, not error |
