# Apollo Operator

**An opinionated go-to-market operator for Apollo.io, built as open-source Claude Code skills.**

The Apollo MCP can run your entire outbound motion. It just has no idea how. Point it at your account raw and it will run any search, build any sequence, and send any email you ask for, with no reason to tell you the targeting is off, the copy reads like a robot, or the send will burn your domain. It does what you say, not what you meant.

Apollo Operator is the knowledge layer. It is a GTM system of 14 skills that turn the raw MCP into a guided outbound motion: understanding the business, who to target, how to build and grade a list, how to write sequences that get replies, how to take them live without torching your domain, and how to tell whether any of it is actually working.

## What it runs

One router skill (`operator-context`) loads first and hands off to specialists by stage:

- **Business brief** (`business-brief`): one readable `brief.md` any agent can read once and understand the business: identity, offer, buyers, objections, proof, voice.
- **Targeting** (`apollo-icp-builder`): turn an ICP into a real Apollo search and a scored profile.
- **List building** (`apollo-list-builder`, `list-quality-scorecard`): search, enrich, verify, and grade before anything sends.
- **Copy and sequences** (`apollo-sequence-builder`, `sequence-reviewer`): short, human sequences matched to the persona, reviewed for spam risk.
- **Multichannel, optional** (`apollo-multichannel`): add LinkedIn and call steps, then work the task queue.
- **Go-live** (`apollo-go-live`): enroll the list, a human approves, and contacts pull on reply.
- **Deliverability** (`apollo-deliverability`): warmup, verification, and the bounce rules that keep you inboxing.
- **Iterate** (`positive-reply-scoring`, `experiment-design`, `weekly-rhythm`): measure intent, run clean experiments, keep a cadence.
- **Signals** (`apollo-signals`): find the reason to reach out now.

## Built by operators, tested live

Every skill holds your campaigns to the same standard we hold ours, written down and made runnable. We built this on our own Apollo account, and the testing paid for itself: a title filter quietly inflated a search from 679 real matches to over 91,000, and a bulk create call returned a success message while silently making contacts you could never use. We caught both, and the fixes are encoded in the library.

## Requirements

- Claude Code
- The Apollo MCP connected
- An email verifier (for example Debounce) for the verification step

## Use

Open this folder in Claude Code. The skills live in `.claude/skills/` and load automatically. Describe what you want ("help me define my ICP and build a list on Apollo") and `operator-context` routes you to the right skill.

## Philosophy

Read `.claude/skills/operator-context/references/outbound-principles.md`. The short version: outbound is a system, relevance beats volume, deliverability is sacred, humans approve the copy, and these skills surface risks but never block. You decide.

## Who it is for

Founders and go-to-market teams running outbound on Apollo who want a system, not a black box. Use it free to run your own motion. If you would rather have it built and run for you, that is what we do at Creatop.

https://creatop.net
