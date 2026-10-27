# 16 — Full Inventory: MCPs, CLIs, Applications & Skills

> The complete, named inventory behind the playbook. Everything here is live-verified on our box. Names only — no credentials, ever.

## A. MCP servers (14 distinct across the fleet)

MCP (Model Context Protocol) servers are how agents get **typed tool access** — see [11](11-tool-integrations-mcp.md) for the assignment philosophy.

| MCP server | Transport | What it gives agents | Which agents |
|---|---|---|---|
| `github` | stdio (npx `@modelcontextprotocol/server-github`) | issues, PRs, repos, reviews | all business agents |
| `trello` | stdio (`@delorenj/mcp-server-trello`) | boards, cards, checklists | business agents |
| `dart-flutter` | stdio (`dart mcp-server`) | Flutter/Dart analysis, hot reload, pub, widget inspector | engineering lane |
| `zai-search` | remote (SSE) | web search | all agents |
| `zai-reader` | remote (SSE) | URL → clean content extraction | all agents |
| `zai-zread` | remote (SSE) | GitHub repo structure/reading | all agents |
| `watermelon` | remote (HTTP) | UI component catalog + compatibility search | orchestrator |
| `stitch` | remote (HTTP) | AI UI screen generation + design systems | orchestrator |
| `honcho` | local (HTTP → Docker) | pluggable semantic memory (pgvector) | orchestrator |
| `playwright` | stdio | browser automation | shared/default profile |
| `gcp` | stdio | Google Cloud operations | shared/default profile |
| `zai-vision` | remote (SSE) | image understanding | shared/default profile |
| `sonarqube` | local | code-quality findings queryable mid-review | engineering lane |
| `markitdown` | stdio | document → markdown conversion | personal assistant |

Per-profile spread (agents share the core five: github, trello, zai-search/reader/zread): orchestrator 9 · engineering 6 · product/marketing/governance/research 5 each · personal assistant 4 · content 2.

## B. CLI tooling (~54 binaries of substance)

**Agent & orchestration** (`~/.local/bin`):
`hermes` (harness) · `hermes-acp`, `hermes-agent` (runtime variants) · per-agent launchers · `mcporter` (MCP bridge/CLI) · `mcp` · `tiny-agents` · `agent-reach` (multi-backend research router) · `opencode` (coding agent CLI)

**Docs & data:**
`gws` (Google Workspace: Drive/Docs/Gmail/Calendar) · `markitdown` · `jsonschema` · `markdown-it` · `yt-dlp` (feeds the voice pipeline)

**Python/packaging:**
`uv`, `uvx`, `pip3`, `dotenv`, secondary interpreter

**AI/ML:**
`huggingface-cli`, `hf` · CTranslate2 converter family (`ct2-*` ×7: fairseq, marian, openai-gpt2, opennmt-py/tf, opus-mt, transformers) · `onnxruntime_test`

**System (on PATH):**
`gh` (GitHub) · `gcloud` · `firebase` · `flutter` · `dart` · `docker` · `kubectl` · `helm` · `tailscale` · `ffmpeg` (voice pipeline prep) · `node`/`npm` (npx-hosted MCP servers) · `codex` (OAuth-backed reviewer bridge)

## C. Applications & services (Docker + system)

| Application | Role | Access |
|---|---|---|
| **SonarQube** (+ Postgres) | code-quality gate: smells, coverage, security hotspots on private repos; findings feed the PR review lane. **Scope queries to one project key** — sandbox/WIP projects pollute unscoped views | tailnet-only + Access-wrapped public hostname |
| **SonarQube MCP** | the same findings, queryable by the engineering agent mid-PR | agent lane |
| **Firecrawl stack** (api, playwright, redis, rabbitmq, postgres, foundationdb) | self-hosted web research/scraping — zero per-page fees, data stays local | localhost only |
| **Honcho** (api, deriver, pgvector DB) | semantic agent-memory server | localhost only |
| **Portainer** | container dashboard: health, stats, logs, console | tailnet-only bind, never public |
| **Uptime Kuma** (on the ops Pi) | uptime checks + alerts on public surfaces | Access-wrapped hostname |
| **Watchtower** | auto-updates stateless containers only | internal |
| **Luna / Merix** | personal-assistant agent containers | chat-only |
| **k3s + Actions Runner Controller** | self-hosted GitHub runners for mobile CI; **Headlamp** (view-only role) as the cluster UI | tailnet + Access |
| **faster-whisper + TTS** | local voice transcription + per-agent speech | on-box |

**Selection rule recap:** self-host what's sensitive or high-volume (SonarQube on private code, Firecrawl on research); host what's commodity (GitHub Actions, Firebase app infra). Admin surfaces: tailnet or Access-wrapped, never raw-public.

## D. Customized skills (~133 curated across 17 categories)

Skills are the procedure library — each a `SKILL.md` playbook with triggers, exact commands, pitfalls, and verification checklists (format: [10](10-memory-skills.md); public example: [frontend-mobile-ui-design](https://github.com/itmanjm/frontend-mobile-ui-design)).

| Category | ~Count | Highlights |
|---|---|---|
| devops | 40 | firebase ops, deploy verification, release gates, k8s runners, SonarQube lifecycle, backup/restore, monitoring |
| software-development | 25 | Flutter testing/networking/auth patterns, static analysis, PRD writing, systematic debugging, plan authoring |
| design | 14 | the front-end/mobile UI lane (see our public skill), Figma bridge, animation review, design systems |
| governance | 8 | decision gates, model-cost routing, production readiness, product intake/vetting |
| communication | 9 | document formatting, inbox triage, voice delivery, token-lean updates |
| productivity | 8 | Google Workspace integration, meeting action items, weekly review, price monitors |
| research | 7 | deep research ops, competitor monitoring, citation grounding, social-content extraction |
| autonomous-ai-agents | 4 | multi-agent orchestration, merge-conflict resolution, external skill absorption |
| github | 3 | issue-to-PR flow, PR triage, git control plane |
| personal-assistant | 3 | containerized PA operations + Hermes updates |
| email, mcp, memory, misc | 7 | AgentMail ops, MCP client/bridge, memory wiki, store transfer, fallback diagnostics |
| creative | 2 | visual design tools, article illustration |

**The library compounds by rule:** after any difficult task, the lessons land back in the skill that carried it — so the next run starts where this one ended.

## E. Numbers at a glance

- **14** distinct MCP servers · **~54** CLI binaries · **16** Docker applications/services · **~133** curated skills in 17 categories · **9** agent profiles · **3** machines (ops box + 2 Pis)
