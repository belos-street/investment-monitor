# 04 - 任务拆解

---

## 迭代总览

| 阶段 | 时间 | 目标 | 交付物 |
|------|------|------|--------|
| M1: 项目搭建 | 第 1 周 | 基础框架可运行 | 托盘图标 + 空壳主窗口 |
| M2: 核心功能 | 第 2-3 周 | 完整 MVP | 可监测 + 可提醒 |
| M3: 收尾打磨 | 第 4 周 | 可发布版本 | 打包 + 测试 |

---

## M1: 项目搭建（第 1 周）

### T1.1 项目初始化
- [ ] `pnpm create tauri-app` 初始化项目
- [ ] 安装前端依赖（react, antd, zustand, stock-sdk, js-yaml）
- [ ] 安装 Tauri 插件（tauri-plugin-store, tauri-plugin-notification）
- [ ] 配置 TypeScript、ESLint、Prettier
- [ ] 配置 Vite 路径别名 `@/`
- [ ] 验证 `pnpm tauri dev` 可正常启动

### T1.2 系统托盘
- [ ] 配置 `tauri.conf.json` 的 `trayIcon`
- [ ] 主窗口默认隐藏（`visible: false`）
- [ ] 点击托盘图标显示/隐藏主窗口
- [ ] 右键托盘菜单（显示主窗口、退出）
- [ ] 窗口关闭时隐藏到托盘（不退出应用）

### T1.3 主窗口骨架
- [ ] 主窗口布局：顶部内容区 + 底部操作栏 + 状态栏
- [ ] 操作栏按钮占位（添加、导出、导入、设置）
- [ ] 状态栏占位（监测状态、最后更新时间）
- [ ] 窗口失焦时自动隐藏

### T1.4 Zustand Store 骨架
- [ ] 创建 `portfolioStore`（空 store + fetch/add/remove 方法签名）
- [ ] 创建 `ruleStore`
- [ ] 创建 `marketDataStore`
- [ ] 创建 `settingsStore`

---

## M2: 核心功能（第 2-3 周）

### T2.1 stock-sdk 封装
- [ ] 创建 `services/marketDataService.ts`
- [ ] 封装 `searchStocks(keyword)` — 调用 `sdk.search()`
- [ ] 封装 `fetchStockQuotes(codes)` — 调用 `sdk.quotes.cn()`
- [ ] 封装 `fetchFundQuotes(codes)` — 调用 `sdk.quotes.fund()`（NAV 参考，可选）
- [ ] 封装 `getDividendYield(code)` — 调用 `sdk.reference.dividendDetail()`
- [ ] 统一错误处理和重试逻辑

### T2.2 持仓管理
- [ ] Rust 端：实现 `get_portfolios` Command
- [ ] Rust 端：实现 `add_portfolio` Command
- [ ] Rust 端：实现 `remove_portfolio` Command
- [ ] 前端：`AddStockModal` 组件（搜索 + 选择 + 确认）
- [ ] 前端：搜索结果展示（代码、名称、类型、市场）
- [ ] 前端：重复标的校验（code + market 唯一性）
- [ ] 前端：`PortfolioTable` 组件（Ant Design Table）
- [ ] 前端：表格列配置（代码、名称、实时价、涨跌幅、PE、PB、股息率）
- [ ] 前端：股票 vs ETF 列显示差异化（ETF 的 PE/PB/股息率显示 `-`）
- [ ] 前端：删除标的（二次确认弹窗）

### T2.3 数据监测
- [ ] 创建 `hooks/useMarketData.ts`
- [ ] 实现轮询调度器（基于 `setInterval`）
- [ ] 交易时间判断工具函数（`tradingTime.ts`）
- [ ] A 股 / ETF 标的批量查询（统一调用 `sdk.quotes.cn`）
- [ ] 基金 NAV 参考查询（`sdk.quotes.fund`，可选）
- [ ] 股息率单独查询（`sdk.reference.dividendDetail`）
- [ ] `marketDataStore` 更新逻辑
- [ ] UI 数据刷新（表格实时更新）
- [ ] 状态栏显示最后更新时间和监测状态

### T2.4 规则引擎
- [ ] Rust 端：实现 `get_rules` Command
- [ ] Rust 端：实现 `add_rule` Command
- [ ] Rust 端：实现 `update_rule` Command
- [ ] Rust 端：实现 `remove_rule` Command
- [ ] 前端：`RuleForm` 组件（选择标的 → 选规则类型 → 填阈值）
- [ ] 前端：规则类型下拉（价格、PE、PB、股息率、涨跌幅）
- [ ] 前端：运算符下拉（<=, >=, <, >, ==）
- [ ] 前端：阈值输入（数字校验）
- [ ] 前端：自定义提醒消息
- [ ] 前端：`ruleMatcher.ts` 规则匹配逻辑
- [ ] 前端：单次触发 + 条件复位策略
- [ ] 前端：规则开关（isActive 切换）
- [ ] 前端：`useMarketData` 中集成规则评估

### T2.5 通知
- [ ] Rust 端：实现 `send_notification` Command（Tauri 原生通知）
- [ ] 前端：`notificationService.ts` 封装
- [ ] 通知内容格式：标题 + 标的名 + 当前值 + 触发条件
- [ ] 规则匹配命中时自动触发通知

### T2.6 导入导出
- [ ] Rust 端：实现 `export_config` Command（序列化为 YAML）
- [ ] Rust 端：实现 `import_config` Command（解析 YAML 写入 store）
- [ ] 前端：导出按钮逻辑（invoke → 保存文件对话框）
- [ ] 前端：导入按钮逻辑（文件选择 → 读取 → invoke）
- [ ] 前端：导入前确认弹窗（覆盖现有数据提示）
- [ ] 前端：导入错误处理（格式校验、提示信息）

### T2.7 设置
- [ ] Rust 端：实现 `get_settings` Command
- [ ] Rust 端：实现 `update_settings` Command
- [ ] 前端：`SettingsPanel` 组件
- [ ] 前端：更新频率下拉（15s / 30s / 60s）
- [ ] 前端：通知开关
- [ ] 前端：声音提醒开关
- [ ] 前端：设置保存后立即生效（轮询间隔动态调整）

---

## M3: 收尾打磨（第 4 周）

### T3.1 UI 优化
- [ ] 空状态处理（持仓为空时的占位提示）
- [ ] 加载状态处理（数据加载中的 Skeleton）
- [ ] 错误状态处理（网络失败的提示）
- [ ] 价格涨跌颜色（红涨绿跌 / 绿涨红跌可配置）
- [ ] PE/PB/股息率数值格式化（保留小数位）
- [ ] 窗口尺寸和布局适配

### T3.2 测试
- [ ] 手动测试：添加/删除标的完整流程
- [ ] 手动测试：规则配置 → 触发 → 通知完整流程
- [ ] 手动测试：导出 → 删除全部 → 导入恢复
- [ ] 手动测试：非交易时间行为（不轮询）
- [ ] 手动测试：网络断开时的表现
- [ ] 手动测试：托盘图标交互（点击、右键、窗口隐藏）
- [ ] macOS 测试
- [ ] Windows 测试

### T3.3 打包发布
- [ ] 配置应用图标（macOS .icns / Windows .ico）
- [ ] `pnpm tauri build` 构建生产版本
- [ ] macOS 安装包验证
- [ ] Windows 安装包验证（如有条件）
- [ ] 编写 README.md
