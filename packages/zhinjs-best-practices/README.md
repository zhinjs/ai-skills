# zhinjs-best-practices

此目录名称用于对齐仓库结构（非 npm 包名）。

Zhin 插件开发最佳实践集合, 参考 `zhin/examples/test-bot` 的结构与用法，帮助 AI agent 生成更可靠的插件方案。

## 适用场景

- 创建新的 Zhin 插件或 bot 项目。
- 设计插件目录结构与入口文件。
- 规划配置文件、插件加载与运行脚本。
- 评审代码是否符合 Zhin 官方示例。

## 规则列表

### 1. 项目结构对齐 test-bot

建议遵循以下结构（与 `examples/test-bot` 相同风格）：

```
project/
├── data/                # 数据目录（设备信息、缓存）
├── src/
│   ├── index.ts         # 应用入口
│   └── plugins/         # 插件目录
│       └── your-plugin.ts
├── zhin.config.ts       # 配置文件
├── package.json
└── tsconfig.json
```

### 2. 明确入口文件职责

`src/index.ts` 只负责创建 App、加载配置与启动：

```ts
import { createApp } from 'zhin.js'

const app = await createApp({
  plugins: ['<plugin-name>']
})

await app.start()
```

> 如果运行环境不支持顶层 await，请使用 `async function main()` 包裹上述逻辑。

### 3. 插件入口必须调用 usePlugin()

确保每个插件文件以 `usePlugin()` 作为初始化入口：

```ts
import { usePlugin } from '@zhin.js/core'

const plugin = usePlugin()

plugin.onMounted(() => {
  plugin.logger.info('Plugin mounted')
})
```

### 4. 配置文件集中管理

使用 `zhin.config.ts` 统一配置插件、bot 与 adapter：

```ts
import { defineConfig } from 'zhin.js'

export default defineConfig({
  plugin_dirs: ['./src/plugins'],
  plugins: ['<plugin-name>'],
  bots: [{
    context: 'process',
    name: 'dev-bot'
  }]
})
```

### 5. 插件职责单一

一个插件只负责一个功能域（命令、日志、定时任务等），避免把不相关的命令与调度逻辑堆在同一文件。

### 6. 使用内置中间件与命令系统

优先通过 `MessageCommand` 与 `addMiddleware` 扩展功能，避免自建消息解析。

### 7. 释放资源必须放在 onDispose

插件中使用的定时器、连接、监听器，必须在 `onDispose` 内清理。

### 8. 运行脚本保持一致

推荐使用以下脚本结构：

```json
{
  "scripts": {
    "dev": "pnpm zhin dev",
    "start": "pnpm zhin start"
  }
}
```

> 以上脚本假设 `zhin` 已作为项目依赖安装（例如在 `package.json` 中）。

### 9. 区分应用入口与插件依赖

- 应用入口使用 `zhin.js`（如 `createApp`、`defineConfig`）。
- 插件内部使用 `@zhin.js/core`（如 `usePlugin`、`MessageCommand`）。

## 参考

- [test-bot example](https://github.com/zhinjs/zhin/tree/main/examples/test-bot)
- Zhin 核心 API: `@zhin.js/core`
