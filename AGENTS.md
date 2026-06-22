# Investment Monitor 项目

## 技能系统

`.agents/skills` 目录包含一系列预定义的技能 (skills)，为 AI 提供结构化的工作流程和专业能力。

### 可用技能

| 技能 | 用途 |
|------|------|
| `belos-street` | 编码规范和最佳实践 |
| `brainstorming` | 创意设计和需求探索（必须在实现前使用） |
| `react-best-practices` | React 开发最佳实践 |
| `ui-templates` | UI 设计模板库 |
| `vibe-flow` | Web/SaaS 项目全流程管理 |
| `writing-plans` | 实施计划编写 |

### 工作流程

1. **构思阶段** - 使用 `brainstorming` 技能探索需求、设计方案
2. **规划阶段** - 使用 `writing-plans` 技能创建实施计划
3. **实现阶段** - 参考 `belos-street`、`react-best-practices` 等技能的规范进行开发

### 技能结构

每个技能目录包含：
- `skill.md` 或 `SKILL.md` - 技能定义和使用说明
- `reference/` - 参考文档（可选）
- `scripts/` - 辅助脚本（可选）