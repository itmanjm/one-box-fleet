# 00 — The Tool Stack at a Glance

> Every named tool in this playbook, what it does for us, and why we picked self-hosted vs hosted for each.

## The stack

| Tool | Role | Hosted here because |
|---|---|---|
| **Hermes Agent** | the agent harness: profiles, gateways, skills, memory, cron | open runtime, local-first, no per-seat fees |
| **Tailscale** | private mesh VPN — every service binds to tailnet IPs | zero exposed ports; ISP/NAT proof |
| **Docker + Compose** | service isolation for everything below | boring, reproducible, one YAML per stack |
| **Portainer** | container dashboard (health, logs, console) | needs to touch the Docker socket — never public |
| **Uptime Kuma** | uptime checks + alerting on public surfaces | lightweight, notifications into our chat |
| **SonarQube** | code-quality scanning (smells, coverage, security hotspots) | self-hosted instance scans private repos pre-merge; findings feed the PR gate. Scope queries to a single project key — sandbox/WIP projects pollute unscoped dashboards |
| **SonarQube MCP** | the code-quality server, queryable by agents mid-review | agents triage findings without leaving the lane |
| **Firecrawl** | self-hosted web scraping/research stack (API + Playwright + Redis + RabbitMQ + Postgres + FoundationDB) | agents do market research constantly; local = no per-page fees, no data leaving |
| **Honcho** | pluggable agent-memory server (pgvector) | semantic memory beyond the flat file |
| **Watchtower** | auto-updates containers | small services track upstream patches without a human |
| **k3s + ARC** | tiny Kubernetes for GitHub Actions self-hosted runners | mobile CI at zero billed minutes (see [05](05-ci-cd.md)) |
| **GitHub Actions** | CI/CD for every repo | the control plane (see [04](04-governance.md), [05](05-ci-cd.md)) |
| **Firebase** | app production: Auth, Firestore, Functions, Hosting | separate staging/prod projects (see [15](15-app-release-pipeline.md)) |
| **faster-whisper + TTS** | local voice transcription + per-agent speech synthesis | privacy + zero per-minute fees (see [09](09-voice-pipeline.md)) |
| **GitHub MCP / Trello MCP / Flutter MCP** | structured tool access for agents (see [11](11-tool-integrations-mcp.md)) | agents act through typed APIs, not screen-scraping |

## Selection rules

1. **Self-host when the data is sensitive or the usage is heavy** (SonarQube on private code, Firecrawl on research volume). Host when it's commodity (GitHub Actions on public repos, Firebase for app infra).
2. **Bind admin surfaces to the tailnet, never the internet** — Portainer, SonarQube, Uptime Kuma, dashboards: all `100.x.y.z:port` only.
3. **Every tool must be reachable by an agent through a typed interface** (API, MCP, CLI) — if a human has to click it, it will silently rot.
4. **Watchtower only on stateless services.** Databases and anything with hand-migrated state update on human schedule, not automatically.
