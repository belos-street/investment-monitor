# 03 - 工程化基建

---

## 1. 项目初始化

### 1.1 创建项目

```bash
# 使用 WXT 官方脚手架（React + TypeScript 模板）
pnpm dlx wxt@latest init investment-monitor

# 选择模板时选 React + TypeScript
cd investment-monitor
pnpm install
```

### 1.2 目录结构

```
investment-monitor/
├── src/
│   ├── entrypoints/              # WXT 入口点
│   │   ├── popup/                # 插件弹窗 UI
│   │   │   ├── index.html
│   │   │   ├── main.tsx
│   │   │   └── App.tsx
│   │   ├── background.ts         # Service Worker
│   │   └── options/              # 可选：扩展选项页（MVP 暂不做）
│   ├── components/               # React 组件
│   ├── hooks/                    # 自定义 Hooks
│   ├── services/                 # 数据访问层
│   ├── storage/                  # wxt/storage 封装
│   ├── stores/                   # Zustand 状态
│   ├── styles/                   # 全局样式 + CSS tokens
│   ├── types/                    # TypeScript 类型
│   └── utils/                    # 工具函数
├── public/                       # 静态资源
├── .wxt/                         # WXT 生成的类型和配置
├── wxt.config.ts                 # WXT 配置
├── package.json
├── tsconfig.json
├── .oxlintrc.json                # oxlint 配置
├── .oxfmtrc.json                 # oxfmt 配置
└── README.md
```

---

## 2. 依赖配置

### 2.1 前端依赖（package.json）

```json
{
  "name": "investment-monitor",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "wxt",
    "dev:firefox": "wxt -b firefox",
    "build": "wxt build",
    "build:firefox": "wxt build -b firefox",
    "zip": "wxt zip",
    "zip:firefox": "wxt zip -b firefox",
    "compile": "tsc --noEmit",
    "lint": "oxlint .",
    "lint:fix": "oxlint . --fix",
    "format": "oxfmt --check .",
    "format:fix": "oxfmt ."
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "stock-sdk": "latest",
    "js-yaml": "^4.1.0",
    "zustand": "^5.0.0",
    "lucide-react": "^0.x.0"
  },
  "devDependencies": {
    "@types/js-yaml": "^4.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@wxt-dev/module-react": "latest",
    "oxfmt": "latest",
    "oxlint": "latest",
    "typescript": "^5.6.0",
    "wxt": "latest"
  }
}
```

> **说明**：Vite 作为 WXT 的依赖被自动引入，具体版本由 WXT 管理，无需在项目中显式安装。

### 2.2 运行时权限（wxt.config.ts）

```typescript
import { defineConfig } from 'wxt';

export default defineConfig({
  manifest: {
    name: 'Investment Monitor',
    description: 'A lightweight browser extension for monitoring A-share and ETF quotes with rule-based alerts.',
    version: '0.1.0',
    permissions: ['storage', 'alarms', 'notifications'],
    host_permissions: [
      '*://qt.gtimg.cn/*',
      '*://push2.eastmoney.com/*',
      '*://*.eastmoney.com/*',
    ],
    action: {
      default_popup: 'popup.html',
      default_icon: {
        '16': 'icon/16.png',
        '32': 'icon/32.png',
        '48': 'icon/48.png',
        '128': 'icon/128.png',
      },
    },
    icons: {
      '16': 'icon/16.png',
      '32': 'icon/32.png',
      '48': 'icon/48.png',
      '128': 'icon/128.png',
    },
  },
});
```

> **host_permissions 说明**：根据 stock-sdk 实际请求的 API 域名调整。上述为常见行情 API 域名示例，需在开发中核实并补充。

---

## 3. 代码规范

### 3.1 oxlint 配置（.oxlintrc.json）

```json
{
  "plugins": ["import", "typescript", "react", "react-hooks", "jsx-a11y"],
  "categories": {
    "correctness": "error",
    "suspicious": "warn",
    "perf": "warn",
    "style": "off"
  },
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "warn",
    "react-hooks/exhaustive-deps": "error"
  },
  "env": {
    "browser": true,
    "es2024": true
  },
  "globals": {
    "chrome": "readonly"
  },
  "ignorePatterns": [
    "node_modules",
    "dist",
    ".wxt",
    "*.config.ts"
  ]
}
```

