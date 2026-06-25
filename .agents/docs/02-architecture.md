# 02 - 全栈系统架构设计

---

## 1. 整体架构

```
┌──────────────────────────────────────────────────────────────────────┐
│                    投资监控浏览器扩展（Manifest V3）                    │
├─────────────────────────────┬────────────────────────────────────────┤
│        Popup（React）        │         Service Worker（TS）           │
│                              │                                        │
│  ┌───────────────────────┐   │   ┌────────────────────────────────┐   │
│  │   UI 层                │   │   │   chrome.alarms 调度器          │   │
│  │   - Popup 持仓表格      │   │   │   - 交易时间判断                │   │
│  │   - 添加标的弹窗        │   │   │   - stock-sdk 行情轮询          │   │
│  │   - 规则配置面板        │   │   │   - 规则匹配引擎                │   │
│  │   - 设置面板           │   │   │   - Web Notification 触发       │   │
│  └───────────────────────┘   │   └────────────────────────────────┘   │
│  ┌───────────────────────┐   │                                        │
│  │   业务逻辑层           │   │                                        │
│  │   - 数据格式化         │   │                                        │
│  │   - 表单校验          │   │                                        │
│  └───────────────────────┘   │                                        │
│  ┌───────────────────────┐   │                                        │
│  │   数据层              │   │                                        │
│  │   - wxt/storage 读取   │   │                                        │
│  │   - stock-sdk 搜索     │   │                                        │
│  └───────────────────────┘   │                                        │
├──────────────────────────────┴────────────────────────────────────────┤
│                         wxt/storage.local                              │
│              （持仓、规则、设置、行情缓存、触发状态）                     │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                     外部数据源（Service Worker 直连）                   │
│  ┌─────────────────┐  ┌─────────────────┐                            │
│  │  腾讯行情 API     │  │  东方财富 API    │                            │
│  └─────────────────┘  └─────────────────┘                            │
└──────────────────────────────────────────────────────────────────────┘
```

### 核心设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 扩展开发框架 | WXT | 文件系统生成 manifest、HMR、多浏览器支持、内置 storage |
| 市场数据获取 | Service Worker 中 stock-sdk 直连 | SDK 是纯 JS，可在 SW 运行；声明 host_permissions 后绕过 CORS |
| 数据持久化 | `wxt/storage` | WXT 封装 `chrome.storage.local`，带类型、默认值、迁移 |
| 通知 | Service Worker + Web Notification | Manifest V3 标准方案，popup 关闭后仍能触发 |
| 规则匹配 | Service Worker 中执行 | 行情在后台获取，匹配也在后台完成，减少 popup 负担 |
| UI 样式 | 原生 CSS + Minimal Dark | 轻量、可控、符合"无干扰"定位 |

---

## 2. 技术栈

| 层次 | 技术 | 版本 | 说明 |
|------|------|------|------|
| 扩展框架 | WXT | latest | 基于 Vite 的浏览器扩展开发框架 |
| 前端框架 | React | 19+ | 生态成熟 |
| 类型系统 | TypeScript | 5.x | 类型安全 |
| 构建工具 | Vite | WXT 内置 | 由 WXT 管理具体版本 |
| 包管理器 | pnpm | 9.x | 磁盘占用小、速度快 |
| 行情数据 | stock-sdk | latest | 零依赖、纯 JS、支持批量查询 |
| 本地存储 | wxt/storage | WXT 内置 | 封装 `chrome.storage.local` |
| 配置导出 | js-yaml | latest | YAML 解析/序列化 |
| Linter | oxlint | latest | Rust 编写，替代 ESLint |
| Formatter | oxfmt | latest | Rust 编写，替代 Prettier |
| 图标 | lucide-react | latest | 细线风格图标，契合 Minimal Dark |

---

## 3. 分层架构

### 3.1 目录结构（WXT 约定）

