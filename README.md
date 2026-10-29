# Talvedo plugin

The [Talvedo](https://talvedo.com) issue tracker in your agent: the Talvedo MCP server with sign-in over OAuth, a skill that teaches the agent the model and the usual tasks, and, in Claude Code, a `talvedo` subagent for bulk work.

## Claude Code

```bash
claude plugin marketplace add Talvedo/plugins
claude plugin install talvedo@talvedo
```

Inside a session the same is `/plugin marketplace add Talvedo/plugins` and `/plugin install talvedo@talvedo`. Then run `/mcp`, pick the Talvedo server and sign in: Talvedo asks which workspace to connect.

Without a GitHub SSH key, set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` before adding the marketplace.

## ChatGPT and Codex

```bash
codex plugin marketplace add Talvedo/plugins
```

Then open the Plugins Directory in the ChatGPT desktop app or in Codex, choose the Talvedo marketplace and install the plugin. Sign in when it asks.

In ChatGPT on the web, add Talvedo as a custom MCP server instead: Plugins, `+`, Add custom MCP server, `https://talvedo.com/mcp`.

## What the agent can do

Everything you can do in Talvedo, as you and with your permissions: search, file and update issues, epics, sprints and releases, set up tracks and labels, split a feature across platforms, read the parity matrix, keep checklists, run test cases. See the [user guide](https://talvedo.com/help/mcp).

This repository is generated from the Talvedo source. Report problems at https://talvedo.com.
