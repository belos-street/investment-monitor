# 02 - 全栈系统架构设计

---

## 1. 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         Tauri 2.0 桌面应用                       │
├───────────────────────────┬─────────────────────────────────────┤
│        前端（React）       │        Rust 后端（Tauri Core）       │
│                           │                                     │
│  ┌───────────────────┐   │   ┌─────────────────────────────┐   │
│  │   UI 层            │   │   │   Tauri Commands             │   │
│  │   - 主窗口 (表格)    │   │   │   - get_portfolios          │   │
│  │   - 规则配置面板     │   │   │   - add/remove_portfolio    │   │
│  │   - 设置面板        │   │   │   - get/add/update_rule     │   │
│  │   - 系统托盘        │   │   │   - send_notification       │   │
│  └───────────────────┘   │   │   - export/import_config     │   │
│  ┌───────────────────┐   │   │   - get/update_settings      │   │
│  │   业务逻辑层        │   │   └─────────────────────────────┘   │
│  │   - 轮询调度器      │   │   ┌─────────────────────────────┐   │
│  │   - 规则匹配引擎    │   │   │   Tauri 插件                 │   │
│  │   - 数据格式化      │   │   │   - tauri-plugin-store       │   │
│  └───────────────────┘   │   │   - tauri-plugin-notification │   │
│  ┌───────────────────┐   │   └─────────────────────────────┘   │
│  │   数据层            │   │                                     │
│  │   - stock-sdk      │   │                                     │
│  │   - zustand store  │   │                                     │
│  └───────────────────┘   │                                     │
├───────────────────────────┴─────────────────────────────────────┤
│                     外部数据源（仅前端可达）                       │
│  ┌─────────────────┐  ┌─────────────────┐                      │
│  │  腾讯行情 API     │  │  东方财富 API    │                      │
│  └─────────────────┘  └─────────────────┘                      │
└─────────────────────────────────────────────────────────────────┘
```

### 核心设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 市场数据获取 | 前端 stock-sdk 直连 | stock-sdk 是纯 JS 库，无法在 Rust 中运行；前端直连无需 IPC 开销 |
| 数据持久化 | Rust 端 tauri-plugin-store | 官方插件，KV 存储，跨平台一致 |
| 系统通知 | Rust 端 Tauri 原生通知 | 比 node-notifier 更可靠，权限管理更简单 |
| 规则匹配 | 前端逻辑 | 匹配对象是前端内存中的行情数据，无需走 IPC |

---

## 2. 技术栈

| 层次 | 技术 | 版本 | 说明 |
|------|------|------|------|
| 桌面框架 | Tauri | 2.x | 轻量、跨平台、Rust 后端 |
| 前端框架 | React | 18+ | 生态成熟 |
| 类型系统 | TypeScript | 5.x | 类型安全 |
| 构建工具 | Vite | 6.x | 快速 HMR |
| 包管理器 | pnpm | 9.x | 磁盘占用小、速度快 |
| UI 组件库 | Ant Design | 5.x | 表格、表单、通知组件齐全 |
| 状态管理 | Zustand | 5.x | 轻量、无 boilerplate |
| 行情数据 | stock-sdk | latest | 零依赖、纯 JS、支持批量查询 |
| 本地存储 | tauri-plugin-store | latest | Tauri 官方 KV 插件 |
| 配置导出 | js-yaml | latest | YAML 解析/序列化 |

---

## 3. 分层架构

### 3.1 前端分层

```
src/
├── components/          # UI 组件（纯展示 + 交互）
│   ├── Portfolio/       # 持仓相关组件
│   │   ├── AddStockModal.tsx      # 添加标的弹窗
│   │   ├── PortfolioTable.tsx     # 持仓列表表格
│   │   └── ExportImportBar.tsx    # 导入导出操作栏
│   ├── Rules/           # 规则相关组件
│   │   ├── RuleForm.tsx           # 规则配置表单
│   │   └── RuleList.tsx           # 规则列表
│   ├── Settings/        # 设置组件
│   │   └── SettingsPanel.tsx      # 设置面板
│   └── Common/          # 通用组件
│       ├── StatusIndicator.tsx    # 状态栏指示器
│       └── EmptyState.tsx         # 空状态占位
├── hooks/               # 自定义 Hooks
│   ├── usePortfolio.ts           # 持仓 CRUD Hook
│   ├── useRules.ts               # 规则 CRUD Hook
│   ├── useMarketData.ts          # 行情轮询 Hook
│   └── useNotification.ts        # 通知 Hook
├── services/            # 数据访问层（封装 invoke + stock-sdk）
│   ├── portfolioService.ts       # invoke 封装
│   ├── ruleService.ts            # invoke 封装
│   ├── marketDataService.ts      # stock-sdk 封装
│   └── notificationService.ts    # invoke 封装
├── stores/              # Zustand 状态
│   ├── portfolioStore.ts         # 持仓状态
│   ├── ruleStore.ts              # 规则状态
│   ├── marketDataStore.ts        # 行情数据状态
│   └── settingsStore.ts          # 设置状态
├── types/               # TypeScript 类型
│   ├── portfolio.ts
│   ├── rule.ts
│   └── marketData.ts
└── utils/               # 工具函数
    ├── tradingTime.ts            # 交易时间判断
    └── ruleMatcher.ts            # 规则匹配逻辑