```
src/
├── entrypoints/              # WXT 入口点
│   ├── popup/                # 插件弹窗主 UI
│   │   ├── index.html
│   │   ├── main.tsx
│   │   └── App.tsx
│   ├── background.ts         # Service Worker
│   └── options/              # 可选：扩展选项页（MVP 暂不做）
│       ├── index.html
│       ├── main.tsx
│       └── App.tsx
├── components/               # React 组件
│   ├── Portfolio/
│   │   ├── AddStockModal.tsx
│   │   ├── PortfolioTable.tsx
│   │   └── ExportImportBar.tsx
│   ├── Rules/
│   │   ├── RuleForm.tsx
│   │   └── RuleList.tsx
│   ├── Settings/
│   │   └── SettingsPanel.tsx
│   └── Common/
│       ├── StatusIndicator.tsx
│       ├── EmptyState.tsx
│       ├── Button.tsx
│       ├── Input.tsx
│       ├── Card.tsx
│       └── Switch.tsx
├── hooks/                    # 自定义 Hooks
│   ├── usePortfolio.ts
│   ├── useRules.ts
│   ├── useMarketData.ts
│   └── useSettings.ts
├── services/                 # 数据访问层
│   ├── marketDataService.ts  # stock-sdk 封装
│   ├── portfolioService.ts   # storage 封装
│   ├── ruleService.ts        # storage 封装
│   └── settingsService.ts    # storage 封装
├── storage/                  # wxt/storage 封装
│   ├── portfolioStorage.ts
│   ├── ruleStorage.ts
│   ├── settingsStorage.ts
│   └── marketDataStorage.ts
├── stores/                   # Zustand 状态（popup 内使用）
│   ├── portfolioStore.ts
│   ├── ruleStore.ts
│   ├── marketDataStore.ts
│   └── settingsStore.ts
├── styles/                   # 全局样式 + 设计 tokens
│   ├── tokens.css            # CSS 变量
│   ├── global.css            # 全局样式
│   └── components/           # 组件样式模块
├── types/                    # TypeScript 类型
│   ├── portfolio.ts
│   ├── rule.ts
│   ├── marketData.ts
│   └── settings.ts
└── utils/                    # 工具函数
    ├── tradingTime.ts
    ├── ruleMatcher.ts
    └── formatters.ts
```

### 3.2 数据流

```
┌──────────────────────────────────────────────────────────────────────┐
│                           数据流架构                                  │
│                                                                      │
│  stock-sdk ──HTTP──▶ marketDataService ──▶ marketDataStorage        │
│                                                                      │
│  Popup ──storage API──▶ portfolioService ──▶ portfolioStorage       │
│                                                                      │
│  Popup ──storage API──▶ ruleService ──▶ ruleStorage                 │
│                                                                      │
│  marketDataStorage + ruleStorage ──▶ ruleMatcher ──▶ Notification   │
│                                                                      │
│  Popup ◀── zustand stores + storage API ──▶ React 组件              │
└──────────────────────────────────────────────────────────────────────┘
```

**关键数据流说明：**

1. **行情轮询流**：Service Worker 启动 `chrome.alarms` → alarm 触发时读取 storage 中的持仓和设置 → 调用 `marketDataService` → 写入 `marketDataStorage`。
2. **规则匹配流**：行情更新后，Service Worker 读取 `ruleStorage` 中的规则 → `ruleMatcher` 逐条匹配 → 命中则调用 `self.registration.showNotification(...)`。
3. **UI 数据流**：Popup 打开时从 storage 读取持仓/规则/行情缓存 → 更新 Zustand store → 渲染组件。用户操作后写回 storage。

---

## 4. 模块设计

### 4.1 持仓管理模块

**职责**：标的 CRUD、导入导出

**Storage API：**

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `portfolioStorage.getAll()` | 无 | `Portfolio[]` | 从 storage 读取 |
| `portfolioStorage.add(item)` | `Partial<Portfolio>` | `Portfolio` | 写入 storage，自动生成 id 和时间戳 |
| `portfolioStorage.remove(id)` | `string` | `void` | 删除持仓及其关联规则 |
| `portfolioStorage.import(data)` | `Portfolio[]` | `void` | 覆盖写入 |

**前端 service 封装：**

```typescript
// services/portfolioService.ts
import { portfolioStorage } from '@/storage/portfolioStorage';

export async function getPortfolios(): Promise<Portfolio[]> {
  return portfolioStorage.getAll();
}

export async function addPortfolio(code: string, name: string, market: string): Promise<Portfolio> {
  return portfolioStorage.add({ code, name, market });
}
```

### 4.2 数据监测模块

**职责**：行情轮询、数据格式化

> 此模块在 Service Worker 中运行，不涉及 Popup。

```typescript
// services/marketDataService.ts
import { StockSDK } from 'stock-sdk';

const sdk = new StockSDK();

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

export async function getDividendYield(code: string): Promise<number | null> {
  const details = await sdk.reference.dividendDetail(code);
  if (!details || details.length === 0) return null;
  return details[0].dividendYield ?? null;
}
```

### 4.3 规则引擎模块

**职责**：规则匹配、触发策略

**核心逻辑（Service Worker）：**

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

**触发策略（Service Worker 内存状态）：**

```typescript
// entrypoints/background.ts 中的触发逻辑

// 运行时内存状态：记录每条规则上一次的匹配结果，不持久化
const lastMatchedStates: Record<string, boolean> = {};

async function evaluateRules(rules: Rule[], marketData: MarketDataMap) {
  for (const rule of rules) {
    if (!rule.isActive) continue;

    const result = matchRule(rule, marketData);
    if (!result) continue;

    const prevMatched = lastMatchedStates[rule.id] ?? false;

    if (result.matched && !prevMatched) {
      showNotification(rule, result.currentValue);
      await ruleStorage.update(rule.id, { lastTriggeredAt: new Date().toISOString() });
    }

    lastMatchedStates[rule.id] = result.matched;
  }
}
```

### 4.4 通知模块

**职责**：浏览器通知弹出

**Service Worker 实现：**

