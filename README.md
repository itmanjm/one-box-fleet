# Jozu Ops Playbook

An open playbook for running a **multi-agent AI operations stack** on a single machine — the real setup we run a software business on daily, decomposed into the discrete workflows and pipelines that make up the whole.

Every document in [`docs/`](docs/) is **self-contained**: adopt one pipeline without adopting the rest. The stack is deliberately boring — Ubuntu, Docker, GitHub, Tailscale, and an open agent harness ([Hermes Agent](https://hermes-agent.nousresearch.com)) — so everything here is reproducible on commodity hardware.

## The whole system in one picture

```mermaid
graph TB
    subgraph Human
        F[Founder / Operators]
    end
    subgraph Chat["Presentation (03)"]
        TG[Telegram bots]
        DC[Discord channels]
    end
    subgraph Box["One Ubuntu box"]
        subgraph Harness["Agent Harness (01)"]
            A1[Orchestrator agent]
            A2[Engineering agent]
            A3[Product agent]
            A4[Research agent]
        end
        MR[Model Routing + Fallback (02)]
        MEM[Memory + Skills (10)]
        VP[Voice Pipeline (09)]
        MON[Monitoring (07)]
        BAK[Backups (08)]
    end
    subgraph GitHub["GitHub = Control Plane (04, 05)"]
        GHG[Governance + Guardrails]
        CICD[CI/CD Pipeline]
    end
    subgraph Prod["Production"]
        FB[Firebase apps]
        MOB[Mobile apps]
    end

    F <--> TG & DC
    TG & DC --> A1
    A1 & A2 & A3 & A4 --> MR
    MR --> LLM[LLM Providers]
    A1 & A2 & A3 & A4 --> MEM
    F --> VP --> A1
    A2 --> CICD --> FB & MOB
    GHG --> CICD
    MON --> F
    BAK -.-> Box
```

## The pipelines

| # | Pipeline | What it covers | Adopt it if you want… |
|---|---|---|---|
| 01 | [Agent Harness & Profiles](docs/01-agent-harness.md) | One box, many specialist agents | multiple AI agents with separate contexts |
| 02 | [Model Routing & Fallback](docs/02-model-routing.md) | Primary/fallback chains, cost tiers | LLM resilience without overpaying |
| 03 | [Chat Presentation Layer](docs/03-chat-gateways.md) | Telegram/Discord as agent interfaces | agents you talk to from your phone |
| 04 | [GitHub-Native Governance](docs/04-governance.md) | Branch protection, AI review, decision gates | agents that can't do damage |
| 05 | [CI/CD Pipeline](docs/05-ci-cd.md) | Staging-first deploys, self-hosted runners | safe shipping with human gates |
| 06 | [The Work Lifecycle](docs/06-work-lifecycle.md) | Idea → vetting → build → verify → live | evidence-based "done" |
| 07 | [Monitoring & Observability](docs/07-monitoring.md) | Container dashboard, uptime, digests | knowing before users tell you |
| 08 | [Backups & Recovery](docs/08-backups.md) | Nightly backups, restore discipline | never losing the stack |
| 09 | [Voice Pipeline](docs/09-voice-pipeline.md) | Voice notes in, voice replies out | talking to your agents |
| 10 | [Memory & Skills](docs/10-memory-skills.md) | Persistent memory + procedure library | agents that learn |

## Principles everything here obeys

1. **Staging-first** — never experiment in production
2. **Evidence over claims** — nothing is "done" until verified live with real output
3. **Human gates on irreversible actions** — agents propose, humans dispose
4. **Private by default** — services bind to a private mesh (Tailscale), never the public internet
5. **Secrets never in git or chat** — `.env` files and token stores only

## Hardware this runs on

One Dell OptiPlex (or any 16GB+ Ubuntu box), a Raspberry Pi for ops/monitoring, and Tailscale on every device. No cloud VMs required for the ops layer itself.

## License

MIT — take it, fork it, run your business on it.
