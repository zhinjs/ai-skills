---
name: zhin-error-handling
description: Guides error handling in Zhin using the built-in error hierarchy, ErrorManager, RetryManager, and CircuitBreaker. Use when implementing resilient error handling, retry logic, or circuit breaker patterns in Zhin plugins.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin Error Handling Guide

Use this skill to handle errors in Zhin plugins using the built-in error classes and resilience patterns.

## Error Class Hierarchy

All custom errors extend `ZhinError`:

```
ZhinError (base)
├── ConfigError         — Configuration issues
├── PluginError         — Plugin loading/execution failures
├── AdapterError        — Adapter/bot connection issues
├── ConnectionError     — Network/connection failures (retryable flag)
├── MessageError        — Message processing errors
├── ContextError        — Context injection failures
├── ValidationError     — Input validation errors
├── PermissionError     — Permission/authorization errors
└── TimeoutError        — Operation timeout errors
```

## Using Error Classes

```ts
import {
  ZhinError, ConfigError, PluginError, AdapterError,
  ConnectionError, MessageError, ContextError,
  ValidationError, PermissionError, TimeoutError,
} from '@zhin.js/core'

// Configuration error
throw new ConfigError('Missing API key', { field: 'apiKey' })

// Plugin error with plugin name
throw new PluginError('Failed to initialize', 'my-plugin', { reason: 'missing dep' })

// Adapter error with adapter and bot name
throw new AdapterError('Connection refused', 'discord', 'bot-1')

// Connection error (retryable)
throw new ConnectionError('Server unreachable', true)

// Permission error
throw new PermissionError('Admin required', 'user-123', 'bot_admin')

// Timeout error
throw new TimeoutError('Operation timed out', 30000)
```

### Error Serialization

```ts
try {
  // ...
} catch (err) {
  if (err instanceof ZhinError) {
    console.log(err.toJSON())        // Full JSON representation
    console.log(err.toUserString())  // User-friendly: "[CODE] message"
    console.log(err.code)            // Error code string
    console.log(err.timestamp)       // When the error occurred
    console.log(err.context)         // Additional context data
  }
}
```

## ErrorManager

Central error handler registry for organized error handling:

```ts
import { errorManager, ErrorHandler } from '@zhin.js/core'

// Register handler for specific error types
errorManager.register('ConnectionError', async (error, context) => {
  console.log('Connection issue:', error.message)
  // Notify admin, log to database, etc.
})

// Register global handler (called for all errors)
errorManager.registerGlobal(async (error, context) => {
  console.log('Error occurred:', error.message)
})

// Handle an error
try {
  await riskyOperation()
} catch (error) {
  await errorManager.handle(error as Error, { source: 'my-plugin' })
}

// Clean up handlers
errorManager.unregister('ConnectionError', myHandler)
errorManager.clear()
```

## RetryManager

Automatic retry with exponential backoff:

```ts
import { RetryManager } from '@zhin.js/core'

const result = await RetryManager.retry(
  async () => {
    // Operation that might fail
    return await fetchExternalAPI()
  },
  {
    maxRetries: 3,              // Maximum retry attempts
    delay: 1000,                // Base delay in ms
    exponentialBackoff: true,   // Double delay each retry
    retryCondition: (error) => {
      // Only retry ConnectionErrors
      return error instanceof ConnectionError && error.retryable
    },
  }
)
```

### Retry Timing

With `exponentialBackoff: true` and `delay: 1000`:
- Attempt 1 → immediate
- Attempt 2 → wait 1000ms
- Attempt 3 → wait 2000ms
- Attempt 4 → wait 4000ms

## CircuitBreaker

Prevent cascading failures with the circuit breaker pattern:

```ts
import { CircuitBreaker } from '@zhin.js/core'

const breaker = new CircuitBreaker(
  5,      // failureThreshold — open after 5 failures
  60000,  // timeoutMs — stay open for 60s
  10000   // monitoringPeriodMs
)

try {
  const result = await breaker.execute(async () => {
    return await callExternalService()
  })
} catch (error) {
  if (error.message === 'Circuit breaker is OPEN') {
    // Service is down, use fallback
    return fallbackValue
  }
  throw error
}

// Check state
console.log(breaker.getState()) // 'CLOSED' | 'OPEN' | 'HALF_OPEN'

// Manual reset
breaker.reset()
```

### Circuit Breaker States

| State | Description |
|-------|-------------|
| `CLOSED` | Normal operation, requests pass through |
| `OPEN` | Too many failures, requests are rejected immediately |
| `HALF_OPEN` | Timeout expired, one test request is allowed |

## Pattern: Plugin Error Handling

```ts
import { usePlugin, PluginError, RetryManager } from 'zhin.js'

const plugin = usePlugin()

plugin.onMounted(async () => {
  try {
    const data = await RetryManager.retry(
      () => loadExternalResource(),
      { maxRetries: 3, delay: 2000 }
    )
    plugin.logger.info('Resource loaded successfully')
  } catch (error) {
    plugin.logger.error(`Failed to load resource: ${error.message}`)
    throw new PluginError('Initialization failed', plugin.name, {
      originalError: error.message,
    })
  }
})
```

## Checklist

- Use specific error classes instead of generic `Error`.
- Include context information when creating errors.
- Register error handlers via `errorManager` for centralized logging.
- Use `RetryManager` for transient failures (network, API).
- Use `CircuitBreaker` to protect against cascading failures.
- Check `ConnectionError.retryable` before retrying.
- Clean up error handlers in `onDispose`.
