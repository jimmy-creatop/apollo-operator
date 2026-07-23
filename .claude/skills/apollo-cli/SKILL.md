---
name: apollo-cli
description: "The three ways to run Apollo headless: MCP (typed tools in context, the default), the CLI (composable, cheap on context, OAuth via apollo auth login), and the raw REST API. Explains when Operator should reach for each, plus CLI setup and a zero-credit auth-health preflight. Use to decide MCP vs CLI vs API for a task."
---

# Apollo Access: MCP, CLI, or API (Level 0 · Access)

Apollo can be driven three ways, and picking the right one per task saves context and credits. This library is MCP-native by default: every other skill assumes Apollo shows up as typed tools inside the model. But Apollo also ships a CLI and a raw REST API, and for some jobs those are the better substrate. This skill is the map of the three lanes and when to reach for each. It does not change how any other skill works today, it tells you what your options are.

## The three lanes (same API underneath)

All three hit the same Apollo API (the same data, enrichment, sequences, and CRM). What differs is *how the work is invoked*.

| Lane | How work is invoked | Auth | Best for |
|---|---|---|---|
| **MCP** (default here) | Apollo appears as typed tools inside the model's context; the model picks a tool and fills a JSON schema | OAuth 2.0 (`https://mcp.apollo.io/mcp`) | Guardrailed, in-conversation steps where structured, typed calls matter |
| **CLI** (new) | A normal terminal command an agent shells out to, or a human/script/cron runs | OAuth 2.0 (`apollo auth login`, stored and auto-refreshed) | High-volume search and enrich, composable pipelines, reproducible scripted jobs |
| **REST API** | You build your own client against the HTTP endpoints | API key (`x-api-key`, `APOLLO_API_KEY`) | Custom integrations and back-end services, most control, most work |

Apollo explicitly warns not to confuse API access with MCP. They are different lanes to the same place.

## MCP vs CLI (the choice that actually comes up)

For an agent running Operator, the real decision is MCP or CLI, because both can be driven from inside a Claude Code session.

- **MCP** loads roughly 40 tool schemas into the model's context whether you use them or not. That is real token overhead, and it crowds the window, but in exchange you get typed, structured, in-conversation calls the model cannot fat-finger.
- **CLI** loads nothing upfront. The agent runs `apollo --help` on demand, then composes commands. It is deterministic (same command, same result), scriptable, version-controllable, and it runs with or without an AI in the loop.

The one-liner: **MCP puts Apollo *inside* the model as tools. The CLI puts Apollo *under* the agent as a composable command it can pipe, script, and re-run.**

The CLI's composability is the standout. A real pipeline like `apollo people search ... | jq ... > leads.csv` chains search, filter, and save in one line, which is exactly the shape of Level 2 list work. MCP tool calls do not compose like that.

## When Operator uses which

**Default to MCP.** Every skill in this library is written against the MCP surface today, and that is the right substrate for anything guardrailed or human-gated.

Reach for the **CLI** when:
- You are doing high-volume search or enrichment and the MCP tool schemas are eating context you would rather spend on the work.
- The step is a pipeline: search, then filter, dedupe, save, loop. The CLI plus `jq` does this natively (see Level 2, `apollo-list-builder`).
- You want the job reproducible: a script you can re-run, commit, or schedule.

Stay on **MCP** when:
- The step is human-gated and reputation-sensitive: enrollment and activation (`apollo-go-live`), where you want typed calls and no shell improvisation.
- You want the model held to a structured schema rather than composing raw commands.

**Status, be honest about it:** as of v1.1 the skills are MCP-native. CLI-backed variants of the search and enrich steps are a planned upgrade (v1.2), to be prototyped and measured (context cost and speed) before they ship. This skill is the map; the rewrite is future work, not a promise that any skill shells out today.

## Setup (CLI)

Per Apollo's docs (https://docs.apollo.io/docs/apollo-cli-overview):

- **Install:** Homebrew (`brew install apolloio/apollo-io-cli/apollo-io-cli`), a prebuilt binary for macOS, Linux, or Windows (no Node needed), or from source (Node 18+).
- **Auth:** `apollo auth login` once. It uses OAuth 2.0, not an API key, and stores and auto-refreshes credentials locally.
- **Output:** JSON, JSONL, CSV, YAML, or table, and it is `jq`-compatible for piping and scripting.
- **Covers:** people search and enrich, company operations, contact, account, and deal management, sequence enrollment, call logging, task creation, and credit usage.

## Preflight (zero credit)

Before real work on any lane, confirm access. The CLI and API expose `GET /v1/auth/health`, which returns `{ "healthy": true, "is_logged_in": true }` and spends **no credits**. It is the cleanest way to verify a session or key works before doing anything that costs money, the same role `apollo_users_api_profile` plays on the MCP preflight in `operator-context`.

## Common mistakes

- **Confusing API access with MCP.** Different lanes, different auth. Apollo warns about this directly.
- **Loading the full MCP tool surface for a bulk search** a single CLI command plus `jq` would do more cheaply. Match the lane to the job.
- **Using the CLI for human-gated activation.** Enrollment and go-live want the MCP's typed, guardrailed calls, not a shell command the model composed.
- **Assuming the CLI needs an API key.** It is OAuth (`apollo auth login`). The API key is the third lane, for building your own client.
