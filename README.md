# Terrads for your coding agent

Make ad creatives for your brand's products from your coding agent, such as Claude Code or Cursor, or any MCP client. This repository is a Claude Code plugin and an Agent Skill for the Terrads MCP server: the plugin signs in at `https://mcp.terrads.com/mcp`, and clients that send an API key connect to `https://terrads.com/api/mcp`.

Included with every paid Terrads plan. You need a Terrads account on a paid plan. The plugin signs in with your Terrads account in the browser; other agents use an API key from **Settings → Integrations → API keys** at terrads.com (shown once, when created).

## Install

**Claude Code plugin** (recommended — no API key needed). In a session:

```text
/plugin marketplace add terrads-hq/plugin
/plugin install terrads@terrads
```

or from a terminal:

```text
claude plugin marketplace add terrads-hq/plugin
claude plugin install terrads@terrads
```

Enable the plugin when asked. Then run `/mcp`, pick `terrads` and choose to sign in: your browser opens Terrads, you sign in, pick the team and select **Allow**. The plugin connects the `terrads` MCP server and installs the `terrads` skill.

**Skill only**, for agents that manage their own MCP connections:

```text
npx skills add terrads-hq/plugin
```

Then connect the MCP server in your client with an API key, following https://terrads.com/docs/mcp. Installing both the plugin and the skill gives you two copies of the skill; pick one.

## What it does to your account

- Creating a run spends your team's credits. The skill estimates the price from the model list, agrees a ceiling with you and passes it as `maxCredits`; above the ceiling nothing is charged and the agent asks again with the exact price.
- Results land in your team's library at terrads.com, next to the runs made in the app.
- A connection acts for the whole team you picked. Anything the agent creates, cancels or approves is visible to your team, and disconnecting it in **Settings → Integrations** (or revoking the key) stops it at once.

## How it signs in

The plugin connects to `https://mcp.terrads.com/mcp` and signs in with your Terrads account in the browser, the same sign-in claude.ai and ChatGPT use. Claude Code keeps the sign-in in the macOS keychain, or in its own credentials store where there is no supported keychain, and renews it on its own. This repository contains no key and no place to put one.

## Updating

In Claude Code: `/plugin marketplace update terrads`, then `/plugin update terrads@terrads`. Skill-only installs: `npx skills update`.

Updating from 1.2 to 1.3 changes how the plugin connects: it no longer uses the API key you entered. After updating, run `/mcp`, pick `terrads` and sign in once in the browser.

## Privacy and terms

https://terrads.com/privacy · https://terrads.com/terms

## License

MIT — see `LICENSE`.
