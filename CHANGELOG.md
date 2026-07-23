# Changelog

All notable changes to Apollo Operator are documented here. This project uses
[semantic versioning](https://semver.org/): MAJOR.MINOR.PATCH.

## v1.1.0 — 2026-07-23

Apollo shipped a CLI and formalized three ways to run headless (MCP, CLI, raw
API). This release makes Apollo Operator aware of all three, and adds the two
skills that were missing from the front and back of the motion: understanding
the business before targeting, and building the sending stack before sending.
16 skills total.

### Added

- **`apollo-cli`** (Level 0, Access): the three lanes to run Apollo headless
  (MCP, CLI, raw REST API), the MCP-vs-CLI tradeoff (context cost,
  composability, determinism), when Operator reaches for each, CLI setup
  (`apollo auth login`, OAuth), and a zero-credit `GET /v1/auth/health`
  preflight. The library stays MCP-native by default; CLI-backed variants of
  the search and enrich steps are planned for a future release.
- **`business-brief`** (Level 0.5): produces one readable `brief.md` capturing
  the business (identity, offer, buyers, pains, objections, proof, voice), the
  narrative context layer every other skill reads. Includes a fill-in
  `brief-template.md`.
- **`sending-infrastructure`** (Level 4, Setup): build the sending stack from
  zero, dedicated domains separate from your primary, mailboxes, DNS
  (SPF, DKIM, DMARC, MX), redirect, and warmup, plus the volume math for how
  many mailboxes and domains a given daily send target needs. Includes a
  `dns-records.md` reference and pre-send checklist.

### Changed

- **`apollo-multichannel`**: reframed the human-in-the-loop design as the
  LinkedIn safety advantage (you act from your own profile, not an API that
  risks a ban), and added explicit pacing: keep connection requests under 30
  per day, 20 to be safe, and trickle them across days rather than clearing
  the queue in one session.
- **`operator-context`**: added the three-lane access note and routing, the
  two infrastructure read tools (`apollo_domain_purchase_index`,
  `apollo_email_account_purchase_index`) to the tool map, and the two-layer
  context model (`brief.md` narrative + `profile.yaml` machine).
- **`apollo-icp-builder`**: reads `brief.md` first when it exists, and offers
  to build one before targeting.
- **`apollo-deliverability`** and **`apollo-go-live`**: point back to
  `sending-infrastructure` when no sending stack exists yet.

## v1.0.0 — 2026-07-06

The launch release. 13 skills turning the raw Apollo MCP into a guided
outbound motion, built and tested live on a real Apollo account.

### Added

- **`operator-context`**: the router. Apollo tool map, universal
  deliverability and copy rules, shared `profile.yaml`, routing to every skill.
- **`apollo-icp-builder`**: ICP into a real Apollo search plus custom scoring
  signals.
- **`apollo-list-builder`** and **`list-quality-scorecard`**: search, enrich,
  verify, and grade a list before it sends.
- **`apollo-sequence-builder`** and **`sequence-reviewer`**: write and review
  short sequences matched to the persona, built as inactive drafts.
- **`apollo-multichannel`**: optional LinkedIn, call, and manual steps plus the
  task queue.
- **`apollo-go-live`**: enroll, human approval, and pulling contacts.
- **`apollo-deliverability`**: mailbox health, warmup, and the bounce rules.
- **`positive-reply-scoring`**, **`experiment-design`**, **`weekly-rhythm`**:
  measure intent, run clean experiments, keep a cadence.
- **`apollo-signals`**: find the offer-fit reason to reach out now.
