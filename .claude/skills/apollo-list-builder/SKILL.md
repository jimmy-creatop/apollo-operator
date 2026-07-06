---
name: apollo-list-builder
description: "Build a clean, enriched, graded lead list on Apollo: paginated search, dedupe, credit-aware enrichment, scoring signals, email verification, then grade. Use to build the full prospect list after the ICP is set."
---

# Apollo List Builder (Level 2)

Take the validated ICP from `profile.yaml` and turn it into a clean, enriched, graded lead list that is ready for a sequence. Search, enrich, apply the scoring signals, dedupe, then grade. Nothing goes to a sequence until it passes the scorecard.

## When to use

- Level 1 is done: `profile.yaml` has a validated ICP, the Apollo filter set, and 1 to 3 scoring signals.
- You need the full list, not the 50-to-100 sample you already validated.

If `profile.yaml` has no validated ICP yet, stop and run **Level 1 (`apollo-icp-builder`)** first. Building a full list on an unvalidated ICP is the most expensive mistake in outbound.

## The order matters: filter and dedupe before you enrich

Enrichment costs credits (1 per matched person). Search is cheap. So the pipeline spends as little as possible: narrow hard in search, dedupe, then enrich only the people you actually want. Never enrich first and filter later.

## Where the list lives (files on your drive)

Important: the list is not "in the model." Apollo's search returns records into the conversation, but you persist them to real files on the local drive, one per stage. This is so the list survives (you cannot hold thousands of rows in context), and so you can open any stage in a spreadsheet and see exactly what is happening. Convention:

```
lists/<campaign>/
  raw.csv        # step 1: every person from search (id, name, title, company, domain)
  deduped.csv    # step 2: after dedupe + per-domain cap
  enriched.csv   # step 3: + verified email and phone
  scored.csv     # step 4: + scoring-signal tag and priority
  final.csv      # step 6: passed the scorecard, ready for a sequence
```

Each step reads the previous file and writes the next. Nothing lives only in the model, and you can inspect the work at any point.

**MCP results overflow context, so route them to disk, never into the model.** A 100-record search, and even a 10-record enrichment, each return a payload too big for the context window; the harness parks it in a file. Process that file into your list, do not try to read it into the conversation. And when you enrich, **preserve the full payload.** Apollo returns roughly 33 fields per person (emails, phones, LinkedIn, employment history, seniority, intent). Save the complete JSON (`enriched.json`), then generate whatever columns you actually want as a CSV view from it (`enriched.csv`), and let the user pick which columns to keep or drop. Never reduce to a handful of columns and throw the rest away: you paid credits for that data, and it does not live anywhere else unless you save it.

## Pipeline

### 1. Build the raw list (search)
Run `apollo_mixed_people_api_search` with the exact `people_search_filters` from `profile.yaml`. Paginate: 100 per page, up to Apollo's 50,000-record ceiling (500 pages). Collect each person's `id`. Search returns no emails and may mask last names, that is expected and fine, enrichment fixes it.

If the count is far above what you need, tighten filters rather than enriching a bloated list.

### 2. Dedupe and cap concentration (before enriching)
- Drop duplicate people.
- Cap per-domain concentration (a rule of thumb: no more than 2 to 3 people per company unless it is deliberate ABM). One company flooding your list skews everything and looks like spam.

### 3. Enrich in batches (the credit step)
Enrich with `apollo_people_bulk_match`, passing the `id` from search (never the names). Hard limits: **max 10 people per call, 1 credit per match**. Before you start, confirm the total scope out loud with the exact wording the tool requires: "This will enrich [N] people and consume up to [N] credits (1 credit per match, no charge for unmatched). Do you want to proceed?" Confirm the whole scope up front, do not drip-confirm batch by batch.

Credit reality, learned the hard way: Apollo may bill **every requested record, not just the matches** (a 10-person batch with 7 matches charged 10 credits, not 7). Do not promise "unmatched are free." Read the actual `credits_consumed` field in the response and report the true number. Enriching costs credits; creating contacts or pushing them into Apollo afterward does not, once a lead is enriched, moving that data around is free.

