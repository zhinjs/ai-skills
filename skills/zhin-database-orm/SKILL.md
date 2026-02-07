---
name: zhin-database-orm
description: Guides database usage in Zhin including model definitions, CRUD queries, transactions, migrations, and lifecycle hooks. Covers the built-in ORM based on @zhin.js/database with SQLite support. Use when working with data persistence in Zhin plugins.
license: MIT
metadata:
  author: zhinjs
  version: "1.0"
  framework: zhin
---

# Zhin Database ORM Guide

Use this skill to define models, query data, and manage database operations in Zhin plugins.

## Configuration

```yaml
database:
  dialect: sqlite
  filename: ./data/bot.db
```

## Defining Models

Use `plugin.defineModel()` inside a `useContext('database', ...)` callback:

```ts
import { usePlugin } from 'zhin.js'

const { useContext } = usePlugin()

useContext('database', async (db) => {
  // Define a model
  plugin.defineModel('users', {
    id: { type: 'integer', primaryKey: true, autoIncrement: true },
    name: { type: 'text', notNull: true },
    email: { type: 'text', unique: true },
    score: { type: 'integer', default: 0 },
    created_at: { type: 'integer', default: () => Date.now() },
  })
})
```

### Supported Column Types

| Type | Description |
|------|-------------|
| `text` | String/text data |
| `integer` | Integer numbers |
| `real` | Floating-point numbers |
| `blob` | Binary data |
| `json` | JSON-serialized objects |
| `boolean` | Boolean values |

### Column Options

| Option | Description |
|--------|-------------|
| `primaryKey` | Mark as primary key |
| `autoIncrement` | Auto-increment (integer only) |
| `notNull` | Disallow NULL values |
| `unique` | Enforce uniqueness |
| `default` | Default value (or function) |
| `references` | Foreign key reference |

## CRUD Operations

### Query (Read)

```ts
useContext('database', async (db) => {
  const User = db.model('users')

  // Find by primary key
  const user = await User.findByPk(1)

  // Find one by condition
  const admin = await User.findOne({ where: { name: 'admin' } })

  // Find all with conditions
  const activeUsers = await User.findAll({
    where: { score: { $gt: 100 } },
    order: [['score', 'DESC']],
    limit: 10,
  })

  // Count records
  const total = await User.count({ where: { score: { $gt: 0 } } })
})
```

### Query Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `$eq` | Equal | `{ age: { $eq: 18 } }` |
| `$ne` | Not equal | `{ status: { $ne: 'banned' } }` |
| `$gt` | Greater than | `{ score: { $gt: 100 } }` |
| `$gte` | Greater or equal | `{ score: { $gte: 100 } }` |
| `$lt` | Less than | `{ age: { $lt: 30 } }` |
| `$lte` | Less or equal | `{ age: { $lte: 30 } }` |
| `$in` | In array | `{ status: { $in: ['active', 'vip'] } }` |
| `$like` | LIKE pattern | `{ name: { $like: '%alice%' } }` |

### Create

```ts
useContext('database', async (db) => {
  const User = db.model('users')

  // Create single record
  const newUser = await User.create({ name: 'Alice', email: 'alice@example.com' })

  // Bulk create
  await User.bulkCreate([
    { name: 'Bob', email: 'bob@example.com' },
    { name: 'Charlie', email: 'charlie@example.com' },
  ])
})
```

### Update

```ts
useContext('database', async (db) => {
  const User = db.model('users')

  // Update by condition
  await User.update({ score: 200 }, { where: { name: 'Alice' } })

  // Increment
  await User.update(
    { score: db.literal('score + 10') },
    { where: { id: 1 } }
  )
})
```

### Delete

```ts
useContext('database', async (db) => {
  const User = db.model('users')

  // Delete by condition
  await User.destroy({ where: { score: { $lt: 0 } } })

  // Delete by primary key
  await User.destroy({ where: { id: 5 } })
})
```

## Transactions

Wrap multiple operations in a transaction for atomicity:

```ts
useContext('database', async (db) => {
  await db.transaction(async (t) => {
    const User = db.model('users')
    const Log = db.model('SystemLog')

    await User.update(
      { score: db.literal('score - 100') },
      { where: { id: 1 }, transaction: t }
    )

    await Log.create({
      level: 'info',
      message: 'Score deducted',
      timestamp: Date.now(),
    }, { transaction: t })

    // If any operation fails, all changes are rolled back
  })
})
```

## Migrations

Migrations manage schema changes over time:

```ts
import { Migration } from '@zhin.js/database'

const migration = new Migration(db)

// Add a column
await migration.addColumn('users', 'avatar', { type: 'text', default: '' })

// Remove a column
await migration.removeColumn('users', 'old_field')

// Rename a column
await migration.renameColumn('users', 'name', 'username')

// Create index
await migration.addIndex('users', ['email'], { unique: true })
```

## Built-in Models

Zhin provides two built-in models:

### SystemLog

```ts
{
  id: { type: 'integer', primaryKey: true, autoIncrement: true },
  level: { type: 'text' },       // 'info' | 'warn' | 'error' | 'debug'
  message: { type: 'text' },
  source: { type: 'text' },
  timestamp: { type: 'integer' },
  extra: { type: 'json' },
}
```

### User

```ts
{
  id: { type: 'integer', primaryKey: true, autoIncrement: true },
  name: { type: 'text' },
  authority: { type: 'integer', default: 1 },
}
```

## Database Context Pattern

The database service is available as a context:

```ts
// Database is ready when callback fires
useContext('database', async (db) => {
  // db is fully initialized
  // Define models, run queries, etc.
})
```

The callback runs only when the database is connected and ready. If the database reconnects (e.g. after config change), the callback fires again.

## Type-safe Models

Extend the `Models` interface for type-safe model access:

```ts
declare module 'zhin.js' {
  interface Models {
    users: {
      id: number
      name: string
      email: string
      score: number
      created_at: number
    }
  }
}
```

## Checklist

- Configure `database` section in `zhin.config.yml`.
- Define models inside `useContext('database', ...)` callbacks.
- Use query operators for complex filtering.
- Wrap related operations in transactions.
- Extend the `Models` interface for type safety.
- Use migrations for schema changes in production.
