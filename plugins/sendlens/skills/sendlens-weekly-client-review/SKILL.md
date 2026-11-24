---
name: sendlens-weekly-client-review
description: Use when someone asks for a weekly or Monday client review, a client update, a "how did last week go" summary, or a report to send to a client about their cold outbound.
---

# Weekly client review

Build a summary a client can read, from SendLens data. Every number follows [sendlens-honesty-labels](../sendlens-honesty-labels/SKILL.md).

SendLens is read-only toward vendors. Nothing here sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison.

## Steps

1. Call `clients` and confirm which client. If the person named one you can't see, see "When a tool refuses".
2. Call `client_report` with `clientId` and `weekStart` (the Monday of the UTC week). This is the base of the review.
3. Call `daily_metrics` for the same week if you need a day-by-day trend.
4. Call `mailbox_health` for the client. List mailboxes with `status` `at_risk` or `watch`.
5. Call `runway` for the client. Flag campaigns with fewer than 14 `runwayDays`, and campaigns where runway is null.
6. Call `unanswered` for the client with `order` `oldest`, so the longest wait comes first. Page with `cursor` until `nextCursor` is null, count positive replies still waiting from their labels, and give the oldest wait. The snippets are enough for counting; don`t fetch whole replies.
7. Call `run_insight` with `name` `period_comparison` for this week against the last, and `campaign_ranking` for the best and worst campaigns, each with `clientId`. Use their intervals: a change inside the interval is noise. See [sendlens-insights](../sendlens-insights/SKILL.md).
8. Write the summary.

## The summary

- **Headline:** two or three sentences. Use the report's `facts`.
- **Numbers:** sent, replied, positive and bounced per campaign, with the reply rate against the week before. Give each number its label.
- **Replies waiting:** how many positive replies need an answer, and the oldest wait in hours.
- **Risks:** mailboxes at risk and campaigns running out of leads.
- **What we can't tell yet:** every item in `caveats` and `meta.notes`, and every null count.

Don't quote prospect text in a client summary. Describe replies by label and count.

## When a tool refuses

- `admin_required`: only an agency admin can do this. Stop and tell the person to ask an admin. Don't retry.
- `view_not_in_scope`: the server URL this connection uses doesn't include that view. Tell the person which view you need, and that they can connect again with a URL that includes it.
- `insight_unsupported`: SendLens doesn't store what that insight needs. Leave it out and say what is missing.
- `client_not_in_scope`: the connection can't see that client. Call `clients` and pick one it can see, or ask the person to connect with a broader URL.
