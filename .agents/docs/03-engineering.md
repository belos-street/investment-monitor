# 03 - 工程化基建

---

## 1. 项目初始化

### 1.1 创建项目

```bash
# 使用 Tauri 官方脚手架（React + TypeScript 模板）
pnpm create tauri-app investment-monitor -- --template react-ts

cd investment-monitor
pnpm install
```

### 1.2 目录结构

```
investment-monitor/
├── src-tauri/                    # Rust 后端
│   ├── src/
│   │   ├── commands/             # Tauri Commands
│   │   │   ├── mod.rs
│   │   │   ├── portfolio.rs
│   │   │   ├── rule.rs
│   │   │   ├── notification.rs
│   │   │   └── settings.rs
│   │   ├── main.rs               # 入口
│   │   └── lib.rs                # Command 注册
│   ├── Cargo.toml
│   └── tauri.conf.json           # Tauri 配置
├── src/                          # React 前端
│   ├── components/
│   │   ├── Portfolio/
│   │   ├── Rules/
│   │   ├── Settings/
│   │   └── Common/
│   ├── hooks/
│   ├── services/
│   ├── stores/
│   ├── types/
│   ├── utils/
│   ├── App.tsx
│   └── main.tsx
├── docs/                         # 项目文档
│   └── vibe-flow/
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

---

## 2. 依赖配置

### 2.1 前端依赖（package.json）

```json
{
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-notification": "^2.0.0",
    "@tauri-apps/plugin-store": "^2.0.0",
    "antd": "^5.20.0",
    "zustand": "^5.0.0",
    "stock-sdk": "latest",
    "js-yaml": "^4.1.0"
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2.0.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@types/js-yaml": "^4.0.0",
    "@typescript-eslint/eslint-plugin": "^8.0.0",
    "@typescript-eslint/parser": "^8.0.0",
    "eslint": "^8.57.0",
    "eslint-plugin-react": "^7.35.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "typescript": "^5.6.0",
    "vite": "^6.0.0",
    "@vitejs/plugin-react": "^4.3.0"
  }
}
```

### 2.2 Rust 依赖（Cargo.toml）

```toml
[dependencies]
tauri = { version = "2", features = ["tray-icon"] }
tauri-plugin-notification = "2"
tauri-plugin-store = "2"
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

---

## 3. 代码规范

### 3.1 ESLint 配置

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "warn",
    "react/react-in-jsx-scope": "off"
  }
}
```

### 3.2 Prettier 配置

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

### 3.3 TypeScript 配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
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
  "include": ["src"],
  "exclude": ["node_modules", "src-tauri"]
}
```

---

## 4. Git 规范

### 4.1 分支策略（单人开发）

| 分支 | 用途 | 命名 |
|------|------|------|
| `main` | 稳定版本 | — |
| `feat/*` | 功能开发 | `feat/portfolio-add` |
| `fix/*` | Bug 修复 | `fix/rule-trigger` |

### 4.2 Commit 规范

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
feat(portfolio): 添加股票搜索功能
fix(rule): 修复规则重复触发问题
refactor(store): 重构 zustand store 结构
```

---

## 5. 开发环境

### 5.1 启动命令

```bash
# 安装依赖
pnpm install

# 启动开发模式（同时启动 Vite + Tauri）
pnpm tauri dev

# 构建生产版本
pnpm tauri build
```

### 5.2 Tauri 配置（tauri.conf.json 关键配置）

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "Investment Monitor",
  "identifier": "com.investment-monitor.app",
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build",
    "devUrl": "http://localhost:1420",
    "frontendDist": "../dist"
  },
  "app": {
    "windows": [
      {
        "label": "main",
        "title": "Investment Monitor",
        "width": 800,
        "height": 600,
        "visible": false,
        "decorations": true,
        "resizable": true
      }
    ],
    "trayIcon": {
      "icon": "icons/tray-icon.png",
      "iconAsTemplate": true
    }
  },
  "plugins": {
    "notification": {},
    "store": {}
  },
  "bundle": {
    "active": true,
    "targets": "all"
  }
}
```

### 5.3 关键配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `windows[0].visible` | `false` | 主窗口默认隐藏，通过托盘图标控制显示 |
| `trayIcon.iconAsTemplate` | `true` | macOS 暗色模式下图标自动适配 |
| `plugins.notification` | `{}` | 启用系统通知插件 |
| `plugins.store` | `{}` | 启用 KV 存储插件 |

---

## 6. Vite 配置

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  // Tauri 需要固定端口
  server: {
    port: 1420,
    strictPort: true,
  },
  // Tauri 在 Windows 上使用固定路径
  clearScreen: false,
  envPrefix: ['VITE_', 'TAURI_'],
});
```

---

## 7. Zustand Store 基础模板

```typescript
// stores/createStore.ts
import { create } from 'zustand';
import { invoke } from '@tauri-apps/api/core';

// 通用 store 创建函数，封装 invoke 调用和错误处理
export function createCrudStore<T extends { id: string }>(
  commandPrefix: string,
) {
  return create<{
    items: T[];
    loading: boolean;
    error: string | null;
    fetch: () => Promise<void>;
    add: (item: Omit<T, 'id' | 'createdAt' | 'updatedAt'>) => Promise<void>;
    remove: (id: string) => Promise<void>;
  }>((set) => ({
    items: [],
    loading: false,
    error: null,

    fetch: async () => {
      set({ loading: true, error: null });
      try {
        const items = await invoke<T[]>(`get_${commandPrefix}s`);
        set({ items, loading: false });
      } catch (e) {
        set({ error: String(e), loading: false });
      }
    },

    add: async (item) => {
      const newItem = await invoke<T>(`add_${commandPrefix}`, item);
      set((state) => ({ items: [...state.items, newItem] }));
    },

    remove: async (id) => {
      await invoke(`remove_${commandPrefix}`, { id });
      set((state) => ({ items: state.items.filter((i) => i.id !== id) }));
    },
  }));
}
```

---

## 8. 环境变量

| 变量 | 用途 | 默认值 |
|------|------|--------|
| `VITE_DEFAULT_INTERVAL` | 默认轮询间隔（秒） | `30` |
| `VITE_TRADING_AM_START` | 上午交易开始时间 | `09:30` |
| `VITE_TRADING_AM_END` | 上午交易结束时间 | `11:30` |
| `VITE_TRADING_PM_START` | 下午交易开始时间 | `13:00` |
| `VITE_TRADING_PM_END` | 下午交易结束时间 | `15:00` |

> 环境变量仅用于开发时的可配置默认值，运行时设置通过 tauri-plugin-store 持久化。A 股交易时段逻辑在 `tradingTime.ts` 中实现。
