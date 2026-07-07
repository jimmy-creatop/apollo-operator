---
name: apollo-sequence-builder
description: "Write a 3-step, 2-variation cold email sequence matched to the persona (ATL or BTL) and build it in Apollo as an inactive draft. Use to write and create outbound sequences on Apollo."
---

# Apollo Sequence Builder (Level 3)

Write the sequence, then build it in Apollo as an inactive draft ready for a human to review and activate. Copy is where most outbound dies, so this level is opinionated: short, one idea per email, matched to the person you are writing to, and never activated by the machine.

## When to use

- `profile.yaml` has the offer, the ICP, and the voice.
- A graded list exists (Level 2 done), or you at least know the persona you are writing to.

## The rules that do not move

From `../operator-context/references/outbound-principles.md` and the core copy rules. Non-negotiable regardless of framework:
- **3 steps, 2 variations per step.** Day 0, Day 3, Day 7.
- **60 to 90 words** per email. Step 3 is the shortest.
- **One CTA, soft.** "Worth a quick look?" not "Book a 30-minute demo."
- **One focus per email: outcome or mechanism, never both.**
- **Rotate the angle across steps.** Never repeat the same value prop.
- **Subject: lowercase, 1 to 3 words, reused across the thread.**
- **Plain text. No em dashes. No buzzwords** (leverage, synergy, revolutionary, best-in-class). Oxford comma.
- **Step 3 is a routing question or a soft breakup. No new pitch.**

## Process

### 1. Match the person (ATL or BTL)
Before choosing words, decide who you are writing to. Executives (VP, C-level, director) and managers/ICs read completely different emails. See `references/atl-btl.md`. This decides length and language for the whole sequence. Pull seniority from the list or `profile.yaml`.

### 2. Pick the angle
Choose a framework that fits the offer and the person from `references/frameworks.md` (Do the Math, Ask Before Pitch, Challenge of Similar Companies, and others). One angle per email, a different one each step.

### 3. Write the three steps
- **Step 1 (Day 0):** one focus (a pain, an outcome, a question, or specific proof), a personalized opener, one soft CTA.
- **Step 2 (Day 3, same thread):** context and credibility, a different angle, a real case snippet, shorter than step 1.
- **Step 3 (Day 7, same thread):** routing ("am I reaching the right person?") or a soft breakup. No new pitch.
- Write **2 variations** of steps 1 and 2, each testing one idea (different pain, different proof, with vs without a lead magnet).

### 4. Personalize the opener honestly
The opener should read like you spent ten minutes on this person. Claude does the research (reads the site, finds a real, specific fact) and writes one suggestive opening line, merged per lead. Do not rely on a black-box AI opener, and never fake specificity. (Apollo has its own per-recipient AI opener, `has_personalized_opener`, which spends credits; our default is to write the opener ourselves so a human can approve it.)

### 5. Review, then build
Run `sequence-reviewer` on the draft. Fix what it flags. Then build it in Apollo with `apollo_sequences_create`:
- `active: false`. Always. The machine never turns a sequence on.
- 3 `auto_email` steps. Step 1 is a `new_thread`; steps 2 and 3 are `reply_to_thread` (same thread, no subject, keeps the conversation).
- Delays are **relative to the previous step**: step 1 `wait_time: 0`, step 2 `wait_time: 3`, step 3 `wait_time: 4` (that is Day 7, not Day 11). This is the mistake everyone makes.
- 2 `emailer_touches` per step (the A/B variants). `creation_type: manual` (a human wrote it). Tone from the profile voice (Direct, Formal, or Casual).
- **Format the body so it breaks into paragraphs.** Write 2 to 3 short paragraphs, not one wall of text. Separate them with `<br><br>` inside a single `<div>`, like `<div>First paragraph.<br><br>Second paragraph.<br><br>Closing line and CTA.</div>`. This is not cosmetic: live-tested on Apollo, `<br><br>` is the only structure Apollo keeps as real line breaks in the sent text. Multiple `<div>` blocks collapse to a single space and `<p>` tags collapse to nothing, so both send as a run-on paragraph. Never dump the whole email into one flat `<div>`.
- **Subject A/B is deliberate, not a bug.** Both variants of a step reuse the same subject on purpose: the subject has to carry across steps so 2 and 3 reply in-thread, and the A/B test changes only the body angle (one variable). If you do want to test subjects, test them on step 1 only, where each lead still threads under whichever subject it got.

### 6. Hand to a human to activate
Report the sequence id and a short step summary. Activation is `apollo_emailer_campaigns_approve`, and only a person runs it, after confirming the sender mailbox, the schedule, and the copy. Recommend, do not activate.

## Common mistakes

- Selling the outcome and the mechanism in one email. Pick one.
- A new pitch in step 3. It is for routing or a graceful exit.
- Absolute delays. Day 7 is `wait_time: 4` after a Day 3 step, not 7.
- Activating without a human. Never.
- A "personalized" opener that is generic ("love what you're doing at {{company}}"). That is theater, and it reads like it.
- Dumping the whole email into one `<div>`. It sends as a wall of text. Break it with `<br><br>` (see step 5).
