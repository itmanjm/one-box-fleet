# 02 — Model Routing & Fallback

> Every agent has a primary model and a fallback chain — so a provider outage or a hit rate limit degrades instead of dying, and cheap work doesn't burn premium tokens.

## The chain

Each profile's `config.yaml` defines a primary plus ordered fallbacks:

```yaml
provider: zai
model: glm-5.3-flash          # fast/cheap primary
fallback_providers:
  - provider: openai-codex    # ChatGPT OAuth entitlement (no extra API key)
    model: gpt-6-sol
  - provider: copilot          # permitted on select profiles only
    model: gpt-4.1
  - provider: zai              # same-provider cheaper tier as last resort
    model: glm-4.7
```

## Rules that took scars to learn

1. **Use entitlements before buying keys.** A ChatGPT/Codex/Copilot subscription often covers API-style access via OAuth. Exhaust those before purchasing a separate provider key.
2. **Set `context: inherit` on every tier** — otherwise fallback models answer without your system context and produce confidently wrong work.
3. **Retries: 3, fleet-wide.** Enough to ride a transient 429, few enough to fail fast into the next tier.
4. **Know your rate-limit windows.** Providers enforce per-minute bursts AND rolling multi-hour windows. A `429` with "resets at 10:06" is the window type — the fallback chain should catch it, and the *routing* should prevent it (see below).
5. **Route by task, not by hope.** Heavy fan-out work (parallel subagents) causes burst 429s. Give high-fanout lanes their own provider or throttle the fan-out.
6. **A silent fallback is a bug surface.** If a degraded model quietly takes over, output quality drifts. Log which tier served each request; alert when you're living on fallbacks.
7. **No blanket model switches.** Change one lane at a time, verify with a real task, then move the next lane. (Ripple outages from fleet-wide swaps are self-inflicted.)

## Verifying a model actually works

Before trusting any provider entry, send one real completion:

```bash
curl -s https://api.example-provider.com/v1/chat/completions \
  -H "Authorization: Bearer $KEY" \
  -d '{"model":"glm-5.3-flash","messages":[{"role":"user","content":"reply with OK"}]}'
```

Check the `request_id` in the response — it's your evidence when disputing billing or rate-limit behavior with the provider.

## Cost discipline

- **Free tiers for low-stakes work** (scheduled scans, digests) — but read the per-minute limits first; a tool-heavy cron job will blow a 15 req/min free tier instantly. Repin heavy crons to a paid lane.
- **Premium models for judgment** (code review, architecture), cheap models for mechanics (formatting, search summarization).
