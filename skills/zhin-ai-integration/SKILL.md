---
name: zhin-ai-integration
description: Guides integration of AI/LLM capabilities in Zhin plugins using @zhin.js/ai. Covers multi-model providers, Agent tool calling, session management, streaming responses, unified tool services, and rich media output. Use when adding AI chat, agents, or tool-calling features to a Zhin bot.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin AI Integration Guide

Use this skill to add AI/LLM capabilities to Zhin bots via the `@zhin.js/ai` plugin.

## Installation and Configuration

```bash
pnpm add @zhin.js/ai
```

Enable in `zhin.config.yml`:

```yaml
plugins:
  - ai

ai:
  enabled: true
  defaultProvider: openai
  providers:
    openai:
      apiKey: sk-xxx
    anthropic:
      apiKey: sk-ant-xxx
    deepseek:
      apiKey: sk-xxx
    ollama:
      baseUrl: http://localhost:11434
  sessions:
    maxHistory: 200
    expireMs: 604800000
    useDatabase: true
  context:
    enabled: true
    maxRecentMessages: 100
    summaryThreshold: 50
    maxContextTokens: 4000
```

## Simple Question/Answer

```ts
import { usePlugin } from 'zhin.js'

const { useContext } = usePlugin()

useContext('ai', async (ai) => {
  const answer = await ai.ask('What is TypeScript?')
  console.log(answer)

  // Specify model
  const answer2 = await ai.ask('Write a poem', {
    provider: 'anthropic',
    model: 'claude-opus-4-20250514',
    temperature: 0.9,
  })
})
```

## Session Chat (Multi-turn)

```ts
useContext('ai', async (ai) => {
  const sessionId = 'user-123'
  await ai.chatWithSession(sessionId, 'My name is Alice')
  const response = await ai.chatWithSession(sessionId, 'What is my name?')
  // response: "Your name is Alice"
})
```

## Streaming Responses

```ts
useContext('ai', async (ai) => {
  const stream = await ai.chatWithSession('user-1', 'Tell a story', { stream: true })
  for await (const chunk of stream) {
    process.stdout.write(chunk)
  }
})
```

## Agent with Tool Calling

Create an agent that can call tools to accomplish tasks:

```ts
useContext('ai', async (ai) => {
  // Use built-in tools
  const result = await ai.runAgent('Calculate sin(45°)')
  console.log(result.content)
  console.log('Tools used:', result.toolCalls)

  // Custom tools
  const agent = ai.createAgent({
    model: 'gpt-4o',
    tools: [{
      name: 'search_database',
      description: 'Search the database',
      parameters: {
        type: 'object',
        properties: {
          query: { type: 'string', description: 'Search keyword' }
        },
        required: ['query']
      },
      async execute(args) {
        return { results: ['result1', 'result2'] }
      }
    }]
  })

  const result2 = await agent.run('Search for user Alice')
})
```

### Streaming Agent Events

```ts
useContext('ai', async (ai) => {
  const agent = ai.createAgent({ model: 'gpt-4o' })

  for await (const event of agent.runStream('Get the current time and calculate days until New Year')) {
    switch (event.type) {
      case 'content':
        process.stdout.write(event.data)
        break
      case 'tool_call':
        console.log(`\nCalling tool: ${event.data.name}`)
        break
      case 'tool_result':
        console.log(`Tool result: ${event.data.result}`)
        break
      case 'done':
        console.log('\nDone:', event.data.usage)
        break
    }
  }
})
```

## Unified Tool Service (ZhinTool)

Register tools that work with both AI and commands:

```ts
import { usePlugin, ZhinTool } from 'zhin.js'

const plugin = usePlugin()

const weatherTool = new ZhinTool('weather')
  .desc('Query weather info')
  .param('city', { type: 'string', description: 'City name' }, true)
  .param('days', { type: 'number', description: 'Forecast days' })
  .platform('qq', 'telegram')   // Restrict to platforms
  .scope('group', 'private')    // Restrict to scene types
  .permission('user')            // Permission level
  .execute(async (args, ctx) => {
    return `Weather in ${args.city}: Sunny`
  })
  .action(async (message, result) => {
    return `Weather: ${result.params.city}`
  })

plugin.addTool(weatherTool)
```

### Using `defineTool` Helper

```ts
import { usePlugin, defineTool } from 'zhin.js'

const plugin = usePlugin()

const calcTool = defineTool<{ expression: string }>({
  name: 'calculator',
  description: 'Calculate a math expression',
  parameters: {
    type: 'object',
    properties: {
      expression: { type: 'string', description: 'Math expression' },
    },
    required: ['expression'],
  },
  command: {
    pattern: 'calc <expression:rest>',
    alias: ['calculate'],
    usage: ['Calculate math expressions'],
  },
  execute: async (args) => {
    // Use a safe math parser (e.g. mathjs) instead of eval
    const math = await import('mathjs')
    const result = math.evaluate(args.expression)
    return `Result: ${result}`
  },
})

plugin.addTool(calcTool)
```

## AI Trigger Middleware

Configure how users trigger AI responses:

```yaml
ai:
  trigger:
    enabled: true
    prefixes: ['#', 'AI:']
    respondToAt: true        # @bot triggers AI
    respondToPrivate: true   # Private messages trigger AI
    ignorePrefixes: ['/', '!']
    timeout: 60000
```

## Rich Media Output

AI can output rich media using XML-like tags in responses:

```
<image url="https://example.com/cat.jpg"/>
<video url="https://example.com/video.mp4"/>
<audio url="https://example.com/song.mp3"/>
<at user_id="123456"/>
<face id="178"/>
```

These tags are automatically parsed into platform-specific message elements.

## Tool Permission Levels

Tools support permission restrictions (from low to high):

- `user` — Regular user
- `group_admin` — Group admin
- `group_owner` — Group owner
- `bot_admin` — Bot admin
- `owner` — Zhin owner

## Type Extension

```ts
declare module 'zhin.js' {
  namespace Plugin {
    interface Contexts {
      ai: import('@zhin.js/ai').AIService
    }
  }
}
```

## Built-in AI Tools

| Tool | Description |
|------|-------------|
| `calculator` | Math calculation (trig, log, etc.) |
| `get_time` | Get current time |
| `get_weather` | Get weather (mock) |
| `web_search` | Web search (requires config) |
| `http_request` | HTTP requests |
| `run_code` | Execute JavaScript code |

## Checklist

- Install `@zhin.js/ai` and add to plugins list.
- Configure at least one provider (apiKey).
- Use `useContext('ai', ...)` to access the AI service.
- Register custom tools via `plugin.addTool()` or `ai.registerTool()`.
- Use `ZhinTool` for tools that work with both AI and commands.
- Handle streaming events for real-time responses.
