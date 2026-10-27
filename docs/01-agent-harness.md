# 01 — Agent Harness & Profiles

> One machine hosts many AI agents, each with its own identity, memory, tools, and chat channels — running concurrently as isolated OS processes.

## Concept

Instead of one do-everything assistant, run **specialist profiles**: an orchestrator, an engineer, a product strategist, a researcher, a content agent. Each profile is a directory containing its own config, memory, skills, cron jobs, and credentials. The harness ([Hermes Agent](https://hermes-agent.nousresearch.com), install via git) runs one gateway process per profile.

```
~/.hermes/
├── hermes-agent/          # the runtime (git checkout, `hermes update` to upgrade)
├── profiles/
│   ├── atlas/             # orchestrator: priorities, escalation, delegation
│   ├── forge/             # engineering: code, CI, deploys
│   ├── geo/               # product: specs, PRDs, roadmaps
│   ├── scout/             # research: competitors, markets
│   ├── spectre/           # content: video, transcripts
│   └── luna/              # personal assistant (Docker-containerized)
└── config.yaml            # shared defaults (model, MCP servers)
```

## Why roles instead of one agent

- **Context hygiene:** the engineer's context isn't polluted by marketing research
- **Blast radius:** a bad model response in one lane can't mutate another lane's state
- **Model routing per role:** cheap models for research sweeps, premium for code — per profile
- **Auditable ownership:** every artifact traces to a named lane

## Setup steps

1. Install the harness: `git clone` the runtime, run its installer (Python 3.12+).
2. `hermes profile create <name>` (or just create the directory) per role.
3. In each profile's `config.yaml`: set `provider`/`model` and the fallback chain (see [02](02-model-routing.md)).
4. Put API keys in the profile's `.env` — never in config files that get synced.
5. Start gateways: `hermes gateway run --profile <name>` (run under systemd for persistence).

## Keep-alive pattern

Run each gateway as a systemd user service (`Restart=on-failure`) so a crash or OOM kill self-heals:

```ini
# ~/.config/systemd/user/hermes-atlas.service
[Unit]
Description=Hermes gateway (atlas)
[Service]
ExecStart=%h/.hermes/tools/python/bin/hermes gateway run --profile atlas
Restart=on-failure
[Install]
WantedBy=default.target
```

## Delegating between lanes

Work moves between agents through the board, not through direct conversation — the card is the contract:

- **One card, one owner, one claim.** Before an agent works a card it claims it (visible state change); a second agent seeing a claimed card moves on, not in parallel. This is what prevents the classic fleet failure: two agents independently solving the same task.
- **Claims are durable.** A claim without a committed artifact (branch, draft, note) within a grace window auto-releases the card back to the queue — dead claims can't park work forever.
- **Hand-offs happen at gate boundaries** ([06](06-work-lifecycle.md)): engineering hands to the requester at the staging gate, product hands to engineering at the spec gate. A hand-off is the card moving status + a comment with the evidence, never a private message.
- **The orchestrator routes, never absorbs.** Its job is prioritization and arbitration; domain work goes to the domain lane. When everything routes through one agent, that agent's context becomes the bottleneck and the single point of failure.

## Pitfalls

- **Don't share one profile across roles** — memory contamination is real and silent.
- **Restart gateways after credential rotation** — `.env` changes aren't hot-reloaded.
- **Version-pin the runtime** in production lanes; update on your schedule, not the maintainer's.
