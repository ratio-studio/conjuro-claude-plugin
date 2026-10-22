# Conjuro for Claude

A Claude Code plugin that connects Claude to [Conjuro](https://conjuro.app) through Conjuro's MCP server. Once installed and signed in, Claude can search your tasks and projects, catch up on team conversations, and work your sales pipeline. It can only make changes you have allowed.

This repository is also a plugin marketplace named `conjuro`, so you can install the plugin with two commands.

## What's included

- **MCP server** `conjuro`, a remote HTTP server at `https://live.conjuro.app/mcp`. It appears in `/mcp` as `plugin:conjuro:conjuro`. Sign-in uses OAuth; no API key is stored in this plugin.
- **Skills** that teach Claude how to use Conjuro safely:
  - `conjuro-tasks`: find, create, and complete tasks.
  - `conjuro-chat`: read conversations and reply on your behalf, always after you approve the exact text.
  - `conjuro-pipeline`: look up customers and create, update, move, or archive leads, confirming each change.

## Install

> Replace `OWNER` below with the GitHub account that hosts this repository.

### Terminal (Claude Code CLI)

```bash
claude plugin marketplace add OWNER/conjuro-claude-plugin
```

```bash
claude plugin install conjuro@conjuro
```

Or both in one step (Claude Code v2.1.292 or later):

```bash
claude plugin install conjuro --marketplace OWNER/conjuro-claude-plugin
```

Then start `claude`, run `/mcp`, select `plugin:conjuro:conjuro`, and choose **Authenticate**. Your browser opens Conjuro: sign in, check the workspace and permissions on the consent screen, and approve. You can also sign in from the shell with `claude mcp login plugin:conjuro:conjuro`.

### Claude desktop app (Code tab)

The desktop app's Code tab uses the same Claude Code plugins as the terminal, so a plugin installed with the commands above is already available there. To install from inside a session instead, type:

```text
/plugin install conjuro --marketplace OWNER/conjuro-claude-plugin
```

Then type `/mcp`, choose `plugin:conjuro:conjuro`, and authenticate in the browser window that opens.

### Try it

Ask Claude things like:

- "What Conjuro tasks are due this week?"
- "Catch me up on unread Conjuro conversations."
- "Move Acme Ltd to the Proposal stage in Conjuro."

## Permissions

Each Claude connection to Conjuro has its own permissions, and Conjuro checks them on every request. Claude acts as you, in the workspace you chose when you connected, and can never do more than you could in the Conjuro app.

| Permission | Lets Claude | Default |
|---|---|---|
| See your profile and current workspace | Read your name and workspace | On |
| Read tasks | Search and read tasks | On |
| Read projects and project updates | List projects, health, and updates | On |
| Read customers | Search customers; read pipeline stages, categories, and sources | On |
| Create and complete tasks | Create tasks; mark them complete | **Off** |
| Create, update, move, and archive leads and customers | Change the pipeline | **Off** |
| Read your chat conversations | List and read conversations | **Off** |
| Send chat messages as you and mark conversations read | Reply as you (shown "via" Claude) | **Off** |
| Read Project Meister status and requests | Read assistant context | **Off** |
| Send requests to Project Meister | Create Meister proposals; you still approve them in Conjuro | **Off** |

To change them, open Conjuro, go to **Account → Agent connections**, find the Claude connection, and use **Manage permissions**. Changes apply to the next request.

Even with a write permission on, the plugin's skills tell Claude to show you each change and wait for your yes before making it. Chat messages and customer records are treated as data, never as instructions to Claude. Every change an agent makes is recorded in Conjuro's audit log, and chat replies are labelled with the client that sent them.

## Revoke access

- **In Conjuro:** **Account → Agent connections**, then revoke the Claude connection. This immediately disables its tokens; Claude has to sign in again to reconnect.
- **In Claude Code:** run `/mcp`, select `plugin:conjuro:conjuro`, and choose **Clear authentication** to forget the local sign-in.
- **Remove the plugin:**

  ```bash
  claude plugin uninstall conjuro@conjuro
  ```

  To also remove the marketplace:

  ```bash
  claude plugin marketplace remove conjuro
  ```

Uninstalling the plugin doesn't revoke the connection on Conjuro's side; revoke it in Conjuro too if you're done with it.

## Data and network use

The plugin contains no code that runs on your machine. Its only network endpoint is Conjuro's MCP server at `https://live.conjuro.app/mcp` (plus Conjuro's OAuth endpoints on the same host during sign-in). Tool results come from your Conjuro workspace and are shown to Claude to answer your requests.

Conjuro's [Privacy Policy](https://conjuro.app/privacy) and [Terms of Service](https://conjuro.app/terms) apply to your use of Conjuro through this plugin.

## Development

Validate the plugin and marketplace:

```bash
claude plugin validate --strict .
```

Install from a local checkout:

```bash
claude plugin marketplace add ./
```

```bash
claude plugin install conjuro@conjuro
```

## License

Copyright 2026 Local Ventures & Investments, LLC (d/b/a ratio.dev). Licensed under the [Apache License, Version 2.0](LICENSE). The Conjuro name and logo aren't covered by the license.