```typescript
// entrypoints/background.ts
function showNotification(rule: Rule, currentValue: number) {
  const body = `${rule.name}\n当前值: ${currentValue}\n阈值: ${rule.condition.value}`;
  self.registration.showNotification('Investment Monitor', {
    body,
    icon: '/icon/128.png',
    badge: '/icon/32.png',
  });
}
```

### 4.5 设置模块

**职责**：更新频率、通知开关

**Settings 类型：**

```typescript
interface Settings {
  updateInterval: number;          // 轮询间隔（秒），默认 30
  notificationEnabled: boolean;    // 是否启用通知，默认 true
  soundEnabled: boolean;           // 是否启用声音，默认 false
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
  fetch: () => Promise<void>;
}

// stores/ruleStore.ts
interface RuleStore {
  rules: Rule[];
  fetch: () => Promise<void>;
  add: (portfolioId: string, rule: RuleInput) => Promise<void>;
  update: (id: string, rule: Partial<RuleInput>) => Promise<void>;
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

## 6. Minimal Dark 设计系统应用

### 6.1 设计 tokens（CSS 变量）

```css
/* styles/tokens.css */
:root {
  --background: #0A0A0F;
  --background-alt: #12121A;
  --muted: #1A1A24;
  --foreground: #FAFAFA;
  --muted-foreground: #71717A;
  --accent: #F59E0B;
  --accent-foreground: #0A0A0F;
  --accent-muted: rgba(245, 158, 11, 0.15);
  --border: rgba(255, 255, 255, 0.08);
  --border-hover: rgba(255, 255, 255, 0.15);
  --card: rgba(26, 26, 36, 0.6);
  --ring: #F59E0B;
  --up: #EF4444;        /* 中国股市：红涨 */
  --down: #22C55E;      /* 中国股市：绿跌 */
}
```

### 6.2 字体

```css
/* styles/global.css */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Space+Grotesk:wght@500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
  --font-display: 'Space Grotesk', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

### 6.3 组件风格

- **卡片**：`background: var(--card)`、`backdrop-filter: blur(8px)`、`border: 1px solid var(--border)`、`border-radius: 12px`
- **按钮 Primary**：`background: var(--accent)`、`color: var(--accent-foreground)`、`border-radius: 12px`
- **按钮 Secondary**：透明背景 + `border: 1px solid var(--border-hover)`
- **输入框**：`background: var(--muted)`、`border: 1px solid var(--border)`、`border-radius: 12px`
- **表格行**：交替使用 `--background` 和 `--background-alt`
- **hover**：边框变亮、按钮出现 amber glow

---

## 7. 非功能设计

### 7.1 性能

| 指标 | 目标 | 实现方式 |
|------|------|----------|
| Popup 打开时间 | < 1s | 原生 CSS、无重型组件库、按需读取 storage |
| Service Worker 启动时间 | < 2s | 逻辑精简、避免重型依赖 |
| 内存占用 | < 80MB | 无 Rust 后端、无 Electron |
| CPU 占用 | < 5%（轮询时） | 轮询间隔 30s，非交易时间不轮询 |
| 数据延迟 | < 3s | stock-sdk 批量查询 |

### 7.2 可靠性

| 场景 | 策略 |
|------|------|
| stock-sdk 请求失败 | 重试 3 次，间隔 1s/2s/4s |
| 网络断开 | badge 变红提示，有网后自动恢复 |
| storage 数据损坏 | 降级为空数据，不影响扩展运行 |
| 规则误触发 | 单次触发策略，不会反复打扰 |
| Service Worker 被回收 | chrome.alarms 会在下次触发时唤醒 |

### 7.3 安全

| 要求 | 实现 |
|------|------|
| 数据不外传 | 所有数据本地存储，仅向行情 API 发请求 |
| 无敏感信息 | 不存储 API Key、账户信息 |
| 最小权限 | manifest 只申请 `storage`、`alarms`、`notifications`、`host_permissions` |

---

## 8. 关键决策记录

| 决策 | 选择 | 备选方案 | 否决理由 |
|------|------|----------|----------|
| 扩展开发框架 | WXT | 原生 Vite 8 + 手写 manifest | WXT 开发体验更好，节省工程化时间 |
| UI 实现方式 | 原生 CSS + Minimal Dark | Ant Design / Tailwind | 原生 CSS 更轻量，Minimal Dark 符合产品气质 |
| 状态管理 | Zustand | Redux / Jotai | Zustand 最轻量，无 boilerplate |
| 本地存储 | wxt/storage | 直接操作 chrome.storage.local | WXT 封装带类型和迁移 |
| 行情数据 | stock-sdk 在 SW 中直连 | 在 Popup 中请求 | Popup 生命周期短，后台轮询需要 SW |
| Linter/Formatter | oxlint + oxfmt | ESLint + Prettier | Rust 工具链更快，2026 年已成熟 |
| 通知触发 | Service Worker + Web Notification | 前端 Notification | popup 关闭后仍能触发 |
