# Terrads for your coding agent

Make ad creatives for your brand's products from your coding agent, such as Claude Code or Cursor, or any MCP client that can send a header. This repository is a Claude Code plugin and an Agent Skill for the Terrads MCP server at `https://terrads.com/api/mcp`.

Included with the Terrads Max plan. You need a Terrads account on Max and an API key from **Settings → API keys** at terrads.com (shown once, when created).

## Install

**Claude Code plugin** (recommended — the key is typed into a prompt, never into a command). In a session:

```text
/plugin marketplace add terrads-hq/agent-mode
/plugin install terrads@terrads
```

or from a terminal:

```text
claude plugin marketplace add terrads-hq/agent-mode
claude plugin install terrads@terrads
```

Enable the plugin when asked and enter your API key. The plugin connects the `terrads` MCP server and installs the `terrads` skill.

**Skill only**, for agents that manage their own MCP connections:

```text
npx skills add terrads-hq/agent-mode
```

Then connect the MCP server in your client following https://terrads.com/docs/mcp. Installing both the plugin and the skill gives you two copies of the skill; pick one.

## What it does to your account

- Creating a run spends your team's credits. The skill estimates the price from the model list, agrees a ceiling with you and passes it as `maxCredits`; above the ceiling nothing is charged and the agent asks again with the exact price.
- Results land in your team's library at terrads.com, next to the runs made in the app.
- An API key acts for the whole team. Anything the agent creates, cancels or approves is visible to your team, and revoking the key in Settings stops it at once.

## Where the key is stored

Claude Code asks for the key when the plugin is enabled and keeps it in the macOS keychain, or in its own credentials store where there is no supported keychain. It is sent only as the `Authorization` header to `https://terrads.com/api/mcp`. This repository contains no key and no place to put one.

## Updating

In Claude Code: `/plugin marketplace update terrads`, then `/plugin update terrads@terrads`. Skill-only installs: `npx skills update`.

## Privacy and terms

https://terrads.com/privacy · https://terrads.com/terms

## License

MIT — see `LICENSE`.
