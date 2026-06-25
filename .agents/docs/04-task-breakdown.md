# 04 - 任务拆解

---

## 迭代总览

| 阶段 | 时间 | 目标 | 交付物 |
|------|------|------|--------|
| M1: 项目搭建 | 第 1 周 | 基础框架可运行 | 工具栏图标 + Popup 空壳 |
| M2: 核心功能 | 第 2-3 周 | 完整 MVP | 可监测 + 可提醒 |
| M3: 收尾打磨 | 第 4 周 | 可发布版本 | 打包 + 测试 |

---

## M1: 项目搭建（第 1 周）

### T1.1 项目初始化
- [ ] `pnpm dlx wxt@latest init` 初始化项目（React + TypeScript 模板）
- [ ] 安装前端依赖（react, zustand, stock-sdk, js-yaml, lucide-react）
- [ ] 安装开发依赖（oxlint, oxfmt, typescript, @types/*）
- [ ] 配置 TypeScript、oxlint、oxfmt
- [ ] 配置路径别名 `@/`
- [ ] 验证 `pnpm dev` 可正常启动并加载扩展

### T1.2 浏览器工具栏与 Popup 骨架
- [ ] 配置 `wxt.config.ts` 的 manifest（permissions、host_permissions、icons、action）
- [ ] 设计 Popup 尺寸（380x600）
- [ ] Popup 主布局：顶部标题栏 + 内容区 + 底部操作栏 + 状态栏
- [ ] 操作栏按钮占位（添加、导出、导入、设置）
- [ ] 状态栏占位（监测状态、最后更新时间）
- [ ] 插件图标资源（16/32/48/128px）

### T1.3 Minimal Dark 样式基础
- [ ] 创建 `styles/tokens.css`（颜色、字体、圆角、过渡）
- [ ] 创建 `styles/global.css`（全局重置、Popup 尺寸、滚动条）
- [ ] 实现基础组件样式：`Button.module.css`、`Input.module.css`、`Card.module.css`
- [ ] 在 Popup 中应用基础主题

### T1.4 wxt/storage 封装骨架
- [ ] 创建 `storage/portfolioStorage.ts`（空实现 + 类型）
- [ ] 创建 `storage/ruleStorage.ts`
- [ ] 创建 `storage/settingsStorage.ts`
- [ ] 创建 `storage/marketDataStorage.ts`
- [ ] 创建 `types/portfolio.ts`、`types/rule.ts`、`types/marketData.ts`、`types/settings.ts`

### T1.5 Zustand Store 骨架
- [ ] 创建 `stores/portfolioStore.ts`（空 store + fetch/add/remove 方法签名）
- [ ] 创建 `stores/ruleStore.ts`
- [ ] 创建 `stores/marketDataStore.ts`
- [ ] 创建 `stores/settingsStore.ts`

---

## M2: 核心功能（第 2-3 周）

### T2.1 stock-sdk 封装
- [ ] 创建 `services/marketDataService.ts`
- [ ] 封装 `searchStocks(keyword)` — 调用 `sdk.search()`
- [ ] 封装 `fetchStockQuotes(codes)` — 调用 `sdk.quotes.cn()`
- [ ] 封装 `getDividendYield(code)` — 调用 `sdk.reference.dividendDetail()`
- [ ] 统一错误处理和重试逻辑
- [ ] 核实并补充 `wxt.config.ts` 中的 `host_permissions`

### T2.2 持仓管理
- [ ] 实现 `portfolioStorage.getAll()`
- [ ] 实现 `portfolioStorage.add()`（自动生成 id、时间戳、唯一性校验）
- [ ] 实现 `portfolioStorage.remove()`（级联删除关联规则）
- [ ] 实现 `portfolioStorage.import()`（覆盖写入）
- [ ] 前端：`AddStockModal` 组件（搜索 + 选择 + 确认）
- [ ] 前端：搜索结果展示（代码、名称、类型、市场）
- [ ] 前端：重复标的校验（code + market 唯一性）
- [ ] 前端：`PortfolioTable` 组件（原生 CSS 表格）
- [ ] 前端：表格列配置（代码、名称、实时价、涨跌幅、PE、PB、股息率）
- [ ] 前端：股票 vs ETF 列显示差异化（ETF 的 PE/PB/股息率显示 `-`）
- [ ] 前端：删除标的（二次确认弹窗）

### T2.3 数据监测
- [ ] 创建 `entrypoints/background.ts`
- [ ] 实现 `chrome.alarms` 调度器（交易时间每 30s）
- [ ] 交易时间判断工具函数（`tradingTime.ts`）
- [ ] A 股 / ETF 标的批量查询（统一调用 `sdk.quotes.cn`）
- [ ] 股息率单独查询（`sdk.reference.dividendDetail`）
- [ ] `marketDataStorage` 更新逻辑
- [ ] Popup 读取行情缓存并刷新表格
- [ ] 状态栏显示最后更新时间和监测状态
- [ ] 网络失败时插件图标 badge 变红

### T2.4 规则引擎
- [ ] 实现 `ruleStorage.getAll()`
- [ ] 实现 `ruleStorage.add()`
- [ ] 实现 `ruleStorage.update()`
- [ ] 实现 `ruleStorage.remove()`
- [ ] 实现 `ruleStorage.toggle()`
- [ ] 前端：`RuleForm` 组件（选择标的 → 选规则类型 → 填阈值）
- [ ] 前端：规则类型下拉（价格、PE、PB、股息率、涨跌幅）
- [ ] 前端：运算符下拉（<=, >=, <, >, ==）
- [ ] 前端：阈值输入（数字校验）
- [ ] 前端：自定义提醒消息
- [ ] 实现 `ruleMatcher.ts` 规则匹配逻辑（Popup 与 Service Worker 共享）
- [ ] Service Worker 中集成规则评估
- [ ] 单次触发 + 条件复位策略
- [ ] 规则开关（isActive 切换）

### T2.5 通知
- [ ] Service Worker 中实现 `showNotification()`
- [ ] 通知内容格式：标题 + 标的名 + 当前值 + 触发条件
- [ ] 规则匹配命中时自动触发通知
- [ ] 通知权限检查和提示
- [ ] 设置中关闭通知后跳过触发

### T2.6 导入导出
- [ ] 实现配置导出逻辑（读取 storage 中的持仓 + 规则，序列化为 YAML）
- [ ] 实现配置导入逻辑（解析 YAML，覆盖写入 storage）
- [ ] 前端：导出按钮逻辑（生成 YAML 文件并下载）
- [ ] 前端：导入按钮逻辑（文件选择 → 读取 → 解析）
- [ ] 前端：导入前确认弹窗（覆盖现有数据提示）
- [ ] 前端：导入错误处理（格式校验、提示信息）

### T2.7 设置
- [ ] 实现 `settingsStorage.get()`
- [ ] 实现 `settingsStorage.update()`
- [ ] 前端：`SettingsPanel` 组件
- [ ] 前端：更新频率下拉（15s / 30s / 60s）
- [ ] 前端：通知开关
- [ ] 前端：声音提醒开关（预留）
- [ ] 前端：设置保存后立即生效（Service Worker 下次轮询读取新设置）

---

## M3: 收尾打磨（第 4 周）

### T3.1 UI 优化
- [ ] 空状态处理（持仓为空时的占位提示）
- [ ] 加载状态处理（数据加载中的 Skeleton）
- [ ] 错误状态处理（网络失败的提示）
- [ ] 价格涨跌颜色（红涨绿跌）
- [ ] PE/PB/股息率数值格式化（保留小数位）
- [ ] Popup 尺寸和布局适配
- [ ] 按钮 hover/active 微交互
- [ ] 卡片 glass effect 和 amber glow

### T3.2 测试
- [ ] 手动测试：添加/删除标的完整流程
- [ ] 手动测试：规则配置 → 触发 → 通知完整流程
- [ ] 手动测试：导出 → 删除全部 → 导入恢复
- [ ] 手动测试：非交易时间行为（不轮询）
- [ ] 手动测试：网络断开时的表现
- [ ] 手动测试：Service Worker 唤醒和轮询
- [ ] 手动测试：Popup 打开/关闭数据一致性
- [ ] Chrome 测试
- [ ] Edge 测试

### T3.3 打包发布
- [ ] 配置应用图标（PNG 各尺寸）
- [ ] `pnpm build` 构建生产版本
- [ ] `pnpm zip` 打包扩展
- [ ] Chrome 加载已解压扩展验证
- [ ] Edge 加载已解压扩展验证
- [ ] 编写 README.md
- [ ] 准备 Chrome Web Store 发布物料（截图、描述）
