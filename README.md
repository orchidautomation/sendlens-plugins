# SendLens plugins

[SendLens](https://sendlens.app) lets your agents read an agency's cold outbound across Instantly, Smartlead, HeyReach and EmailBison, with every number labelled for how far it can be trusted.

This repository is a Claude Code plugin marketplace. It holds one plugin, `sendlens`, with two parts:

- **The MCP server config.** It points at the SendLens remote MCP server, `https://api.sendlens.app/mcp`. Sign-in is OAuth: your client opens the SendLens sign-in page the first time it connects. There are no tokens or keys in this repository.
- **Skills.** Short procedures that tell an agent which SendLens tools to call and how to report what they return.

SendLens is read-only toward vendors. No tool sends, pauses, edits or deletes anything in Instantly, Smartlead, HeyReach or EmailBison. The write tools change SendLens's own records only, such as clients, subscriptions and alert channels.

## Skills

| Skill | Use it for |
| --- | --- |
| [sendlens-weekly-client-review](plugins/sendlens/skills/sendlens-weekly-client-review/SKILL.md) | A client-ready weekly summary |
| [sendlens-reply-triage](plugins/sendlens/skills/sendlens-reply-triage/SKILL.md) | Replies that need an answer, with drafts for a person to send |
| [sendlens-campaign-health](plugins/sendlens/skills/sendlens-campaign-health/SKILL.md) | Bounce and reply rates, mailbox health and lead runway |
| [sendlens-client-onboarding](plugins/sendlens/skills/sendlens-client-onboarding/SKILL.md) | Adding a client, its vendor connections, Slack alerts and event subscriptions (admins only) |
| [sendlens-insights](plugins/sendlens/skills/sendlens-insights/SKILL.md) | Running insights on the stored history across tools and clients |
| [sendlens-honesty-labels](plugins/sendlens/skills/sendlens-honesty-labels/SKILL.md) | How to read and phrase every SendLens number |

## Install

### Claude Code

In a Claude Code session:

```text
/plugin marketplace add orchidautomation/sendlens-plugins
/plugin install sendlens@sendlens
```

Or from a terminal:

```sh
claude plugin marketplace add orchidautomation/sendlens-plugins
claude plugin install sendlens@sendlens
```

Then run `/mcp` in a session, pick `sendlens` and sign in.

The MCP server only, without the skills:

```sh
claude mcp add --transport http sendlens https://api.sendlens.app/mcp
```

### Cursor

Add this to `~/.cursor/mcp.json`, or to `.cursor/mcp.json` in a project:

```json
{
  "mcpServers": {
    "sendlens": {
      "url": "https://api.sendlens.app/mcp"
    }
  }
}
```

Cursor signs in with OAuth when it first connects.

### ChatGPT

Custom MCP servers need a Business, Enterprise or Edu workspace, and the workspace has to allow them. Open **Plugins**, select the plus button, then **Add custom MCP server**. Enter `https://api.sendlens.app/mcp` as the URL and choose OAuth. Older workspaces put this under **Settings → Connectors → Advanced → Developer mode**, then **Create**.

### Claude Desktop and claude.ai

Open **Customize → Connectors**, select **Add**, then **Custom → Web**. Enter `https://api.sendlens.app/mcp` and continue through sign-in. On Team and Enterprise plans an owner adds the connector under **Organization settings → Connectors** first, and members then select **Connect**.

### Codex

```sh
codex mcp add sendlens --url https://api.sendlens.app/mcp
codex mcp login sendlens
```

`codex mcp login` opens the SendLens sign-in page in your browser.

## Scoped URLs

The URL sets what a connection can see. Each sign-in gets a token for that URL only.

```text
https://api.sendlens.app/mcp
https://api.sendlens.app/mcp/clients/<client ids>
https://api.sendlens.app/mcp/clients/<client ids>/views/<views>
https://api.sendlens.app/mcp/clients/<client ids>/views/<views>/unmasked
```

- **Client ids** are comma-separated and sorted.
- **Views** are comma-separated, in this order: `replies`, `unanswered`, `lead_timeline`, `metrics`, `events`, `mailbox_health`, `runway`, `client_report`, `query_sql`, `admin`. A tool outside the URL's views is hidden and answers `view_not_in_scope`.
- **`/unmasked`** shows prospect names and emails. Only agency admins can use it. Without it, emails and names are masked.
- **Admin tools** (clients, connections and Slack) answer `admin_required` to members and client viewers.

Use a scoped URL anywhere the instructions above use `https://api.sendlens.app/mcp`.

With the Claude Code plugin, set `SENDLENS_MCP_URL` before starting Claude Code. The plugin's `.mcp.json` reads `${SENDLENS_MCP_URL:-https://api.sendlens.app/mcp}`, so it falls back to the default URL when the variable is unset. Set it to a full URL, not an empty string: an empty value counts as set and gives an invalid URL.

```sh
export SENDLENS_MCP_URL=https://api.sendlens.app/mcp/clients/<client ids>
claude
```

## Use the skills without the plugin

Clients that don't load Claude Code plugins can still use the skills. Each skill is one Markdown file with a short header, in [`plugins/sendlens/skills`](plugins/sendlens/skills).

- **Claude Code without the plugin:** copy each skill folder to `~/.claude/skills/`, or to `.claude/skills/` in a project.
- **Codex:** copy each skill folder to `~/.agents/skills/`, or to `.agents/skills/` in a project.
- **claude.ai and Claude Desktop:** zip a skill folder and upload it under **Customize → Skills**.
- **Cursor, ChatGPT and other clients:** paste the body of the skills you need into the client's rules or project instructions. Always include sendlens-honesty-labels.

## Check the plugin

```sh
claude plugin validate .
claude plugin validate ./plugins/sendlens
```

## License

[MIT](LICENSE)
