---
name: zhin-tool-service
description: Guides creation and management of Zhin tools using ZhinTool, defineTool, and ToolService. Covers the unified tool system that bridges AI agent tool-calling and message commands, including Tool↔Command conversion, permission levels, platform/scope filtering, and tool collection. Use when registering tools, converting between tools and commands, or integrating tools with AI agents.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin Tool Service Guide

Use this skill to create, register, and manage tools in Zhin plugins. The Tool Service provides a unified abstraction that bridges AI agent tool-calling and traditional message commands.

## Core Concepts

A **Tool** in Zhin can serve three purposes simultaneously:

1. **AI Agent tool** — callable by LLM agents via function calling
2. **Message Command** — executable by users as a chat command
3. **MCP tool** — exposable via the Model Context Protocol

## ZhinTool Class (Chain API)

`ZhinTool` provides a builder-style API similar to `MessageCommand`:

```ts
import { usePlugin, ZhinTool } from 'zhin.js'

const plugin = usePlugin()

const weatherTool = new ZhinTool('weather')
  .desc('Query weather information')
  .param('city', { type: 'string', description: 'City name' }, true)
  .param('days', { type: 'number', description: 'Forecast days' })
  .platform('qq', 'telegram')
  .scope('group', 'private')
  .permission('user')
  .usage('Query the weather for a city')
  .examples('weather Beijing', 'weather Tokyo 3')
  .alias('天气')
  .execute(async (args, ctx) => {
    return `Weather in ${args.city}: Sunny`
  })
  .action(async (message, result) => {
    return `Weather: ${result.params.city}`
  })

plugin.addTool(weatherTool)
```

### ZhinTool Methods

| Method | Description |
|--------|-------------|
| `.desc(text)` | Set tool description (for AI and help) |
| `.param(name, schema, required?)` | Add a parameter (ordered) |
| `.platform(...names)` | Restrict to platforms (e.g. `'qq'`, `'telegram'`) |
| `.scope(...types)` | Restrict to scene types (`'private'`, `'group'`, `'channel'`) |
| `.permission(level)` | Set minimum permission level |
| `.permit(...perms)` | Add legacy permission strings (compatible with MessageCommand) |
| `.tag(...tags)` | Add tags for filtering |
| `.hidden(value?)` | Hide from help listings |
| `.usage(...texts)` | Add usage descriptions |
| `.examples(...texts)` | Add usage examples |
| `.alias(...names)` | Add command aliases |
| `.pattern(pat)` | Override the auto-generated command pattern |
| `.execute(fn)` | **Required.** Set the AI tool execution function `(args, ctx) => result` |
| `.action(fn)` | Set the command callback `(message, matchResult) => result` |

### execute vs action

- **`execute`** (required) — Called when the tool is invoked by an AI agent. Receives `(args, context?)` where `args` are parsed parameters and `context` is a `ToolContext`.
- **`action`** (optional) — Called when the tool is invoked as a chat command. Receives `(message, matchResult)` just like `MessageCommand.action`. If omitted, `execute` is used as a fallback to auto-generate the command handler.

## defineTool Helper

Use `defineTool` for a type-safe object-style definition:

```ts
import { usePlugin, defineTool } from 'zhin.js'

const plugin = usePlugin()

const calcTool = defineTool<{ expression: string }>({
  name: 'calculator',
  description: 'Calculate a math expression',
  parameters: {
    type: 'object',
    properties: {
      expression: {
        type: 'string',
        description: 'Math expression',
      },
    },
    required: ['expression'],
  },
  command: {
    pattern: 'calc <expression:rest>',
    alias: ['calculate'],
    usage: ['Calculate math expressions'],
  },
  execute: async (args) => {
    // args.expression has full type inference
    const math = await import('mathjs')
    return `Result: ${math.evaluate(args.expression)}`
  },
})

plugin.addTool(calcTool)
```

The generic parameter `<{ expression: string }>` gives you type checking on `args` inside `execute`.

## Registering Tools

### addTool — register with command generation

```ts
plugin.addTool(tool)
```

Registers the tool **and** auto-generates a corresponding `MessageCommand` (unless `command: false`).

### addToolOnly — register without command

```ts
plugin.addToolOnly(tool)
```

Registers the tool for AI agent use only, without generating a command.

Both methods return a dispose function:

```ts
const dispose = plugin.addTool(weatherTool)
// Later: dispose() removes the tool and its command
```

Tools are automatically cleaned up when the plugin is disposed.

## Tool ↔ Command Conversion

The ToolService can convert in both directions:

### Tool → Command (automatic)

When you call `plugin.addTool(tool)`, a `MessageCommand` is auto-generated from the tool's parameters:

```ts
// Tool with parameters: { city: string (required), days: number (optional) }
// Generates command pattern: "weather <city:text> [days:number]"
```

Parameter ordering: required parameters come first, then optional ones.

### Command → Tool (via ToolService)

Existing commands can be wrapped as tools for AI agent use:

```ts
const toolService = plugin.inject('tool')
const tool = toolService.commandToTool(existingCommand, 'my-plugin')
```

### collectAll — gather all available tools

```ts
const toolService = plugin.inject('tool')
const allTools = toolService.collectAll(plugin)
```

This collects tools from three sources:
1. **ToolService tools** — tools registered via `addTool`/`addToolOnly`
2. **Commands** — existing `MessageCommand` instances converted to tools
3. **Adapter tools** — platform-specific tools from adapters

## Permission Levels

Tools support a hierarchical permission system (from lowest to highest):

| Level | Priority | Description |
|-------|----------|-------------|
| `user` | 0 | Regular user (default) |
| `group_admin` | 1 | Group administrator |
| `group_owner` | 2 | Group owner |
| `bot_admin` | 3 | Bot administrator |
| `owner` | 4 | Zhin instance owner |

```ts
new ZhinTool('admin-action')
  .desc('Admin-only operation')
  .permission('bot_admin')
  .execute(async (args) => { /* ... */ })
```

The ToolService automatically checks permission levels when filtering tools for a given context.

## Platform and Scope Filtering

Restrict where a tool can be used:

```ts
new ZhinTool('group-only')
  .platform('qq', 'discord')       // Only on QQ and Discord
  .scope('group')                   // Only in group chats
  .execute(async (args) => { /* ... */ })
```

The `filterByContext` method checks all restrictions:

```ts
const toolService = plugin.inject('tool')
const context = {
  platform: 'qq',
  scope: 'group',
  senderPermissionLevel: 'user',
}
const available = toolService.filterByContext(allTools, context)
```

## ToolContext

When a tool is executed, it receives a `ToolContext` with runtime information:

```ts
interface ToolContext {
  platform?: string        // Source platform (e.g. 'qq')
  botId?: string           // Bot identifier
  sceneId?: string         // Scene ID (group/channel/user)
  senderId?: string        // Sender's user ID
  scope?: ToolScope        // 'private' | 'group' | 'channel'
  message?: Message        // Original message object
  senderPermissionLevel?: ToolPermissionLevel
  isGroupAdmin?: boolean
  isGroupOwner?: boolean
  isBotAdmin?: boolean
  isOwner?: boolean
  extra?: Record<string, any>
}
```

## ToolService API

Access the ToolService via context injection:

```ts
const toolService = plugin.inject('tool')
```

| Method | Description |
|--------|-------------|
| `add(tool, pluginName, generateCommand?)` | Register a tool |
| `remove(name)` | Remove a tool by name |
| `get(name)` | Get a tool by name |
| `getAll()` | Get all registered tools |
| `getByTags(tags)` | Filter tools by tags |
| `execute(name, args, context?)` | Execute a tool by name |
| `commandToTool(command, pluginName)` | Convert a Command to a Tool |
| `collectAll(plugin)` | Collect all tools from all sources |
| `filterByContext(tools, context)` | Filter tools by platform/scope/permission |

## Parameter Schema

Tool parameters use JSON Schema format with Zhin extensions:

```ts
{
  type: 'object',
  properties: {
    city: {
      type: 'string',
      description: 'City name',
      paramType: 'text',    // Zhin extension: command param type
    },
    count: {
      type: 'number',
      description: 'Number of results',
      default: 5,
    },
    format: {
      type: 'string',
      enum: ['json', 'text'],
      description: 'Output format',
    },
  },
  required: ['city'],
}
```

The `paramType` extension controls how the parameter is parsed in command mode:
- `'text'` (default for strings)
- `'number'` (default for numbers)
- `'boolean'`
- `'rest'` (captures all remaining text)

## Serialization

### toJSON — for AI function calling

```ts
const tool = new ZhinTool('weather').desc('...').param(...)
const schema = tool.toJSON()
// Returns: { name, description, parameters, platforms?, scopes?, permissionLevel?, tags? }
// Compatible with OpenAI Function Calling format
```

### help — human-readable help text

```ts
console.log(tool.help)
// weather <city:text> [days:number]
//   Query weather information
//   Parameters:
//     city: string (required) City name
//     days: number (optional) Forecast days
```

## Disabling Command Generation

Set `command: false` to prevent auto-generating a command:

```ts
const tool = defineTool({
  name: 'internal_lookup',
  description: 'Internal data lookup',
  parameters: { type: 'object', properties: {} },
  command: false,
  execute: async () => { /* AI-only tool */ },
})
```

## Checklist

- Use `ZhinTool` (chain API) or `defineTool` (object API) to create tools.
- Always provide `execute()` — it is required for AI agent invocation.
- Use `action()` on ZhinTool for custom command behavior; omit it to auto-generate from `execute`.
- Register with `plugin.addTool()` (tool + command) or `plugin.addToolOnly()` (tool only).
- Set `permission()`, `platform()`, and `scope()` to control access.
- Use `collectAll()` and `filterByContext()` when building AI agent tool lists.
- Tools are auto-disposed when their parent plugin is disposed.
