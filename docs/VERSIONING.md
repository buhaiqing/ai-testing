# 文档版本化管理规范

> **原则**：本文档定义项目所有文档的版本管理规则，确保版本可追溯、变更可管理。

| 属性 | 值 |
|-----|-----|
| **文档类型** | SPEC |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | Quality Agent 培养计划 |
| **关联文档** | AGENTS.md, docs/TERMINOLOGY.md |

---

## 版本号格式

采用**语义化版本控制**：`主版本.次版本.修订版本`

| 版本位 | 升级条件 | 示例 |
|-------|---------|------|
| **主版本** | 架构重大变更、核心概念重新定义、文档结构重组 | 1.0.0 → 2.0.0 |
| **次版本** | 新增章节、能力扩展、重要内容补充 | 1.1.0 → 1.2.0 |
| **修订版本** | 错别字修正、链接更新、格式调整、术语同步 | 1.1.0 → 1.1.1 |

## 文档类型标识

| 类型 | 标识 | 说明 | 代表文档 |
|-----|------|------|---------|
| **架构文档** | ARCH | 描述系统架构、组件职责、设计原则 | AGENTS.md, QUALITY_AGENT_ARCHITECTURE.md |
| **实现文档** | IMPL | 描述技术实现、代码示例、集成细节 | QUALITY_AGENT_EXECUTION.md, QUALITY_AGENT_TEST_GENERATOR.md |
| **计划文档** | PLAN | 描述培养计划、学习路径、时间安排 | 8-week-daily-task-list.md, week1-8-detailed-plan.md |
| **知识文档** | KNOW | 描述知识体系、行业分析、认证指南 | ai-agent-testing-knowledge-system.md, interview-guide.md |
| **规范文档** | SPEC | 描述规范标准、术语定义、管理规则 | TERMINOLOGY.md, VERSIONING.md |

## 文档头部标准格式

每个文档必须在头部包含以下信息：

```markdown
# 文档标题

| 属性 | 值 |
|-----|-----|
| **文档类型** | [ARCH/IMPL/PLAN/KNOW/SPEC] |
| **版本** | vX.Y.Z |
| **最后更新** | YYYY-MM-DD |
| **维护者** | Quality Agent 培养计划 |
| **关联文档** | [相关文档链接] |

---
```

## 变更历史记录

每个文档必须包含变更历史章节：

```markdown
## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.1.0 | 2026-04-26 | 新增XX章节 | - |
| v1.0.0 | 2026-04-23 | 初始版本 | - |
```

## 版本升级流程

1. **评估变更影响**：判断变更属于主版本/次版本/修订版本
2. **更新版本号**：在文档头部修改版本号
3. **记录变更历史**：在变更历史表中添加新记录
4. **同步关联文档**：如变更影响其他文档，需同步更新
5. **提交变更说明**：在提交时说明版本升级原因

## 当前文档版本清单

| 文档 | 类型 | 当前版本 | 最后更新 |
|-----|------|---------|---------|
| AGENTS.md | ARCH | v2.1.0 | 2026-04-26 |
| docs/TERMINOLOGY.md | SPEC | v1.0.0 | 2026-04-26 |
| docs/VERSIONING.md | SPEC | v1.0.0 | 2026-04-26 |
| docs/QUALITY_AGENT_ARCHITECTURE.md | ARCH | v2.0.0 | 2026-04-23 |
| docs/QUALITY_AGENT_EVOLUTION.md | ARCH | v2.0.0 | 2026-04-23 |
| docs/QUALITY_AGENT_EXECUTION.md | IMPL | v1.1.0 | 2026-04-23 |
| docs/QUALITY_AGENT_RISK_MANAGEMENT.md | IMPL | v1.0.0 | 2026-04-23 |
| docs/QUALITY_AGENT_TEST_GENERATOR.md | IMPL | v2.2.0 | 2026-04-23 |
| docs/ai-agent-testing-knowledge-system.md | KNOW | v1.0.0 | 2026-04-22 |
| docs/8-week-daily-task-list.md | PLAN | v1.0.0 | 2026-04-22 |

