# 前端技术需求文档

## 1. 目标

构建一个文字版 MMORPG《贝克莱世界》的 Web 客户端，提供顺畅的命令输入、实时文本流输出、账户系统及基本社交界面。首要 KPI：

- 首屏可交互时间 ≤ 1.5 s（4G 网络）
- WebSocket 文本流延迟 ≤ 300 ms
- 核心交互（发送指令、查看反馈）错误率 < 0.1 %

## 2. 技术栈

| 类别     | 选型                          | 说明                                 |
| -------- | ----------------------------- | ------------------------------------ |
| 框架     | Next.js 14 (React 18)         | App Router、Server/Client Components |
| 语言     | TypeScript 5.x                | 严格模式                             |
| UI 库    | Tailwind CSS + shadcn/ui      | 原子化样式 + 可访问的组件            |
| 状态管理 | Zustand + TanStack Query      | 本地 UI 状态 / 服务器数据            |
| 实时通信 | WebSocket (WS)                | 双向流；备用 SSE                     |
| 认证     | NextAuth.js (JWT)             | 邮箱 / OAuth                         |
| 构建     | Vite (via Turbopack 未来可切) | Dev < 100 ms HMR                     |
| 质量     | ESLint、Prettier、Vitest      | 代码规范 / 单测覆盖率 60 % 起步      |
| 部署     | Vercel / 自托管 Node          | 无 Docker 依赖                       |

## 3. 路由设计

| 路径       | 类型      | 描述                   |
| ---------- | --------- | ---------------------- |
| `/`        | `static`  | 营销落地页             |
| `/login`   | `RSC`     | 登录 / 注册            |
| `/play`    | `RSC`+CSR | 游戏主界面（需要登录） |
| `/profile` | `RSC`     | 用户信息               |

> RSC：React Server Component；CSR：Client-Side Rendering

## 4. 组件拆分

- `GameLayout`
  - `Sidebar`
    - 在线玩家列表
    - 系统菜单
  - `MainWindow`
    - `MessageStream`（只读，按时间追加）
    - `CommandInput`（历史 ↑↓、快捷键 Tab→ 补全）
  - `HUD`（显示位置、状态、背包摘要）
- `AuthForm`（邮箱 / OAuth）
- `UserAvatar`、`Tooltip` 等基础组件

## 5. 状态与数据流

```
            ┌──────────────┐
            │  Zustand     │  UI 本地状态（输入框、面板开关）
            └─────▲────────┘
                  │  跨组件
┌──────────┐  请求 │          │  订阅
│ React UI │ ─────┼──────────┼────────► TanStack Query
└──────────┘      │          │  HTTP / WS
                  ▼
            Backend API (Fastify)
```

- **Zustand**：轻量、函数式。避免冗余 Redux。
- **TanStack Query**：负责缓存、重试、后台刷新。
- WebSocket hooks：`useGameStream`, `usePlayerPresence`。

## 6. 与后端接口约定

```ts
// post player text command
POST /api/action
{
  text: string
}

// example response stream frame
{
  type: 'chunk',
  content: '你走进了一条狭窄的巷子…'
}
```

- WebSocket URL：`wss://domain.com/ws/play?token=...`
- 心跳 30 s；服务器 60 s 超时无交互断链。

## 7. 实时文本流

1. 客户端 `WebSocket` 连接成功后订阅 `message` 事件。
2. 服务端分片推送：每 ≤ 50 字符一个 `chunk` 带序号。
3. 客户端按序号重组，滚动到底部，支持暂停自动滚动。
4. 历史消息保存在 IndexedDB（近期 500 条）。

## 8. 性能优化

- 使用 `next/font/google` 动态子集。
- 游戏页启用 `Suspense` + `use streaming` 渐进渲染。
- Tree-shaking + route-level code splitting。
- 图片资源（未来头像等）用 `next/image` lazy。

## 9. 可访问性 & 国际化

- 所有交互元素 `tabIndex`、`aria-*` 完整。
- 文本方向、字号可调整。
- i18n：`next-intl`，默认简体中文，预留 EN。

## 10. 测试策略

| 层级   | 工具                            | 覆盖内容                 |
| ------ | ------------------------------- | ------------------------ |
| 单元   | Vitest + @testing-library/react | 工具函数、组件状态       |
| 集成   | Playwright Component            | 关键表单、流式展示       |
| 端到端 | Playwright E2E                  | 登录 → 指令 → 反馈流闭环 |

CI 拉取 `main` 分支后自动运行全部测试。

## 11. 目录结构（示例）

```text
src/
  app/
    layout.tsx
    page.tsx
    play/
      page.tsx        # RSC 入口
      GameLayout.tsx
      MessageStream.tsx
      CommandInput.tsx
  components/
    ui/...
  hooks/
    useGameStream.ts
    useZustandStore.ts
  lib/
    api.ts            # fetch 封装
    ws.ts             # WebSocket 客户端
  styles/
    globals.css
  tests/
```

## 12. 迭代计划

| Sprint | 重点                    |
| ------ | ----------------------- |
| S1     | 路由 scaffold、Auth UI  |
| S2     | WebSocket 通道 & 流渲染 |
| S3     | HUD、Sidebar、命令历史  |
| S4     | 性能 & 无障碍优化       |

---

以上为前端 MVP 技术需求，后续根据产品反馈持续迭代。
