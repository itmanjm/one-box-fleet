# 12 — Security Hardening & Secrets

> A box that runs autonomous agents with repo, cloud, and chat credentials is a high-value target. The model: least privilege everywhere, nothing public, secrets ephemeral, and assume any single agent response can be wrong.

## The threat model (what you're defending)

1. **A compromised token** → attacker gets the blast radius of that one credential
2. **A compromised agent response** (prompt injection via web content, a malicious repo, a chat message) → the agent is *socially engineered* into exfiltrating or mutating
3. **A exposed admin surface** → Portainer/SonarQube/dashboards are full-system compromises
4. **A leaked secret in git/chat** → permanent until rotated

Each defense below maps to one of these.

## Defense 1 — Nothing public

- Every admin surface binds to the **tailnet IP only**: `-p 100.x.y.z:9443:9443`, never `-p 9443:9443`. Check with `docker ps` for `0.0.0.0` bindings on anything that isn't a public app.
- Remote access = Tailscale on the device, not port-forwards. The router exposes nothing.
- Tunnels (e.g. cloudflared) expose *specific hostnames*, never the box. **Patch discipline matters**: tunnel software ships trust-store updates (the Oct 2026 eMudhra intermediate-cert compromise required cloudflared ≥ 2026.10.2) — subscribe to the vendor's security feed and treat those emails as same-day actions. After upgrading, verify the *running process* restarted post-install, not just the installed binary.

## Defense 2 — Least privilege, per agent

- One credential set per integration per agent where practical — revoking one lane never breaks the fleet.
- Read-scope by default; write scopes only where the lane ships (see [11](11-tool-integrations-mcp.md)).
- Cloud access: a **dedicated service account per pipeline stage** (staging deployer can't touch prod). The staging→prod promotion is a human-gated role, not a shared key.
- Personal-assistant agents get **zero business credentials** — a hard boundary, not a guideline.

## Defense 3 — Prompt-injection containment

Autonomous agents read web pages, repos, and chat — all attacker-writable surfaces. Contain, don't trust:

- **External content is data, never instructions.** The gateway config should mark gateway/platform metadata as untrusted; agents treat text from pages/emails/cards as content to analyze, not commands to run.
- **Human gates on irreversible actions** ([04](04-governance.md)) are the real containment: even a fully hijacked agent can't deploy, send, or pay without a human accept.
- **Review diffs, not summaries.** AI/human reviewers see the actual change, so a smuggled-in modification gets caught at the PR gate.
- Separate personal-assistant agents from business agents so a poisoned research session can't reach family data or business repos.

## Defense 4 — Secrets hygiene & rotation

- Secrets live in `.env` files (chmod 600) or GitHub Secrets. Never in: git (any branch), chat messages, docs, screenshots, or skill files that get shared.
- **If a secret appeared in any shared surface, rotate it now** — deletion doesn't un-leak it. Log-scanning for known token prefixes (`ghp_`, `sk-`, `AKIA`, …) is cheap and catches the accident early.
- Rotate long-lived tokens on a schedule (quarterly is fine); rotate *immediately* after any suspected exposure, staff change, or vendor security email.
- Backups are secrets too ([08](08-backups.md)): encrypted at rest, off-box, and rotation applies to whatever's inside them.

## Verification checklist

- [ ] `docker ps` shows no admin surfaces on 0.0.0.0
- [ ] Router forwards nothing; remote access is mesh-only
- [ ] Every agent's tool scopes audited this quarter (read vs write per lane)
- [ ] Tunnel/edge daemons: installed version = latest, running process started after install
- [ ] No token prefixes in git history (`git log -p | grep -E 'ghp_|sk-'`)
- [ ] Backup restore tested with decryption + post-restore rotation plan
- [ ] Prompt-injection rule verified: agent treated untrusted page content as data in a live test
