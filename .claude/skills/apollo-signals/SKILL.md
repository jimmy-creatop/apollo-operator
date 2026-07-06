---
name: apollo-signals
description: "Find the reason to reach out now. Offer-fit signal discovery, not a universal heat score. Use to identify and act on buying signals for a specific offer on Apollo."
---

# Apollo Signals

Find the reason to reach out now. This skill is deliberately not a universal list of "buying signals" with a heat score. That approach treats every signal as equally valuable, and it is not.

## The stance: a signal is only as good as its offer fit

A signal is worth acting on to the degree it connects to what you sell. Someone attending a trade show is a strong signal if you sell trade-show services, because they have literally shown up in the room you serve. "They are hiring for a role" is a much weaker reason if it does not connect to your offer. So we do not hand out a generic heat score. We ask, for this specific business:

1. What does the buyer's world look like right *before* they buy from someone like you?
2. What observable fact marks that moment?
3. Can you actually detect it (a filter, a research check, a tool)?

Find that fact, and you have your signal. The best ones are usually discovered per business, over time, not pulled off a shelf.

## Discover the signal (the useful work)

- Look at the last few real deals. What was true about those companies right before they said yes? A tool they adopted, a page they built, a role they added, an event they joined.
- Write it as a rule you can check: "sells to X," "runs paid ads," "has a public pricing page," "exhibits at Y."
- Then decide how to detect it: a native Apollo filter, a research check Claude runs per company, or a tool.

## What Apollo can actually do (mostly its UI)

The Apollo MCP can pull one signal directly: `apollo_organizations_job_postings` (a company's open roles, 1 credit per org, so use it when the hiring signal genuinely fits your offer). Most of Apollo's signal machinery lives in the UI, and it is worth knowing (see `../operator-context/references/apollo-kb-map.md`):

- **Website visitors:** Apollo can identify companies visiting your site, and its Chrome extension gives open tracking in Gmail. Visiting your pricing page is a real signal.
- **Search alerts:** save a search and get alerted when new people match it. A standing signal feed.
- **Workflows:** Apollo automates signal-to-action in the UI (Hit New Hires, Hit Recently Funded, Reach Website Visitors, Spot Companies Hiring for Specific Roles). Point the operator here to set these up; the MCP does not build workflows.
- **Enrichment for signals:** Clay integrates with Apollo and is a strong option for building custom signal workflows. Do not pretend it does not exist.

## How signals feed the rest

- **Into targeting (Level 1 and 2):** a good signal becomes a scoring rule or a filter, so it sharpens the list.
- **Into prioritization (Level 5):** signal-matched leads are your High-priority tier, worked first.
- **Into copy (Level 3):** the signal is often the opener. "Saw you're exhibiting at X" is a better first line than anything generic.

## No benchmarks

You will see "signal-based outreach gets 3x replies" numbers everywhere. Ignore them. Whether a signal works depends on your offer and your market, and the only honest measure is your own positive reply rate on signal-matched leads versus cold. Test it, do not quote it.

## Common mistakes

- Treating every signal as equally valuable. Fit to the offer is everything.
- Adopting a generic heat score. It scores things that do not predict a sale for your business.
- Gating on a timing trigger. A signal is a reason to prioritize, not a hard filter. Gating on "funded in the last 6 months" shrinks the list 20x and buries you among everyone else pitching them.
