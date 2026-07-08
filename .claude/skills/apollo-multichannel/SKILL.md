---
name: apollo-multichannel
description: "Optional layer on top of email: add LinkedIn, call, and manual steps to an Apollo sequence and work the task queue they create. Apollo automates email only; the rest are human tasks. Use to run multichannel outbound on Apollo."
---

# Apollo Multichannel (optional)

Email is the automated core of this library. This skill is the optional layer on top: add LinkedIn, call, and manual steps to a sequence, and work the task queue those steps create. It is opt-in on two levels. You do not have to add multichannel steps to a sequence, and you do not have to use this skill at all. A clean email-only sequence is a complete, valid campaign.

## The one thing to be honest about

**Apollo automates email only.** Every non-email step is a **task for a human to do**, not something Apollo performs. A `linkedin_step_message` does not send a LinkedIn message; it drops a task in the rep's queue that says "send this message." A `call` step queues a call task. So multichannel here means "automated email plus a structured human task queue for LinkedIn and calls." Never tell the operator Apollo will send the LinkedIn touch or place the call. It queues the work; a person does it.

Note: call and LinkedIn steps require an Apollo plan with task functions, so they may return a plan error. If they do, that capability is gated on your plan, and email-only still works.

## Part A: add multichannel steps to a sequence

Multichannel steps go in the same `apollo_sequences_create` call as the email steps (see `../apollo-sequence-builder`), you just use more step types. The step types:

| Step type | What it is | Needs content? |
|---|---|---|
| `auto_email` | Automated email (the core) | Yes (`emailer_touches`) |
| `manual_email` | Email the rep reviews and sends | Yes (`emailer_touches`) |
| `linkedin_step_view_profile` | Soft touch, just view the profile | No |
| `linkedin_step_connect` | Send a connection request | Yes (short note) |
| `linkedin_step_message` | Message a 1st-degree connection | Yes (message body) |
| `linkedin_step_interact_post` | Like or comment on a recent post | No |
| `call` | Call task, with a script in `note` and a `priority` | No touch; use `note` |
| `action_item` | Generic manual to-do | No |

A sensible, non-annoying default cadence (relative `wait_time`, so Day 0/2/3/5/8):
1. `auto_email`, wait 0
2. `linkedin_step_view_profile`, wait 2 (warm the name before the next email)
3. `auto_email`, wait 1
4. `linkedin_step_message`, wait 2 (only fires if they connected; write it like a human)
5. `call`, wait 3, `priority: high`, `note` = a two-line call script

Rules that carry over: still build the sequence `active: false`, still let a human activate it, still keep the email copy to the Level 3 standard. LinkedIn and call steps do not change any of that.

## Part B: work the task queue

The steps above (and any standalone tasks) create tasks a person clears each day. The tools:

- **See what is due:** `apollo_tasks_search` with no status filter returns open (scheduled) tasks. Sort `task_due_at` ascending for soonest-first. Filter by `task_type_cds` (for example the four `linkedin_step_*` values for a LinkedIn block, or `phone_call` for a call block), by `priority`, by `date_range`, or by `emailer_campaign_id` to work one sequence's tasks.
- **Clear them:** `apollo_tasks_complete` when done, `apollo_tasks_skip` when skipping, `apollo_tasks_update` to reschedule or re-prioritize.
- **Create work in bulk:** `apollo_tasks_bulk_create` to drop a task on a whole segment, for example a `call` task on every High-priority lead who opened three times. Each task needs `user_id` (the assignee), `type`, and at least one association (`contact_id`, `account_id`, or `opportunity_id`). Put call scripts in `note`; put LinkedIn or email content in `standalone_outreach_task_message`.

A good weekly habit: batch by channel. Work all LinkedIn tasks in one sitting, all calls in another. Context-switching per lead is what makes multichannel feel like a slog. This slots into `../weekly-rhythm`.

## Common mistakes

- **Saying Apollo "sends" the LinkedIn message or "makes" the call.** It queues a task. A human acts.
- **Turning every lead into a five-channel gauntlet.** Reserve calls and LinkedIn for High-priority leads (from Level 2 scoring or a signal match). Blanket multichannel is just more noise, faster.
- **A task with no association.** `contact_id`, `account_id`, or `opportunity_id` is required, or the create fails.
- **Forgetting these are gated.** If task-function steps error, it is your Apollo plan, not your setup. Fall back to email-only.
