---
name: zhin-mcp-server
description: Guides setup and usage of the Zhin MCP (Model Context Protocol) server plugin. Covers configuration, available tools, resources, and prompts for AI assistant integration. Use when integrating Zhin with AI coding assistants like Claude or Cursor via MCP.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin MCP Server Guide

Use this skill to set up the `@zhin.js/mcp` plugin, enabling AI assistants (Claude, Cursor, etc.) to understand and generate Zhin plugins via the Model Context Protocol.

## Installation

```bash
pnpm add @zhin.js/mcp
```

## Configuration

MCP requires the HTTP plugin. Enable both in `zhin.config.yml`:

```yaml
plugins:
  - http
  - mcp

http:
  port: 8086

mcp:
  enabled: true
  path: /mcp
```

The MCP endpoint is available at `http://localhost:8086/mcp`.

## Connecting AI Assistants

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "zhin": {
      "command": "curl",
      "args": ["-N", "http://localhost:8086/mcp"]
    }
  }
}
```

### Cursor/VSCode

Install an MCP extension and configure the server URL in settings.

## Available Tools

### create_plugin

Create a new Zhin plugin file.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Plugin name |
| `description` | Yes | Plugin description |
| `features` | No | Feature list: `command`, `middleware`, `component`, `context`, `database` |
| `directory` | No | Save directory (default: `src/plugins`) |

### create_command

Generate command code snippets.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `pattern` | Yes | Command pattern, e.g. `hello <name:text>` |
| `description` | Yes | Command description |
| `hasPermission` | No | Whether to include permission checks |

### create_component

Generate message component code.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Component name |
| `props` | Yes | Property definitions |
| `usesJsx` | No | Whether to use JSX syntax |

### create_adapter

Generate platform adapter code.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Adapter name |
| `description` | Yes | Adapter description |
| `hasWebhook` | No | Whether webhook support is needed |

### create_model

Generate database model definition.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `name` | Yes | Model name |
| `fields` | Yes | Field definitions |

### query_plugin

Query details of an existing plugin.

| Parameter | Required | Description |
|-----------|----------|-------------|
| `pluginName` | Yes | Plugin name to query |

### list_plugins

List all loaded plugins (no parameters).

## Available Resources

MCP exposes documentation as resources:

| URI | Description |
|-----|-------------|
| `zhin://docs/architecture` | Zhin architecture design |
| `zhin://docs/plugin-development` | Plugin development guide |
| `zhin://docs/best-practices` | Development best practices |
| `zhin://docs/command-system` | Command system docs |
| `zhin://docs/component-system` | Component system docs |
| `zhin://docs/context-system` | Context system docs |
| `zhin://examples/basic-plugin` | Basic plugin example |
| `zhin://examples/command-plugin` | Command plugin example |
| `zhin://examples/adapter` | Adapter example |

## Available Prompts

### create-plugin-workflow

Guides the full plugin creation workflow.

Parameter: `feature_type` — `command`, `middleware`, `component`, or `adapter`.

### debug-plugin

Step-by-step plugin debugging guidance.

Parameter: `error_message` (optional) — the error to diagnose.

### best-practices

Returns Zhin development best practices.

## Usage Scenarios

### Create a Plugin via AI

```
User: Create a plugin named "welcome" that replies when users say hello

AI: [Uses create_plugin tool] Created plugin with command feature...
```

### Debug a Plugin via AI

```
User: My plugin reports "Context not found"

AI: [Uses query_plugin to inspect] This error usually means the context
    dependency hasn't been registered yet...
```

### Generate an Adapter via AI

```
User: Create a WhatsApp adapter with webhook support

AI: [Uses create_adapter tool] Generated WhatsApp adapter scaffold...
```

## Checklist

- Install `@zhin.js/mcp` and `@zhin.js/http`.
- Add both `http` and `mcp` to the plugins list.
- Configure `mcp.path` (default `/mcp`).
- Start the Zhin app, then connect your AI assistant.
- Use MCP tools to scaffold plugins, commands, components, and adapters.
- Access `zhin://docs/*` resources for framework documentation.