```

### 3.2 数据流

```
┌─────────────────────────────────────────────────────────────┐
│                    数据流架构                                  │
│                                                             │
│  stock-sdk ──HTTP──▶ marketDataService ──▶ marketDataStore  │
│                                                          │     │
│  Tauri Command ──IPC──▶ portfolioService ──▶ portfolioStore│  │
│                                                          │     │
│  Tauri Command ──IPC──▶ ruleService ──▶ ruleStore        │  │
│                                                          │     │
│  marketDataStore + ruleStore ──▶ ruleMatcher ──▶ notificationService
│                                                               │
│  UI ◀── zustand stores ──▶ React 组件                        │
└─────────────────────────────────────────────────────────────┘
```

**关键数据流说明：**

1. **行情轮询流**：`useMarketData` Hook 启动定时器 → 调用 `marketDataService` → 更新 `marketDataStore` → UI 自动重渲染
2. **规则匹配流**：`marketDataStore` 更新时触发 `ruleMatcher` → 逐条匹配 `ruleStore` 中的活跃规则 → 命中则调用 `notificationService` → 调用 `invoke('send_notification')`
3. **CRUD 流**：用户操作 UI → 调用 `portfolioService/ruleService` → `invoke()` IPC → Rust 端写 `tauri-plugin-store` → 返回结果 → 更新 Zustand store

---

## 4. 模块设计

### 4.1 持仓管理模块

**职责**：标的 CRUD、导入导出

**Tauri Commands（Rust 端）：**

| Command | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `get_portfolios` | 无 | `Portfolio[]` | 从 tauri-plugin-store 读取 |
| `add_portfolio` | `{ code, name, market }` | `Portfolio` | 写入 store，自动生成 id 和时间戳 |
| `remove_portfolio` | `{ id }` | `void` | 删除持仓及其关联规则 |
| `export_config` | 无 | `string`（YAML） | 读取持仓 + 规则，序列化为 YAML |
| `import_config` | `{ yaml }` | `void` | 解析 YAML，覆盖写入 store |

**前端 service 封装：**

```typescript
// services/portfolioService.ts
import { invoke } from '@tauri-apps/api/core';

export async function getPortfolios(): Promise<Portfolio[]> {
  return invoke<Portfolio[]>('get_portfolios');
}