### 4. Apply the scoring signals
From `profile.yaml`:
- **Filter signals** were already applied in search (step 1), so they are done.
- **Research signals** get applied now, per company. Claude reads the company site and decides (e.g. "has a public pricing page"), or you pull a firmographic signal with `apollo_organizations_enrich`, or a hiring signal with `apollo_organizations_job_postings` (1 credit per org, so use it only when the signal is worth it). Tag each lead by priority: **High** (hits the signal), **Medium** (fits the ICP but not the signal), **Low** (edge of the ICP). Write the tier to `scored.csv`. Work the High-priority leads first.

### 5. Verify emails
Every email gets verified before it can be sent to. Use the email status Apollo returns, and run anything uncertain through Debounce (or an equivalent verifier). Unverified equals bounce risk, and bounces kill domains.

### 6. Grade before you ship
Run **`list-quality-scorecard`** on the finished list. If it grades below B, fix the top issues and re-grade. Do not hand a C-grade list to a sequence.

### 7. Hand off
A clean, enriched, verified, graded list, with scoring tags, ready for **Level 3 (copy and sequences)**.

### Optional: also put the list into Apollo (not just the CSV)

The CSV lives on the drive; it does not touch the Apollo account. To also have the list *in Apollo* (under Lists), create each enriched person as a contact with `apollo_contacts_create`, passing `label_names: ["<list name>"]`. The label is the Apollo List, and it appears in the UI.

Two hard-won rules here:
- **Use single `apollo_contacts_create`, looped, not `apollo_contacts_bulk_create`.** Bulk-create returns a confident success payload but silently creates *ghost* contacts (`existence_level: "none"`): invisible in the UI, unsearchable, no label attached. Single create fully materializes the contact (`existence_level: "full"`) and creates + attaches the list. This is not intuitive and cost real debugging to find.
- **Verify, do not trust the response.** After creating, confirm with `apollo_contacts_search` (by name or keyword) that the contact actually landed. A success payload is not proof.

Apollo dedupes on create (it matches by email and updates the existing record rather than duplicating), and pushing enriched people into the account costs no credits, the enrichment already paid for the data.

## Sourcing beyond Apollo

This library keeps sourcing strictly in Apollo, because that is what it can automate. Other real sources exist (Sales Navigator, Google Maps for local SMBs, trade-show exhibitor lists, industry directories, government databases). They are manual and out of scope here. See `references/sourcing-beyond-apollo.md` for when each is worth the manual effort.

## In action: a real run

What actually happens when someone says "build my list":

1. **Check.** Claude confirms `profile.yaml` has a validated ICP. It does.
2. **Search.** Runs `apollo_mixed_people_api_search` page by page with the saved filters, writes `lists/acme/raw.csv`. Reports: "Found 3,180 people, saved to raw.csv."
3. **Dedupe.** Removes duplicates and caps at 3 per company, writes `deduped.csv`. Reports: "Removed 440 (120 duplicates, 320 over the per-company cap). 2,740 left."
4. **Confirm the spend.** "This will enrich 2,740 people and consume up to 2,740 credits (1 per match, no charge for unmatched). Do you want to proceed?" You say yes.
5. **Enrich.** Runs `apollo_people_bulk_match` in batches of 10 (274 calls), passing the search ids, writes `enriched.csv` with emails and phones.
6. **Score.** Reads each company's site for the pricing-page signal from `profile.yaml`, tags each row High, Medium, or Low priority, writes `scored.csv`.
7. **Verify.** Checks email status, runs the uncertain ones through Debounce, flags or drops the risky ones.
8. **Grade.** Runs `list-quality-scorecard`. Result: "Fix first, 9% catch-all addresses." You drop the catch-alls, re-grade to "Ready," and it writes `final.csv`.

You end with `final.csv` on your drive: clean, verified, prioritized, ready for Level 3. Every intermediate file is still there if you want to check the work.

## Common mistakes

- **Enriching before deduping and filtering.** You pay credits for leads you were going to cut.
- **Building 50,000 leads on an ICP you never sampled.** That is a Level 1 failure showing up as a Level 2 bill.
- **Skipping verification.** One bouncy list can take a domain down.
- **Writing to the Apollo CRM before dedupe.** The bulk-create tools make a new record every time, duplicates included.
- **Flooding on a few big domains.** Cap per-company concentration.
