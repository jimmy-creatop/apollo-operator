---
name: apollo-icp-builder
description: "Turn a business and its ICP into a real Apollo search, a shared profile.yaml, and one to three custom scoring signals, validated on a sample. Use to define who to target on Apollo before building a list."
---

# Apollo ICP Builder (Level 1)

Turn a business into three things: a filled `client-profile.yaml`, a concrete Apollo search that returns the right people, and one to three custom scoring signals, all validated by a human on real samples before anyone builds a full list.

This is the front door. Nothing downstream is honest without it. A great sequence sent to the wrong list is a great sequence wasted.

## When to use

- Setting up outbound for a new business or campaign, with no `profile.yaml` yet.
- The ICP is vague ("B2B founders," "SMBs") and needs to become a real Apollo search.
- Re-targeting: the current list is not converting and you suspect the ICP, not the copy.

## Principle

Read `references/outbound-principles.md` if you have not. The two that drive this skill:
- **Scoring is custom, not a universal rubric.** Define one to three data points that actually separate good fit from bad for this client. Do not import a generic 100-point model.
- **Validate with samples.** Pull 50 to 100 real prospects and get a human "yes, this is my customer" before building the full list. Never skip this.

## Process

### 1. Anchor on the business
Start from the website. Read it, then say back in one paragraph: what they sell, who to, and the value proposition. Propose an ICP starting point (titles, industries, size) drawn from their case studies and homepage. Ask for corrections. This anchor saves an hour of interview.

If there is no website, get two sentences: what they sell and to whom.

### 2. Separate hard filters from soft preferences
For every criterion, ask: "If someone matches everything except this, do we still reach out?"
- **No** to hard filter (must match, goes in the search).
- **Yes** to soft preference (a personalization angle or a scoring signal, not a search filter).

Job title and industry are usually hard. Headcount is usually soft at the edges. Triggers (funded, hiring, tech installed) are almost never hard filters, they are signals.

### 3. Translate the ICP into Apollo filters
Map each hard criterion to a real Apollo field. Do not invent filters. The full field map with gotchas is in `references/apollo-search-reference.md`. The common ones:

- Titles to `person_titles` (set `include_similar_titles: false` for strict matches).
- Seniority to `person_seniorities` (`senior`, `manager`, `director`, `vp`, `c_suite`).
- Location to `person_locations`.
- Company size to `organization_num_employees_ranges` (e.g. `['51,200']`).
- Industry to `q_organization_keyword_tags` (e.g. `['SaaS']`) or NAICS/SIC codes.
- Revenue to `revenue_range` (raw integers, no symbols).
- Tech stack to `currently_using_any_of_technology_uids`.

### 4. Define one to three scoring signals
What separates a good-fit account from a barely-fit one for this business? Pick one to three. Two kinds:

- **Filter signals** map to a native Apollo field, so they narrow the search directly. Example: "has a real sales team" to `organization_department_or_subdepartment_counts: {master_sales: {min: 5}}`. "Recently funded" to `latest_funding_date_range` (company search).
- **Research signals** need a look at the company. Example: "has a pricing page," "sells a specific product line." Apollo's own AI research is not on the MCP, so Claude does this natively: read the site, decide, tag the record. This runs at list time (Level 2), but define the rule here.

Each lead ends up tagged **High, Medium, or Low** priority (that is all the user sees). If you have nothing to start from, begin with Apollo's built-in Bronze-to-Platinum fit score, then upgrade to your own one to three signals as the campaign teaches you what predicts fit. See `references/scoring-starter-frameworks.md`. Always frame scoring as custom, never as the one true rubric.

### 5. Pick the buying-committee personas
Decide who at the account you are reaching. Set `person_seniorities` and `person_titles` for each persona (champion, economic buyer, end user). Keep the list tight. How you *write* to executives vs managers (the ATL/BTL split) is Level 3, not here.

### 6. Build and sanity-check in Apollo
Run `apollo_mixed_people_api_search` with the filters, `per_page: 10`, `page: 1`, and read the total available count.
- Over 50,000: too broad (that is Apollo's display ceiling anyway). Tighten with more filters.
- A few hundred or fewer: too narrow. Loosen a soft edge, or widen titles/geo.
- A workable range: good. Note that search does not return emails and may mask last names; enrichment happens at Level 2.

The count is the `total_entries` field in the response. **When it comes back huge, do not just say "tighten," find what is inflating it.** Broad title buckets ("manager", "events", "marketing") plus `include_similar_titles: true` can multiply the count 100x or more. Re-run once with `include_similar_titles: false` and only the strict core titles to see the real base, then widen on purpose. Report the difference to the user so they choose the breadth, for example: "679 people have a literal trade-show title; 91,000 if we widen to all marketing and events roles." One inflated number trusted blindly is how bad lists start.

If you use `apollo_mixed_companies_search`, it costs 1 credit per call that returns results. Confirm first with the exact words: "This will consume 1 credit. Do you want to proceed?"

### 7. Validate with samples
Pull 50 to 100 people from the search. Put them in front of a human: "Is this your ideal customer?" Walk a few concrete examples, not just the count. Document every "avoid X" and "prioritize Y" as an exclusion or a soft preference. Do not proceed to the full list build until the samples pass.

### 8. Write the profile
Save the ICP, the scoring signals, and the exact Apollo filter set to `profile.yaml`. The filter set is the source of truth (the MCP cannot create a named Apollo saved search, so store the filters, so store the filters, and optionally recreate the saved search by hand in the Apollo UI).

```yaml
icp:
  titles: [...]
  seniority: [...]
  industries: [...]
  headcount: <range>
  geos: [...]
  exclusions: [...]
scoring_signals:
  - name: <e.g. sales team >= 5>
    type: filter | research
    apollo: <field, if a filter signal>
    rule: <how to decide, if a research signal>
apollo:
  people_search_filters: { <the exact param object you ran> }
```

## Output

A filled `profile.yaml` with a validated ICP, one to three scoring signals, and a reproducible Apollo filter set. Hand off to **Level 2 (list building)** to enrich, apply research signals, grade, and build the full list.

## Common mistakes

- Treating a trigger (funded, hiring) as a hard filter. It shrinks the list 20x and it is a signal, not a requirement.
- Building the full list before samples are approved.
- Importing a generic scoring rubric instead of finding the one to three signals that matter here.
- Titles too broad. "Manager" alone pulls every department. Pair titles with seniorities and departments.
