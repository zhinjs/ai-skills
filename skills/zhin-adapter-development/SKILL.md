---
name: zhin-adapter-development
description: Guides development of custom platform adapters in Zhin. Covers extending the Adapter abstract class, creating Bot instances, handling message events, registering tools, and lifecycle management. Use when building adapters for new chat platforms.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin Adapter Development Guide

Use this skill to create custom platform adapters that connect Zhin to chat platforms (QQ, Discord, Telegram, etc.).

## Adapter Architecture

An adapter manages multiple Bot instances for a single platform. Each Bot connects to the platform and handles message sending/receiving.

```
Adapter (platform)
├── Bot 1 (account A)
├── Bot 2 (account B)
└── Tools (platform-specific capabilities)
```

## Minimal Adapter

```ts
import { Adapter, Bot, Plugin, Message } from '@zhin.js/core'

// 1. Define Bot config type
interface MyBotConfig {
  token: string
  name: string
}

// 2. Define platform message type
interface MyMessage {
  id: string
  content: string
  channelId: string
  senderId: string
}

// 3. Create Bot class
class MyBot extends Bot<MyBotConfig, MyMessage> {
  $connected = false

  async $connect() {
    // Connect to platform API
    this.$connected = true
  }

  async $disconnect() {
    // Disconnect from platform
    this.$connected = false
  }

  async $sendMessage(options: { id: string; type: string; content: string }) {
    // Send message via platform API
    return 'message-id'
  }

  async $recallMessage(messageId: string) {
    // Recall/delete a message
  }
}

// 4. Create Adapter class
class MyAdapter extends Adapter<MyBot> {
  constructor(plugin: Plugin, config: Adapter.BotConfig<MyBot>[]) {
    super(plugin, 'my-platform' as any, config)
  }

  createBot(config: MyBotConfig): MyBot {
    return new MyBot(config)
  }
}

// 5. Register the adapter
Adapter.register('my-platform', MyAdapter as any)
```

## Using the Adapter in a Plugin

```ts
import { usePlugin, Adapter } from 'zhin.js'

const plugin = usePlugin()

// Create and start adapter
const adapter = new MyAdapter(plugin, [
  { token: 'bot-token-1', name: 'bot1' },
])

plugin.onMounted(async () => {
  await adapter.start()
})

plugin.onDispose(async () => {
  await adapter.stop()
})
```

## Message Handling

When a message arrives from the platform, emit it on the adapter:

```ts
class MyAdapter extends Adapter<MyBot> {
  private handleIncomingMessage(rawMsg: MyMessage) {
    const message = new Message({
      $adapter: 'my-platform',
      $bot: rawMsg.botId,
      $channel: { id: rawMsg.channelId, type: 'group' },
      $sender: { id: rawMsg.senderId, name: rawMsg.senderName },
      $content: rawMsg.content,
      $raw: rawMsg.content,
      $reply: async (content) => {
        // Reply implementation
        return 'reply-id'
      },
    })
    // This triggers the middleware pipeline
    this.emit('message.receive', message)
  }
}
```

### Message Lifecycle

1. Platform receives raw message.
2. Adapter converts to `Message` object.
3. `message.receive` event triggers the root plugin middleware pipeline.
4. Middleware chain processes the message (logging, command matching, etc.).

## Registering Adapter Tools

Adapters can provide tools for the AI service:

```ts
class MyAdapter extends Adapter<MyBot> {
  constructor(plugin: Plugin, config: any[]) {
    super(plugin, 'my-platform' as any, config)
    this.registerDefaultTools() // Adds send_message and list_bots

    // Add custom platform tools
    this.addTool({
      name: 'my_platform_get_user',
      description: 'Get user info from my-platform',
      parameters: {
        type: 'object',
        properties: {
          userId: { type: 'string', description: 'User ID' },
        },
        required: ['userId'],
      },
      execute: async (args) => {
        return await this.getUserInfo(args.userId)
      },
    })
  }
}
```

### Default Tools (via `registerDefaultTools`)

- `{platform}_send_message` — Send message to a target
- `{platform}_list_bots` — List connected bots

## Before-Send Hook

Adapters support `before.sendMessage` hooks for message transformation:

```ts
plugin.root.on('before.sendMessage', async (options) => {
  // Modify options.content before sending
  return options
})
```

## Adapter Lifecycle Events

| Event | Description |
|-------|-------------|
| `message.receive` | A message was received |
| `message.private.receive` | A private message received |
| `message.group.receive` | A group message received |
| `call.recallMessage` | Request to recall a message |

## Configuration in `zhin.config.yml`

```yaml
bots:
  - name: my-platform
    context: my-platform
    token: bot-token-here

plugins:
  - my-adapter-plugin
```

## Checklist

- Extend `Adapter` abstract class with your platform name.
- Create a `Bot` subclass implementing `$connect`, `$disconnect`, `$sendMessage`, `$recallMessage`.
- Use `Adapter.register(name, factory)` to register the adapter factory.
- Emit `message.receive` with a `Message` object when messages arrive.
- Call `registerDefaultTools()` in the constructor for AI tool support.
- Implement `start()` and `stop()` lifecycle in the plugin via `onMounted`/`onDispose`.
