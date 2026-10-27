# 11 — Tool Integrations: MCP & Connectors

> Agents act through **typed tool interfaces** — Model Context Protocol (MCP) servers, CLIs, and APIs — never through screen-scraping. This doc is how tools get assigned, trusted, and debugged.

## What MCP gives you

An MCP server is a small service that exposes a toolset (search GitHub, move a Trello card, run a Flutter analyze) over a standard protocol. The agent discovers tools at connect time and calls them with typed arguments — structured, auditable, permission-scoped.

Two shapes:

- **stdio (local):** the agent spawns the server as a subprocess (`npx @modelcontextprotocol/server-github`). Secrets pass via env vars. Good for tools bound to this machine.
- **remote (URL):** an HTTP/SSE endpoint — a hosted connector or your own service on the tailnet.

## Our per-agent tool map (the principle: least tooling per lane)

| Agent | Tools | Why |
|---|---|---|
| Orchestrator | browser, search/read MCPs, memory MCP, design/BI connectors, chat platforms | breadth; no repo write access |
| Engineering | GitHub MCP, code-quality MCP, Flutter/Dart MCP, CI tooling | everything needed to ship; nothing marketing-ish |
| Product | GitHub MCP (read/issues), search MCPs, Trello | specs and boards, not deploys |
| Research | search/reader MCPs, scraping stack | read the world, touch nothing |

Rules: an agent gets **the tools its lane needs and no more**; write-scoped tools go only to lanes that ship; personal-assistant agents get zero business tools (hard boundary, separately enforced).

## Connecting a new tool — the checklist

1. **Scope first:** what's the minimal permission set? (Read before Edit; one repo before all repos.)
2. ** secrets in env, never in config files that sync or in chat.** (`TOKEN=[REDACTED]` in any shared artifact — always.)
3. **Prove the scope bites:** an "active" credential isn't enough — run one real read *and* one real write probe per granted scope. Scope gaps surface as generic auth errors mid-task and burn an hour every time.
4. **Assign to the narrowest lane.** Document the assignment in the agent's config, in a comment, not in tribal memory.

## Debugging a tool that "suddenly fails"

The failure is almost never the tool — it's one of these, in order of likelihood:

1. **Credential expired/rotated** (OAuth refresh died, token revoked)
2. **Scope gap** — token valid, but missing the permission for *this* operation (the error says "auth failed"; it means "scope missing")
3. **Rate limit / quota** — provider-side burst or window caps; retry policy should catch it
4. **Protocol mismatch** — server updated, schema changed; pin versions for stdio servers

Diagnostic discipline: run the failing call *by hand* with the same credential before blaming the agent or the server.

## A scar worth sharing: audit metadata must reference real state

One of our coordination servers wrote an **idempotency/audit header verbatim into an activity log with a foreign-key to real run records**. A caller inventing a plausible-looking ID got a silent 500 and a fully rolled-back write — the create *succeeded* in the logs and *didn't happen* in the database.

The general rule: **any audit/provenance metadata you attach to a mutation must reference server-verified state — or be omittable.** Never invent plausible IDs to satisfy a "required" header. And when a write reports success-but-didn't-persist, check FK constraints on audit tables before anything else.
