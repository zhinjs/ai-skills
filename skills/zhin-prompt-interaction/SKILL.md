---
name: zhin-prompt-interaction
description: Guides interactive user input collection using the Zhin Prompt class. Covers text, number, confirm, list, and pick prompts with timeout handling and Schema-based input. Use when a plugin needs to collect input from users in a conversational flow.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin Prompt Interaction Guide

Use this skill to collect interactive input from users during conversations in Zhin plugins. The `Prompt` class supports text, number, confirmation, list, option selection, and Schema-driven input.

## Creating a Prompt

```ts
import { usePlugin, Prompt, MessageCommand } from 'zhin.js'

const plugin = usePlugin()

plugin.addCommand(
  new MessageCommand('setup')
    .desc('Interactive setup wizard')
    .action(async (message) => {
      const prompt = new Prompt(plugin, message)

      const name = await prompt.text('What is your name?')
      const age = await prompt.number('How old are you?')
      const confirm = await prompt.confirm('Is this correct?')

      if (confirm) {
        return `Welcome, ${name} (age ${age})!`
      }
      return 'Setup cancelled.'
    })
)
```

## Prompt Types

### Text Input

```ts
const name = await prompt.text('Enter your name')

// With timeout (ms) and default value
const name = await prompt.text('Enter your name', 30000, 'Anonymous', 'Timed out')
```

### Number Input

```ts
const count = await prompt.number('How many items?')

// With timeout and default
const count = await prompt.number('How many items?', 30000, 1, 'Timed out')
```

### Confirmation

```ts
// User must type "yes" to confirm
const ok = await prompt.confirm('Are you sure?')

// Custom confirmation word
const ok = await prompt.confirm('Continue?', 'ok')

// With timeout and default
const ok = await prompt.confirm('Continue?', 'yes', 30000, false, 'Timed out')
```

### List Input

Collect multiple values separated by a delimiter:

```ts
// Text list (comma-separated by default)
const tags = await prompt.list('Enter tags')
// User types: "tag1,tag2,tag3"  → ['tag1', 'tag2', 'tag3']

// Number list with custom separator
const scores = await prompt.list('Enter scores', {
  type: 'number',
  separator: ' ',
  defaultValue: [0],
  timeout: 60000,
})
```

### Option Selection (Pick)

```ts
// Single select
const color = await prompt.pick('Choose a color', {
  type: 'text',
  options: [
    { label: 'Red', value: 'red' },
    { label: 'Green', value: 'green' },
    { label: 'Blue', value: 'blue' },
  ],
})
// Displays:
// Choose a color
// 1.Red
// 2.Green
// 3.Blue
// User types: "2" → 'green'

// Multi-select
const colors = await prompt.pick('Choose colors', {
  type: 'text',
  multiple: true,
  separator: ',',
  options: [
    { label: 'Red', value: 'red' },
    { label: 'Green', value: 'green' },
    { label: 'Blue', value: 'blue' },
  ],
})
// User types: "1,3" → ['red', 'blue']
```

## Schema-based Input

Collect structured input driven by Schema definitions:

```ts
import { Schema } from '@zhin.js/schema'

// Single schema value
const value = await prompt.getValueWithSchema(
  Schema.string().description('Enter your email')
)

// Multiple schema values at once
const result = await prompt.getValueWithSchemas({
  name: Schema.string().description('Your name'),
  age: Schema.number().description('Your age'),
  admin: Schema.boolean().description('Are you admin?'),
})
// result = { name: 'Alice', age: 25, admin: true }
```

### Supported Schema Types

| Type | Prompt Used |
|------|-------------|
| `string` | `text()` |
| `number` | `number()` |
| `boolean` | `confirm()` |
| `object` | Recursively prompts each field |
| `list` | `list()` |
| `date` | Text input parsed as Date |
| `regexp` | Text input parsed as RegExp |
| `const` | Returns the default value |
| Schema with `options` | `pick()` |

## Timeout and Error Handling

All prompts accept a timeout parameter (default: 3 minutes). On timeout, the default value is used if provided, otherwise an error is thrown:

```ts
try {
  const name = await prompt.text('Enter name', 10000) // 10 seconds
} catch (error) {
  // error.message = 'Input timeout'
  await message.$reply('You took too long!')
}
```

## Using One-time Middleware

For lower-level control, use `prompt.middleware()` directly:

```ts
prompt.middleware(
  (input) => {
    if (input instanceof Error) {
      // Timeout occurred
      return
    }
    // input is the raw message string
    console.log('User said:', input)
  },
  60000,           // timeout in ms
  'Input timed out' // timeout message
)
```

## Checklist

- Create `Prompt` with `new Prompt(plugin, message)` inside a command action.
- Choose the appropriate prompt type for each input.
- Always handle timeouts with default values or try/catch.
- Use Schema-based prompts for structured configuration collection.
- The prompt automatically matches user sessions by adapter-bot-channel-sender.