export async function addPortfolio(code: string, name: string, market: string): Promise<Portfolio> {
  return invoke<Portfolio>('add_portfolio', { code, name, market });
}
```

### 4.2 数据监测模块

**职责**：行情轮询、数据格式化

> 此模块纯前端实现，不涉及 Tauri IPC。

```typescript
// services/marketDataService.ts
import { StockSDK } from 'stock-sdk';

const sdk = new StockSDK();

// A 股 / ETF 实时行情（含 PE、PB；ETF 的 pe/pb 通常为空）
export async function fetchStockQuotes(codes: string[]): Promise<StockQuote[]> {
  const results = await sdk.quotes.cn(codes);
  return results.map(item => ({
    code: item.code,
    name: item.name,
    price: item.price,
    changePercent: item.changePercent,
    pe: item.pe,
    pb: item.pb,
  }));
}

// 基金净值（仅作参考，不用于主界面实时价/涨跌幅）
export async function fetchFundQuotes(codes: string[]): Promise<FundQuote[]> {
  const results = await sdk.quotes.fund(codes);
  return results.map(item => ({
    code: item.code,
    name: item.name,
    nav: item.nav,        // 单位净值
    accNav: item.accNav,   // 累计净值
    change: item.change,   // 净值日变动（非实时涨跌幅）
  }));
}

// 个股股息率：取最近一条分红记录的 dividendYield
export async function getDividendYield(code: string): Promise<number | null> {
  const details = await sdk.reference.dividendDetail(code);
  if (!details || details.length === 0) return null;
  return details[0].dividendYield ?? null;
}
```

### 4.3 规则引擎模块

**职责**：规则匹配、触发策略

**核心逻辑（前端）：**

```typescript
// utils/ruleMatcher.ts

interface MatchResult {
  ruleId: string;
  matched: boolean;
  currentValue: number;
  threshold: number;
}

export function matchRule(rule: Rule, marketData: MarketDataMap): MatchResult | null {
  const data = marketData[rule.portfolioId];
  if (!data) return null;

  const { field, operator, value: threshold } = rule.condition;
  const currentValue = data[field];
  if (currentValue === undefined || currentValue === null) return null;

  const matched = compare(currentValue, operator, threshold);
  return { ruleId: rule.id, matched, currentValue, threshold };
}

function compare(current: number, operator: string, threshold: number): boolean {
  switch (operator) {
    case '<=': return current <= threshold;
    case '>=': return current >= threshold;
    case '<':  return current < threshold;
    case '>':  return current > threshold;
    case '==': return current === threshold;
    default:   return false;
  }
}
```

**触发策略：**

```typescript
// hooks/useMarketData.ts 中的触发逻辑

// 运行时内存状态：记录每条规则上一次的匹配结果，不持久化
const lastMatchedStates: Record<string, boolean> = {};

function evaluateRules(rules: Rule[], marketData: MarketDataMap) {
  for (const rule of rules) {
    if (!rule.isActive) continue;

    const result = matchRule(rule, marketData);
    if (!result) continue;

    const prevMatched = lastMatchedStates[rule.id] ?? false;

    // 条件从 false → true，触发通知并记录触发时间
    if (result.matched && !prevMatched) {
      sendNotification(rule, result.currentValue);
      updateRule(rule.id, { lastTriggeredAt: new Date().toISOString() });
    }

    // 更新内存中的匹配状态（用于下次判断是否复位）
    lastMatchedStates[rule.id] = result.matched;
  }
}
```

### 4.4 通知模块

**职责**：系统通知弹出

**Tauri Command：**

```rust
// Rust 端
use tauri_plugin_notification::NotificationExt;

