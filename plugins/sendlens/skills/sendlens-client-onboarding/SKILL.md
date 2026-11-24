---
name: sendlens-client-onboarding
description: Use when an agency admin wants to add a new client to SendLens, connect a vendor workspace (Instantly, Smartlead, HeyReach, EmailBison), set up Slack alerts, subscribe an endpoint to SendLens events, or check whether a connection is syncing or why it is failing.
---

# Client onboarding

Set up a new client in SendLens. Only agency admins can do this. Every number follows [sendlens-honesty-labels](../sendlens-honesty-labels/SKILL.md).

SendLens is read-only toward vendors. Nothing here sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison. These tools change SendLens's own records only.

## Vendor keys never go through chat

Never ask anyone to paste a vendor API key into the chat, and never accept one. If someone pastes a key, tell them to rotate it in the vendor tool. `add_connection`, `rotate_connection_key` and `add_connection_webhook` don't take a key or secret.

`add_connection` and `rotate_connection_key` return a `connectUrl` (with `handoff: "web"`) and a `sendlens` command. Offer the link first: the person opens it in a browser and enters the key on SendLens's own page. It works once and expires at `expiresAt` (10 minutes); anyone holding it can enter one key for this client, so give it only to whoever holds the key. The command is the fallback for someone who prefers a terminal: the CLI asks for the key without showing it. If the result has no `connectUrl` (`handoff: "cli"`), give the command alone.

When the client holds the key, call `create_connect_link` with `clientId` (and `provider` or `workspaceId` if you know them) and `expiresInMinutes` up to 1440, so the admin can forward the link to the client. The client never sees anything but that page.

## Steps

1. Call `me`. If `role` is not `admin`, stop: tell the person an agency admin has to do this.
2. Call `clients` to check that the client doesn't exist already. Then call `create_client` with `name`.
3. Call `add_connection` with `clientId`, `provider` and `workspaceId`. `workspaceId` is a label the admin chooses, not an ID from the vendor tool, so there is nothing to look up and no need to ask for one: suggest the client's name in lowercase with dashes, such as `acme-co`, and use it unless the admin wants another. Clients that share one vendor workspace or account use the same label, each with its own `campaignScope`. Give the person `connectUrl` and say it works once, for 10 minutes. If they'd rather use a terminal, give them the command word for word, to run where `sendlens login` has been run. If the link expired, call `add_connection` again for a new one.
4. When they say it worked, call `connections` and check that the new connection is there. Then read its `sync` (see [Sync status](#sync-status)) and tell the person where the first sync stands.
   - For real-time replies, call `add_connection_webhook` with the `connectionId` and hand over its command the same way. The command prints a webhook URL that carries a secret; the person pastes it into the vendor tool themselves, since SendLens never registers webhooks. Never ask for that URL, or a Smartlead signing secret, in the chat. Running it again on an Instantly, HeyReach or EmailBison connection replaces the secret, and the old URL stops working.
   - Check it later with `connection_webhook`: a `status` of awaiting_secret means a Smartlead endpoint still needs its signing secret (the person runs the same command again and pastes the `whsec_` secret when asked), and `reconcile` compares what the webhooks delivered with what sync stored. Its null counts are unknown, not zero.
5. To change which campaigns a connection covers, call `set_connection_scope` with `connectionId` and `campaignScope`, or null for the whole workspace.
6. For Slack:
   - Call `slack_installations`. If the agency has none, call `slack_install` and give the person `authorizeUrl`. It works once, for 10 minutes.
   - Call `set_alert_channel` with `clientId`, `installationId`, `channelId` and `audience`. Use `client` when the client's own people read the channel. Ask the person to invite the SendLens bot to the channel.
   - Check with `alert_channels`.
7. For events, call `subscribe` with an https `url` and `eventTypes`. The signing secret is in this response only. Tell the person to store it now, and don't repeat it later. Check with `subscriptions`.

To replace a key later, call `rotate_connection_key` and hand over its `connectUrl`, or its command, the same way.

A link that no longer works says why on the page: used, expired, or too many failed attempts. Make a new one.

## Sync status

Each connection in `connections` has a `sync` object. Read it instead of guessing from a view's notes, which look the same whether a sync is 10 minutes in or has failed for an hour.

- `state` is one of: never_run (no sync has started; the first starts at `nextTickAt`), running, backoff (the vendor rate limited it; it waits until `nextTickAt`), succeeded, failed (the last pass failed once) or failing (2 or more passes in a row failed).
- When it is failed or failing, report `lastErrorCode` and `consecutiveFailures`. `Unauthorized` usually means the key was revoked or lacks access: call `rotate_connection_key` and hand over its `connectUrl`, or its command. For other codes, give the person the `connectionId` and the code, and don't guess at a cause.
- `collections` has each collection's `lastRun` and `lastSuccess`. A `lastSuccess` with `complete: false`, or `campaignsComplete` below `campaigns`, means that collection is still catching up. `null` counts are unknown, not zero.
- The connection's `syncCampaigns` says whether it syncs only active campaigns or all of them.
- Sync writes to SendLens first, and `query_sql` and the warehouse views read a warehouse that loads after. `warehouse.includesLastSync: false` means those don't have the newest sync yet, and `warehouse.due: true` means a load is queued. `nextTickAt` is the earliest the next sync can start; a busy tick may reach it later, and `null` means the connection or its client isn't active.

## When a tool refuses

- `admin_required`: only an agency admin can do this. Stop and tell the person to ask an admin. Don't retry.
- `view_not_in_scope`: the server URL this connection uses doesn't include that view. Tell the person which view you need, and that they can connect again with a URL that includes it.
- `client_not_in_scope`: the connection can't see that client. Call `clients` and pick one it can see, or ask the person to connect with a broader URL.
- `agency_scope_required` from `subscribe`: subscriptions need a connection with agency-wide access. Ask the person to connect with a URL that is not narrowed to clients.
