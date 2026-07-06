---
name: weekly-rhythm
description: "The weekly operating cadence for running outbound continuously: Monday deliverability, Wednesday reply sweep, Friday retrospective, plus biweekly, monthly, and quarterly reviews. Use to run outbound as an ongoing system."
---

# Weekly Rhythm (Level 5)

Having a dozen skills does not help if you do not run them on a schedule. This is the schedule. What separates a hobbyist from a real operator is not tooling, it is doing the boring cadence every week without fail.

This is a pure playbook: no automation, no reminders on purpose. Put it on your own calendar and run the prescribed skill at the prescribed time.

## Put it on your calendar (do this first)

Create these as recurring events, exactly:

| Event | Cadence |
|---|---|
| Monday deliverability check | Every Monday |
| Wednesday positive-reply sweep | Every Wednesday |
| Friday campaign retrospective | Every Friday |
| Inbox rotation | Every other Monday |
| Monthly spam-placement check | 1st of the month |
| Quarterly review | First Monday of the quarter |

## Monday: deliverability (15 min)

Run `apollo-deliverability` on active campaigns. Check bounce rate and mailbox health over the last 7 days. If anything is flagged, fix deliverability before touching anything else. If clean, log it and move on.

## Wednesday: positive-reply sweep (30 to 60 min)

Run `positive-reply-scoring` on everything active. Respond to interested replies within minutes. Reach referrals within a day. Read any hostile replies yourself and check for a targeting problem. If you are getting more than a handful of positive replies a week, this is where you hand them to whoever closes.

## Friday: retrospective (20 min per campaign hitting 21 days)

For each campaign that just hit its 21-day mark, run `positive-reply-scoring` and decide:
- **Winner** (well above your baseline): keep it, consider scaling to more mailboxes.
- **Middling:** iterate. Plan the next single-variable test with `experiment-design`.
- **Loser:** kill it, and write down why. The loss is a lesson if you record it.

Log the decision and the reasoning. Over a quarter these notes are what you learn from.

## Every other Monday: inbox rotation (30 min)

Review mailbox health (`apollo_email_accounts_index`). Retire failing mailboxes, promote warmed backups into rotation. If your backup pool is thin, start warming new domains now, it takes weeks.

## Monthly: spam-placement check

Run a spam-placement test (Apollo's inbox-placement tooling, see the KB map). Above 90% inbox placement, keep going. Below 80%, pause and fix before sending more.

## Quarterly: review

Read the quarter's retrospectives. Which campaigns, lists, and angles worked? Which ICP converted best? Adjust the ICP in `profile.yaml` if the data points somewhere better, and set the next quarter's experiments.

## What to skip

You do not need to check Apollo every day. Daily poking is procrastination dressed as diligence. Wait for 7-day averages, and trust the rhythm.
