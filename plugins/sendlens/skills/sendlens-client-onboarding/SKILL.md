---
name: sendlens-client-onboarding
description: Use when an agency admin wants to add a new client to SendLens, connect a vendor workspace (Instantly, Smartlead, HeyReach, EmailBison), set up Slack alerts, or subscribe an endpoint to SendLens events.
---

# Client onboarding

Set up a new client in SendLens. Only agency admins can do this. Every number follows [sendlens-honesty-labels](../sendlens-honesty-labels/SKILL.md).

SendLens is read-only toward vendors. Nothing here sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison. These tools change SendLens's own records only.

## Vendor keys never go through chat

Never ask anyone to paste a vendor API key into the chat, and never accept one. If someone pastes a key, tell them to rotate it in the vendor tool. `add_connection`, `rotate_connection_key` and `add_connection_webhook` don't take a key or secret. They return a `sendlens` command for a person to run in their own terminal, where the CLI asks for the key without showing it.

## Steps

1. Call `me`. If `role` is not `admin`, stop: tell the person an agency admin has to do this.
2. Call `clients` to check that the client doesn't exist already. Then call `create_client` with `name`.
3. Call `add_connection` with `clientId`, `provider` and the vendor `workspaceId`. Add `campaignScope` when several clients share one vendor workspace. Give the person the command it returns, word for word, and ask them to run it in a terminal where `sendlens login` has been run.
4. When they say it worked, call `connections` and check that the new connection is there.
   - For real-time replies, call `add_connection_webhook` with the `connectionId` and hand over its command the same way. The command prints a webhook URL that carries a secret; the person pastes it into the vendor tool themselves, since SendLens never registers webhooks. Never ask for that URL, or a Smartlead signing secret, in the chat. Running it again on an Instantly, HeyReach or EmailBison connection replaces the secret, and the old URL stops working.
   - Check it later with `connection_webhook`: a `status` of awaiting_secret means a Smartlead endpoint still needs its signing secret (the person runs the same command again and pastes the `whsec_` secret when asked), and `reconcile` compares what the webhooks delivered with what sync stored. Its null counts are unknown, not zero.
5. To change which campaigns a connection covers, call `set_connection_scope` with `connectionId` and `campaignScope`, or null for the whole workspace.
6. For Slack:
   - Call `slack_installations`. If the agency has none, call `slack_install` and give the person `authorizeUrl`. It works once, for 10 minutes.
   - Call `set_alert_channel` with `clientId`, `installationId`, `channelId` and `audience`. Use `client` when the client's own people read the channel. Ask the person to invite the SendLens bot to the channel.
   - Check with `alert_channels`.
7. For events, call `subscribe` with an https `url` and `eventTypes`. The signing secret is in this response only. Tell the person to store it now, and don't repeat it later. Check with `subscriptions`.

To replace a key later, call `rotate_connection_key` and hand over its command the same way.

## When a tool refuses

- `admin_required`: only an agency admin can do this. Stop and tell the person to ask an admin. Don't retry.
- `view_not_in_scope`: the server URL this connection uses doesn't include that view. Tell the person which view you need, and that they can connect again with a URL that includes it.
- `client_not_in_scope`: the connection can't see that client. Call `clients` and pick one it can see, or ask the person to connect with a broader URL.
- `agency_scope_required` from `subscribe`: subscriptions need a connection with agency-wide access. Ask the person to connect with a URL that is not narrowed to clients.
