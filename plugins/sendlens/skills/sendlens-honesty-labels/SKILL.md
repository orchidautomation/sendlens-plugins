---
name: sendlens-honesty-labels
description: Use before stating any number from SendLens, such as sent, replied, bounced, a rate, runway or a report total. Explains null counts, meta.complete, meta.stale, meta.evidence and meta.notes, and how to phrase a number so it is not overstated.
---

# SendLens honesty labels

Every SendLens answer says how far its numbers can be trusted. Read the labels before you repeat a number.

SendLens is read-only toward vendors. Nothing here sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison.

## Where the labels are

- List and item tools return `meta` with `complete`, `stale`, `evidence`, `syncedAt` and `notes`.
- Rows from `mailbox_health`, `runway` and `client_report` campaigns carry their own `evidence`, `complete` and `stale`.
- `run_insight` returns outbound-sdk's own labels in `data.sdk` (`evidence`, `complete`, `stale`, `notes`) beside `meta`. Use the weaker of the two.
- `client_report` also returns `facts` (what the numbers support) and `caveats` (what they don't).

## What each label means

- **null:** the vendor did not report this count. It is unknown. It is never 0.
- **A rate that is null:** one of its inputs is unknown. Don't work it out yourself from other numbers.
- `meta.complete` false: at least one connection in scope has not finished a full sync. Numbers may be missing rows.
- `meta.stale` true: at least one connection has not synced in the last 24 hours. Say when the data is from, using `meta.syncedAt`.
- `meta.evidence`:
  - `exact`: counted from the full records.
  - `sampled`: measured on a sample. It shows a pattern, not an exact count.
  - `derived`: worked out from other numbers, such as a rate.
- `meta.notes`: why the view is incomplete or stale, in plain words. Pass these on.

## How to phrase a number

- Exact, complete and fresh: state it plainly. "412 emails sent last week."
- Sampled: "about", plus where it came from. "About 3% of replies in a sample were positive."
- Derived: name the inputs. "A 1.9% reply rate (replied / sent)."
- Incomplete: "at least" or "so far", and say why. "At least 412 sent; one workspace is still syncing."
- Stale: give the date. "As of Nov 20, 412 sent."
- Null: "unknown" or "not reported by the vendor". Never "0" or "none".

## Adding numbers up

- Never treat null as 0 in a sum, average or rate.
- If some parts are null, give the sum of the known parts and say how many parts it covers. "412 sent across 3 of 4 campaigns; the fourth didn't report sends."
- A total built from incomplete, stale or sampled parts takes the weakest label of its parts.

## If a number looks wrong

Call `report_issue` with category `wrong_number`. Use placeholders such as `<campaign>` and `<lead>`, never real names, emails or IDs. Reports with prospect data are turned back.