### 3.2 oxfmt 配置（.oxfmtrc.json）

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "ignore": [
    "node_modules",
    "dist",
    ".wxt"
  ]
}
```

### 3.3 TypeScript 配置（tsconfig.json）

```json
{
  "extends": "./.wxt/tsconfig.json",
  "compilerOptions": {
    "target": "ES2024",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "react-jsx",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src", ".wxt/types"]
}
```

---

## 4. WXT 配置

### 4.1 完整 wxt.config.ts 示例

```typescript
import { defineConfig } from 'wxt';

export default defineConfig({
  srcDir: 'src',
  extensionApi: 'chrome',
  manifestVersion: 3,
  modules: ['@wxt-dev/module-react'],
  manifest: {
    name: 'Investment Monitor',
    description: 'A lightweight browser extension for monitoring A-share and ETF quotes with rule-based alerts.',
    version: '0.1.0',
    permissions: ['storage', 'alarms', 'notifications'],
    host_permissions: [
      '*://qt.gtimg.cn/*',
      '*://push2.eastmoney.com/*',
      '*://*.eastmoney.com/*',
    ],
    action: {
      default_popup: 'popup.html',
      default_icon: {
        '16': 'icon/16.png',
        '32': 'icon/32.png',
        '48': 'icon/48.png',
        '128': 'icon/128.png',
      },
    },
    icons: {
      '16': 'icon/16.png',
      '32': 'icon/32.png',
      '48': 'icon/48.png',
      '128': 'icon/128.png',
    },
  },
  runner: {
    startUrls: ['https://www.baidu.com'],
  },
});
```

### 4.2 关键配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `srcDir` | `src` | 源码目录 |
| `extensionApi` | `chrome` | 使用 Chrome 扩展 API |
| `manifestVersion` | `3` | Manifest V3 |
| `modules` | `['@wxt-dev/module-react']` | React 支持模块 |
| `permissions` | `storage, alarms, notifications` | 最小必要权限 |
| `host_permissions` | 行情 API 域名 | 允许跨域请求行情数据 |

---

## 5. 开发环境

### 5.1 启动命令

```bash
# 安装依赖
pnpm install

# 启动开发模式（自动打开浏览器加载扩展）
pnpm dev

# 构建生产版本
pnpm build

# 打包为 zip（用于 Chrome Web Store）
pnpm zip
```

### 5.2 调试

- **Popup**：右键工具栏图标 → 点击弹出窗口 → 在弹窗内右键 → 检查
- **Service Worker**：打开 `chrome://extensions/` → 找到扩展 → 点击「Service Worker」链接
- **Storage**：在 Popup 或 Service Worker 的 DevTools 中查看 `chrome.storage.local`

---

## 6. Minimal Dark CSS 基础

### 6.1 tokens.css

```css
/* src/styles/tokens.css */
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
  --up: #EF4444;
  --down: #22C55E;

  --font-display: 'Space Grotesk', system-ui, sans-serif;
  --font-body: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;

  --transition-fast: 150ms ease-out;
  --transition-base: 200ms ease-out;
  --transition-slow: 300ms ease-out;
}
```

### 6.2 global.css

```css
/* src/styles/global.css */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Space+Grotesk:wght@500;600;700&family=JetBrains+Mono:wght@400;500&display=swap');
@import './tokens.css';

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body {
  width: 380px;
  min-height: 600px;
  background: var(--background);
  color: var(--foreground);
  font-family: var(--font-body);
  font-size: 14px;
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

button,
input,
select,
textarea {
  font-family: inherit;
  font-size: inherit;
}

button {
  cursor: pointer;
  border: none;
  background: none;
}

/* 滚动条 */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: var(--border-hover);
  border-radius: 3px;
}
```

### 6.3 组件样式约定

每个组件一个 `.module.css` 文件，使用 CSS 变量：

```css
/* src/components/Common/Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  height: 40px;
  padding: 0 16px;
  border-radius: var(--radius-lg);
  font-weight: 500;
  transition: all var(--transition-base);
}

.primary {
  background: var(--accent);
  color: var(--accent-foreground);
}

.primary:hover {
  filter: brightness(1.1);
  box-shadow: 0 0 20px rgba(245, 158, 11, 0.4);
}

.secondary {
  background: transparent;
  color: var(--foreground);
  border: 1px solid var(--border-hover);
}

.secondary:hover {
  background: rgba(255, 255, 255, 0.05);
}
```

---

## 7. Git 规范

### 7.1 分支策略（单人开发）

| 分支 | 用途 | 命名 |
|------|------|------|
| `main` | 稳定版本 | — |
| `feat/*` | 功能开发 | `feat/popup-ui` |
| `fix/*` | Bug 修复 | `fix/rule-trigger` |

### 7.2 Commit 规范

```
<type>(<scope>): <description>

类型：
- feat:     新功能
- fix:      修复
- refactor: 重构
- style:    样式调整
- docs:     文档
- chore:    工程化

示例：
feat(popup): 添加股票搜索弹窗
fix(background): 修复规则重复触发问题
refactor(storage): 重构 wxt/storage 封装
```

---

## 8. VS Code 配置建议

### 8.1 推荐扩展（.vscode/extensions.json）

```json
{
  "recommendations": [
    "oxc.oxc-vscode"
  ]
}
```

### 8.2 工作区设置（.vscode/settings.json）

```json
{
  "editor.defaultFormatter": "oxc.oxc-vscode",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.oxlint": "explicit"
  },
  "oxc.enable": true,
  "oxc.configPath": ".oxlintrc.json"
}
```

---

## 9. 环境变量

WXT 支持通过 `.env` 文件注入环境变量，但扩展的运行时环境变量主要用于构建时配置。

| 变量 | 用途 | 默认值 |
|------|------|--------|
| `VITE_DEFAULT_INTERVAL` | 默认轮询间隔（秒） | `30` |
| `VITE_TRADING_AM_START` | 上午交易开始时间 | `09:30` |
| `VITE_TRADING_AM_END` | 上午交易结束时间 | `11:30` |
| `VITE_TRADING_PM_START` | 下午交易开始时间 | `13:00` |
| `VITE_TRADING_PM_END` | 下午交易结束时间 | `15:00` |

> 环境变量仅用于开发时的可配置默认值，运行时设置通过 `wxt/storage` 持久化。A 股交易时段逻辑在 `tradingTime.ts` 中实现。
