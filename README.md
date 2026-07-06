# Apollo Operator

Open-source Claude Code skills for running B2B outbound on Apollo.io. Built by Creatop.

The Apollo MCP is powerful but has no opinion. Apollo Operator is the expert layer on top: it knows which tool to call, in what order, and against what standard, so you are not starting from scratch.

## What's inside

`operator-context` loads first (the Apollo tool map, the rules, the routing). Under it, skills organized by stage:

- **ICP and targeting:** `apollo-icp-builder`
- **List building:** `apollo-list-builder`, `list-quality-scorecard`
- **Copy and sequences:** `apollo-sequence-builder`, `sequence-reviewer`
- **Deliverability:** `apollo-deliverability`
- **Iterate:** `positive-reply-scoring`, `experiment-design`, `weekly-rhythm`
- **Signals:** `apollo-signals`

## Requirements

- Claude Code
- The Apollo MCP connected
- An email verifier (e.g. Debounce) for the verification step

## Use

Open this folder in Claude Code. The skills live in `.claude/skills/` and load automatically. Describe what you want ("help me define my ICP and build a list on Apollo") and `operator-context` routes you to the right skill.

## Philosophy

Read `.claude/skills/operator-context/references/outbound-principles.md`. Short version: outbound is a system game, relevance beats volume, deliverability is sacred, humans approve the copy, and these skills surface risks but never block. You decide.

Built by Creatop. https://creatop.net
