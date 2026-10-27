# 03 — Chat Presentation Layer

> Agents you talk to from your phone: Telegram DMs and Discord channels become the interface to every agent in the fleet.

## The shape

- **Telegram** — one bot per agent; DMs are private working sessions. This is where the founder lives.
- **Discord** — team-visible surface: a home channel per agent for status, plus threads per project so work history is browsable.
- **Desktop/CLI** — local TUI for deep sessions when chat is the wrong tool.

## Setup

1. **Telegram:** create a bot with @BotFather → token into the profile's `.env` → the gateway wires the rest. Each profile gets its own bot, so each conversation is unambiguous.
2. **Discord:** one application in the Developer Portal, bot token in `.env`, invite to your server, map channels to profiles in the gateway config.
3. Route notifications deliberately: cron outputs and job completions go to the home channel, not the void.

## Rules that matter

- **Threads are the unit of work history.** For every project, open a thread with a kickoff post (what, where the artifacts live, links). Long-lived context stays findable.
- **Chat is the approval surface.** Decision gates (see [04](04-governance.md)) deliver their accept/override prompts into chat — human sign-off happens where the human already is.
- **Bot-gated links are human-only.** Confirmation links with CAPTCHAs can't be opened by the agent — the agent fetches and summarizes, the human clicks.

## The IPv6 gotcha (mobile ISPs)

Some ISPs intermittently break Telegram bot connectivity over IPv4 (long-polling just goes silent). Fix on a Docker host:

1. Enable IPv6 on the host interface (`sysctl net.ipv6.conf.<iface>.accept_ra=2`)
2. Docker daemon: `"ip6tables": true` in `/etc/docker/daemon.json`
3. Compose networks: `enable_ipv6: true` with a ULA subnet per stack
4. Verify: `curl -6 https://api.telegram.org` answers, `getUpdates` returns 409 Conflict (means another poller — the gateway — holds the connection, which is *healthy*)

## Voice in, voice out

See [09 — Voice Pipeline](09-voice-pipeline.md) — voice notes arrive as audio attachments, get transcribed locally, and agents reply with synthesized speech in a per-agent voice.