#[tauri::command]
fn send_notification(app: tauri::AppHandle, title: String, body: String) {
    app.notification()
        .builder()
        .title(title)
        .body(body)
        .show()
        .unwrap();
}
```

**前端调用：**

```typescript
// services/notificationService.ts
export async function sendNotification(rule: Rule, currentValue: number) {
  const body = `${rule.name}\n当前值: ${currentValue}\n阈值: ${rule.condition.value}`;
  await invoke('send_notification', { title: 'Investment Monitor', body });
}
```

### 4.5 设置模块

**职责**：更新频率、通知开关、开机自启

**Tauri Commands：**

| Command | 参数 | 返回值 | 说明 |
|---------|------|--------|------|
| `get_settings` | 无 | `Settings` | 读取设置 |
| `update_settings` | `{ settings }` | `void` | 保存设置 |

**Settings 类型：**

```typescript
interface Settings {
  updateInterval: number;    // 轮询间隔（秒），默认 30
  notificationEnabled: boolean;  // 是否启用通知，默认 true
  soundEnabled: boolean;     // 是否启用声音，默认 false
}
```

---

## 5. 状态管理设计

```typescript
// stores/portfolioStore.ts
interface PortfolioStore {
  portfolios: Portfolio[];
  loading: boolean;
  error: string | null;
  fetch: () => Promise<void>;
  add: (code: string, name: string, market: string) => Promise<void>;
  remove: (id: string) => Promise<void>;
}

// stores/marketDataStore.ts
interface MarketDataStore {
  data: Record<string, MarketData>;   // key = portfolioId
  lastUpdated: string | null;
  status: 'idle' | 'fetching' | 'error';
  update: (portfolios: Portfolio[]) => Promise<void>;
}

// stores/ruleStore.ts
interface RuleStore {
  rules: Rule[];
  fetch: (portfolioId?: string) => Promise<void>;
  add: (portfolioId: string, rule: RuleInput) => Promise<void>;
  update: (id: string, rule: RuleInput) => Promise<void>;
  remove: (id: string) => Promise<void>;
  toggle: (id: string) => Promise<void>;
}

// stores/settingsStore.ts
interface SettingsStore {
  settings: Settings;
  fetch: () => Promise<void>;
  update: (settings: Partial<Settings>) => Promise<void>;
}
```

---

## 6. 非功能设计

### 6.1 性能

| 指标 | 目标 | 实现方式 |
|------|------|----------|
| 启动时间 | < 2s | Tauri 比 Electron 快 10x+ |
| 内存占用 | < 100MB | Rust 后端轻量，前端 React 按需渲染 |
| CPU 占用 | < 5%（轮询时） | 轮询间隔 30s，非交易时间不轮询 |
| 数据延迟 | < 3s | stock-sdk 批量查询，单次请求覆盖所有持仓（实际受网络影响） |

### 6.2 可靠性

| 场景 | 策略 |
|------|------|
| stock-sdk 请求失败 | 重试 3 次，间隔 1s/2s/4s |
| 网络断开 | 状态栏提示，有网后自动恢复 |
| 存储文件损坏 | 降级为空数据，不影响运行 |
| 规则误触发 | 单次触发策略，不会反复打扰 |

### 6.3 安全

| 要求 | 实现 |
|------|------|
| 数据不外传 | 所有数据本地存储，仅向行情 API 发请求 |
| 无敏感信息 | 不存储 API Key、账户信息 |
| 代码签名 | macOS/Windows 应用签名（发布时） |

---

## 7. 关键决策记录

| 决策 | 选择 | 备选方案 | 否决理由 |
|------|------|----------|----------|
| 前端框架 | React + Ant Design | Vue + Element Plus | React 生态更成熟，Ant Design 表格组件更强 |
| 状态管理 | Zustand | Redux / Jotai | Zustand 最轻量，无 boilerplate |
| 本地存储 | tauri-plugin-store | SQLite / JSON 文件 | 官方插件，KV 够用，无需引入数据库 |
| 行情数据 | stock-sdk 直连 | 自封装 HTTP 请求 | SDK 内置限流、重试、统一格式 |
| 包管理器 | pnpm | npm / bun | pnpm 最稳定，磁盘占用小 |
