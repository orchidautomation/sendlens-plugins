---
name: sendlens-campaign-health
description: Use when someone asks whether campaigns or mailboxes are healthy, about bounce rates, reply rates, deliverability, sending pace, or how long campaigns' leads will last.
---

# Campaign health

Check bounce and reply rates, mailbox health and lead runway. Every number follows [sendlens-honesty-labels](../sendlens-honesty-labels/SKILL.md).

SendLens is read-only toward vendors. Nothing here sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison. You can point out a problem and suggest a fix. A person makes the change in the vendor tool.

## Steps

1. Call `mailbox_health` with `clientId` and `days` (7 is a good start). Each mailbox has `bounceRate`, `replyRate` and `status`:
   - `at_risk`: bounce rate 5% or more.
   - `watch`: bounce rate 2% or more.
   - `healthy`: below that.
   - null: the vendor did not report sent or bounced counts, so there is no call. Say so. Don't mark it healthy.
2. Call `daily_metrics` with `clientId`, `from` and `to` to see when a rate changed. It is latest day first, 20 rows a page; page with `cursor` and the same `order`, or pass `order` `oldest` to read forward in time.
3. Call `runway` with `clientId`. `runwayDays` is the leads left divided by the daily pace. It is null when the pace is 0 or unknown.
4. For a campaign that looks wrong, call `run_insight` with `name` `campaign_diagnosis` and the campaign in `input`, and `anomalies` for the days that moved. `bounce_watch`, `sender_capacity` and `warmup_readiness` check sending across campaigns. They run on stored history with no vendor request; see [sendlens-insights](../sendlens-insights/SKILL.md).
5. For a question these tools don't answer, call `sql_catalog` first to see the tables and columns you may read. Then call `query_sql` with one `SELECT`. Tables are written `sendlens.<table>`. Keep the `evidence`, `complete` and `stale` columns in your query and in your answer.

## Stale and incomplete data

- If `meta.stale` is true, give the date of the data from `meta.syncedAt` before any finding.
- If `meta.complete` is false, say the finding may change once the sync finishes, and pass on `meta.notes`.
- Don't call a mailbox or campaign healthy from incomplete or null data.

## The answer

Lead with the mailboxes and campaigns that need action. For each one, give the number, its label and one suggested fix.

## When a tool refuses

- `admin_required`: only an agency admin can do this. Stop and tell the person to ask an admin. Don't retry.
- `view_not_in_scope`: the server URL this connection uses doesn't include that view. Tell the person which view you need, and that they can connect again with a URL that includes it.
- `insight_unsupported`: SendLens doesn't store what that insight needs. Say so with the message, and use the other tools.
- `client_not_in_scope`: the connection can't see that client. Call `clients` and pick one it can see, or ask the person to connect with a broader URL. If `query_sql` answers `sql_rejected`, read the message, fix the query and try once more.
