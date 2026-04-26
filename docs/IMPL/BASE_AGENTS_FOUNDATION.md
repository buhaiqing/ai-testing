# 基础Agent能力集标准化文档

| 属性 | 值 |
|-----|-----|
| **文档类型** | IMPL |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | 质量智能中台团队 |
| **关联文档** | [AGENTS.md](../../AGENTS.md), [MIGRATION_STRATEGY.md](./MIGRATION_STRATEGY.md) |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.0.0 | 2026-04-26 | 初始版本 | 质量智能中台团队 |

---

## 1. 概述

### 1.1 文档目的

本文档定义了**基础Agent能力集（Base Agent Foundation）**，这些基础Agent具备**无需额外开发和定义**即可被不同业务团队（特别是运维相关Agent）直接调用的通用能力。

### 1.2 基础Agent定位

```
┌─────────────────────────────────────────────────────────────────┐
│                    Agent能力分层架构                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    业务Agent层 (Business Agent)               │ │
│  │                                                             │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │ │
│  │  │ 异常检测    │ │ 根因分析    │ │ 容量规划    │          │ │
│  │  │ 业务Agent  │ │ 业务Agent  │ │ 业务Agent  │          │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    基础Agent层 (Base Agent) ⭐              │ │
│  │                                                             │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │ │
│  │  │ 知识检索 │ │ 日志分析 │ │ 指标查询 │ │ 报告生成 │         │ │
│  │  │ 基础Agent│ │ 基础Agent│ │ 基础Agent│ │ 基础Agent│         │ │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘         │ │
│  │                                                             │ │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         │ │
│  │  │ 告警通知 │ │ 实体识别 │ │ 文本摘要 │ │ 意图分类 │         │ │
│  │  │ 基础Agent│ │ 基础Agent│ │ 基础Agent│ │ 基础Agent│         │ │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘         │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│                              ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    平台服务层 (Platform Service)              │ │
│  │                                                             │ │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐   │ │
│  │  │ LLM服务   │ │ 知识图谱   │ │ 向量数据库 │ │ 消息队列  │   │ │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────┘   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 设计原则

| 原则 | 说明 |
|-----|------|
| **可复用性** | 同一基础Agent可被多个业务Agent调用 |
| **可组合性** | 基础Agent可自由组合形成复杂能力 |
| **标准化** | 统一接口规范、调用参数、返回格式 |
| **可观测性** | 调用链路可追踪，性能指标可观测 |
| **容错性** | 单点故障不影响整体系统 |

---

## 2. 基础Agent类型及其核心功能清单

### 2.1 基础Agent总览

| Agent名称 | 英文名 | 功能定位 | 被调用频率 |
|----------|-------|---------|-----------|
| **知识检索Agent** | KnowledgeRetrievalAgent | 语义检索、相似度匹配、RAG增强 | 极高 |
| **日志分析Agent** | LogAnalysisAgent | 日志解析、模板提取、异常模式识别 | 极高 |
| **指标查询Agent** | MetricQueryAgent | 时序指标查询、聚合统计、趋势分析 | 极高 |
| **报告生成Agent** | ReportGenerationAgent | 结构化报告生成、Markdown/PDF导出 | 高 |
| **告警通知Agent** | AlertNotificationAgent | 多渠道通知、去重聚合、升级策略 | 高 |
| **实体识别Agent** | EntityRecognitionAgent | 命名实体识别、关系抽取、属性提取 | 中 |
| **文本摘要Agent** | TextSummarizationAgent | 长文本压缩、关键信息提取 | 中 |
| **意图分类Agent** | IntentClassificationAgent | 用户意图识别、问题分类 | 中 |

### 2.2 知识检索Agent (KnowledgeRetrievalAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **语义检索** | 查询文本、Top-K | 相关文档列表及相似度 | 基于向量相似度 |
| **混合检索** | 查询文本、Top-K | 融合关键词+语义的结果 | BM25 + 向量检索 |
| **知识图谱检索** | 查询实体/关系 | 实体详情、关联路径 | 基于Neo4j |
| **历史案例检索** | 故障描述 | 相似历史案例 | 专用案例库 |
| **多跳推理检索** | 复杂查询 | 推理路径及结论 | 结合知识图谱 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **检索引擎** | Elasticsearch + Milvus混合 |
| **向量模型** | text2vec-base-chinese |
| **Top-K默认值** | 5 |
| **最大Top-K** | 50 |
| **单次检索延迟P99** | < 200ms |
| **支持语言** | 中文、英文 |
| **知识库容量** | 支持1亿级文档 |

#### 调用示例

```python
# 调用知识检索Agent
from qaas import KnowledgeRetrievalAgent

agent = KnowledgeRetrievalAgent()

result = agent.retrieve(
    query="MySQL连接池耗尽如何处理",
    top_k=5,
    retrieval_mode="hybrid",
    filters={
        "category": "operation_manual",
        "env": "production"
    }
)

# 返回结果
{
    "results": [
        {
            "id": "doc_001",
            "content": "MySQL连接池耗尽的处理步骤...",
            "similarity": 0.92,
            "source": "operation_manual"
        }
    ],
    "total": 5,
    "processing_time_ms": 125
}
```

---

### 2.3 日志分析Agent (LogAnalysisAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **日志解析** | 原始日志文本 | 结构化日志JSON | JSON/Grok正则 |
| **模板提取** | 多条相似日志 | 日志模板 | 自动聚类提取 |
| **异常模式识别** | 日志流 | 异常日志及类型 | 基于规则+ML |
| **日志聚合** | 日志列表 | 聚合统计报告 | 按模板/时间/级别 |
| **上下文关联** | 异常日志 | 前后相关日志 | 时间窗口内 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **解析格式** | JSON/Grok/正则/CSV |
| **模板库容量** | 10万级模板 |
| **单次解析延迟** | < 50ms |
| **批量处理能力** | 10万条/秒 |
| **日志保留** | 30天热数据 |

#### 调用示例

```python
from qaas import LogAnalysisAgent

agent = LogAnalysisAgent()

# 解析日志
parsed = agent.parse(
    raw_log='2026-04-26 10:30:00 ERROR [MySQL] Connection pool exhausted',
    format="grok",
    grok_pattern="%{TIMESTAMP} %{LOGLEVEL} %{DATA:service} %{MSG}"
)

# 提取模板
templates = agent.extract_templates(
    logs=["log1", "log2", "log3"],
    min_occurrences=10
)

# 识别异常模式
anomalies = agent.detect_anomalies(
    log_stream="recent_5m_logs",
    detection_mode="rule_ml_hybrid"
)
```

---

### 2.4 指标查询Agent (MetricQueryAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **单指标查询** | 指标名、时间范围 | 时序数据点 | Prometheus格式 |
| **多指标查询** | 指标列表、时间范围 | 多指标时序 | 批量查询 |
| **指标聚合** | 指标、聚合方式 | 聚合结果 | AVG/SUM/MAX/MIN |
| **同比环比** | 指标、对比维度 | 变化率分析 | 日/周/月对比 |
| **趋势分析** | 指标、历史窗口 | 趋势报告 | 上升/下降/平稳 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **数据源** | Prometheus/InfluxDB |
| **查询延迟P99** | < 100ms |
| **最大时间范围** | 90天 |
| **最大数据点数** | 1万点/次 |
| **支持指标数** | 100万+ |

#### 调用示例

```python
from qaas import MetricQueryAgent

agent = MetricQueryAgent()

# 查询单个指标
result = agent.query(
    metric_name="cpu_usage",
    labels={"host": "server-01", "env": "prod"},
    start_time="2026-04-26T00:00:00Z",
    end_time="2026-04-26T12:00:00Z",
    step="60s"
)

# 同比分析
comparison = agent.compare(
    metric_name="order_qps",
    current_period=("2026-04-25", "2026-04-26"),
    compare_period=("2026-04-18", "2026-04-19"),
    group_by="hour"
)
```

---

### 2.5 报告生成Agent (ReportGenerationAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **结构化报告生成** | 数据JSON、模板 | Markdown/PDF | 模板渲染 |
| **故障复盘报告** | 故障ID | RCA报告 | 自动生成初稿 |
| **巡检报告** | 巡检配置 | 巡检报告 | 周期性生成 |
| **自定义报告** | SQL/查询+模板 | 定制报告 | 支持拖拽配置 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **模板引擎** | Jinja2 |
| **导出格式** | Markdown/PDF/HTML |
| **生成延迟P99** | < 5s |
| **支持图表** | ECharts/Mermaid |
| **模板数量** | 20+预设模板 |

#### 调用示例

```python
from qaas import ReportGenerationAgent

agent = ReportGenerationAgent()

# 生成故障复盘报告
report = agent.generate(
    template="rca_report",
    data={
        "fault_id": "fault_20260426_001",
        "title": "订单服务响应超时",
        "root_cause": "MySQL连接池耗尽",
        "timeline": [...],
        "action_items": [...]
    },
    output_format="markdown"
)

# 导出PDF
pdf_url = agent.export_pdf(
    report_id="rpt_001",
    template="incident_report"
)
```

---

### 2.6 告警通知Agent (AlertNotificationAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **多渠道通知** | 告警内容、渠道列表 | 发送结果 | 钉钉/短信/邮件 |
| **告警去重** | 告警列表 | 去重后告警 | 时间窗口去重 |
| **告警聚合** | 相似告警 | 聚合告警 | 根因聚合 |
| **升级策略** | 告警+升级规则 | 升级后告警 | 多级升级 |
| **通知排班** | 值班表 | 通知目标 | 自动路由 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **渠道** | 钉钉、短信、邮件、电话 |
| **去重窗口** | 5分钟（可配置） |
| **聚合能力** | 1000条/秒 |
| **发送延迟** | < 1s |
| **送达率** | > 99.9% |

#### 调用示例

```python
from qaas import AlertNotificationAgent

agent = AlertNotificationAgent()

# 发送告警
result = agent.notify(
    alert={
        "title": "CPU使用率过高",
        "severity": "P1",
        "content": "server-01 CPU使用率达到95%",
        "source": "anomaly_detection"
    },
    channels=["dingtalk", "sms"],
    receivers=["oncall_team", "ops_lead"]
)

# 去重处理
deduped = agent.deduplicate(
    alerts=[
        {"id": "alert_001", "signature": "cpu_high"},
        {"id": "alert_002", "signature": "cpu_high"}
    ],
    window_minutes=5
)
```

---

### 2.7 实体识别Agent (EntityRecognitionAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **命名实体识别** | 文本 | 实体列表及类型 | 人名/地名/机构等 |
| **关系抽取** | 文本 | 关系三元组 | 主谓宾关系 |
| **属性提取** | 实体 | 属性键值对 | 实体画像 |
| **运维实体识别** | 运维文本 | 运维实体 | 服务/主机/组件 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **模型** | BERT-based NER |
| **实体类型** | 50+预定义类型 |
| **识别精度** | F1 > 90% |
| **延迟P99** | < 100ms |
| **支持语言** | 中文、英文 |

#### 调用示例

```python
from qaas import EntityRecognitionAgent

agent = EntityRecognitionAgent()

# 识别运维实体
entities = agent.recognize(
    text="订单服务部署在杭州机房的server-01和server-02上",
    entity_types=["service", "location", "host"]
)
# 输出: [{"实体": "订单服务", "类型": "service"}, {"实体": "杭州机房", "类型": "location"}, ...]

# 关系抽取
relations = agent.extract_relations(
    text="order-service调用了payment-service",
    relations=["calls", "depends_on"]
)
```

---

### 2.8 文本摘要Agent (TextSummarizationAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **抽取式摘要** | 长文本 | 关键句子 | _TEXTRANK |
| **生成式摘要** | 长文本 | 压缩描述 | LLM生成 |
| **关键词提取** | 文本 | 关键词列表 | TF-IDF + LLM |
| **日志摘要** | 日志列表 | 日志总结 | 异常日志专用 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **输入长度** | 最大10万字符 |
| **摘要长度** | 原文10-30% |
| **延迟P99** | < 2s |
| **压缩比** | 3-10x |

#### 调用示例

```python
from qaas import TextSummarizationAgent

agent = TextSummarizationAgent()

# 生成摘要
summary = agent.summarize(
    text="这是一段很长的文本...",
    mode="extractive",
    max_length=200
)

# 提取关键词
keywords = agent.extract_keywords(
    text="MySQL连接池配置优化...",
    top_k=10
)
```

---

### 2.9 意图分类Agent (IntentClassificationAgent)

#### 核心功能

| 功能点 | 输入 | 输出 | 说明 |
|-------|------|------|------|
| **意图识别** | 用户查询文本 | 意图类别及置信度 | 多分类 |
| **槽位提取** | 查询文本+意图 | 参数键值对 | 语义槽位 |
| **意图澄清** | 模糊查询 | 澄清问题 | 多意图时触发 |
| **意图演变追踪** | 对话历史 | 意图转移 | 上下文理解 |

#### 技术规格

| 规格 | 值 |
|-----|-----|
| **预定义意图数** | 100+ |
| **意图准确率** | > 92% |
| **延迟P99** | < 50ms |
| **多轮支持** | 最多10轮 |

#### 调用示例

```python
from qaas import IntentClassificationAgent

agent = IntentClassificationAgent()

# 识别意图
intent = agent.classify(
    text="帮我查一下订单服务的CPU使用情况",
    domain="operations"
)
# 输出: {"意图": "metric_query", "置信度": 0.95, "槽位": {"service": "订单服务", "metric": "cpu_usage"}}

# 澄清多意图
if intent.is_ambiguous:
    clarification = agent.clarify(intent)
```

---

## 3. 标准化接口规范

### 3.1 统一接口设计

#### 请求标准格式

```json
{
  "header": {
    "request_id": "req_20260426_001",
    "trace_id": "trace_abc123",
    "timestamp": "2026-04-26T10:30:00Z",
    "caller": "anomaly_detection_agent",
    "mode": "sync"
  },
  "body": {
    "action": "retrieve",
    "parameters": {}
  }
}
```

#### 响应标准格式

```json
{
  "header": {
    "request_id": "req_20260426_001",
    "status": "success",
    "timestamp": "2026-04-26T10:30:00Z",
    "processing_time_ms": 125
  },
  "body": {
    "result": {},
    "metadata": {
      "agent_version": "v1.2.0",
      "model_version": "v2.1.0"
    }
  }
}
```

#### 错误响应格式

```json
{
  "header": {
    "request_id": "req_20260426_001",
    "status": "error",
    "error_code": "AGENT_001",
    "error_message": "知识库查询超时",
    "timestamp": "2026-04-26T10:30:00Z"
  },
  "body": null
}
```

### 3.2 接口版本管理

| 版本 | 发布时间 | 主要变更 |
|-----|---------|---------|
| v1.0.0 | 2026-04-26 | 初始版本 |

### 3.3 调用约束

| 约束项 | 限制值 | 说明 |
|-------|--------|------|
| **请求超时** | 30秒 | 超过则返回超时错误 |
| **请求体大小** | 10MB | 超过则拒绝 |
| **QPS限制** | 100次/秒 | 单Agent |
| **并发限制** | 50并发 | 单Agent |
| **重试次数** | 3次 | 指数退避 |

---

## 4. 性能指标、安全要求和可靠性保障

### 4.1 性能指标要求

| Agent类型 | 延迟P50 | 延迟P99 | QPS | 可用性 |
|----------|---------|---------|-----|--------|
| 知识检索Agent | < 100ms | < 300ms | 100 | >= 99.9% |
| 日志分析Agent | < 30ms | < 100ms | 1000 | >= 99.9% |
| 指标查询Agent | < 50ms | < 150ms | 500 | >= 99.95% |
| 报告生成Agent | < 2s | < 5s | 10 | >= 99.5% |
| 告警通知Agent | < 500ms | < 1s | 100 | >= 99.9% |
| 实体识别Agent | < 30ms | < 100ms | 200 | >= 99.9% |
| 文本摘要Agent | < 500ms | < 2s | 50 | >= 99.5% |
| 意图分类Agent | < 20ms | < 50ms | 500 | >= 99.95% |

### 4.2 安全要求

| 安全类别 | 要求 | 实现方式 |
|---------|------|---------|
| **传输安全** | TLS 1.3加密 | 全链路HTTPS |
| **认证授权** | API Key + RBAC | 统一认证服务 |
| **数据脱敏** | 敏感信息脱敏 | 请求/响应自动脱敏 |
| **审计日志** | 全操作审计 | 日志保留1年 |
| **限流熔断** | 防DDoS/过载 | 令牌桶+熔断器 |
| **数据隔离** | 租户数据隔离 | 独立存储命名空间 |

### 4.3 可靠性保障措施

| 措施 | 说明 | 生效时间 |
|-----|------|---------|
| **多副本部署** | 每个Agent至少2副本 | 立即 |
| **自动故障转移** | 副本故障自动切换 | < 30s |
| **熔断器** | 下游故障时快速失败 | < 100ms |
| **限流保护** | 超出QPS限制时排队/拒绝 | 立即 |
| **降级策略** | 服务不可用时降级到基础版本 | < 1s |
| **健康检查** | 定期健康检查，故障自动隔离 | 每10s |

### 4.4 监控指标

| 指标类别 | 具体指标 | 告警阈值 |
|---------|---------|---------|
| **可用性** | Agent可用率 | < 99.5% |
| **延迟** | P99响应时间 | > 阈值1.5倍 |
| **错误率** | 5xx错误率 | > 1% |
| **队列** | 待处理请求数 | > 1000 |
| **资源** | CPU/内存使用率 | > 80% |

---

## 5. 基础Agent与业务Agent集成方式

### 5.1 集成架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    基础Agent与业务Agent集成架构                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  业务Agent层                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  异常检测Agent ──▶ 根因分析Agent ──▶ 修复执行Agent      │   │
│  │       │               │                                │   │
│  │       │               │                                │   │
│  │       ▼               ▼                                │   │
│  │  ┌─────────────────────────────────────────────────┐  │   │
│  │  │              基础Agent调用                         │  │   │
│  │  │                                                  │  │   │
│  │  │   ┌─────────┐ ┌─────────┐ ┌─────────┐          │  │   │
│  │  │   │知识检索 │ │ 日志分析 │ │ 指标查询 │          │  │   │
│  │  │   │  Agent  │ │  Agent  │ │  Agent  │          │  │   │
│  │  │   └────┬────┘ └────┬────┘ └────┬────┘          │  │   │
│  │  │        │            │            │                │  │   │
│  │  └────────┼────────────┼────────────┼────────────────┘  │   │
│  └──────────┼────────────┼────────────┼────────────────────┘   │
│             │            │            │                          │
│             └────────────┼────────────┘                          │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Agent网关 (Agent Gateway)              │   │
│  │                                                         │   │
│  │   ├── 负载均衡                                           │   │
│  │   ├── 限流熔断                                           │   │
│  │   ├── 路由分发                                           │   │
│  │   ├── 监控埋点                                           │   │
│  │   └── 统一认证                                           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 数据交互标准

#### 5.2.1 同步调用模式

```
业务Agent ──▶ [Agent网关] ──▶ 基础Agent
                    │
                    └──▶ 响应: < 30s
```

| 调用模式 | 适用场景 | 超时设置 |
|---------|---------|---------|
| **同步调用** | 需要即时结果 | 30s |
| **异步回调** | 长时间处理 | 5min |
| **轮询获取** | 任务队列 | 30s轮询间隔 |

#### 5.2.2 数据格式标准

| 数据类型 | 格式 | 编码 |
|---------|------|------|
| **结构化数据** | JSON | UTF-8 |
| **半结构化数据** | JSON/JSONB | UTF-8 |
| **文本数据** | String | UTF-8 |
| **时间数据** | ISO8601 | UTC |
| **大对象** | Base64/URL | - |

#### 5.2.3 数据流规范

```
┌─────────────────────────────────────────────────────────────────┐
│                    数据流标准格式                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  输入数据流:                                                     │
│  {                                                               │
│    "data_type": "metric" | "log" | "alert" | "text",          │
│    "data": { ... },                                             │
│    "metadata": {                                                │
│      "source": "prometheus",                                     │
│      "timestamp": "2026-04-26T10:30:00Z",                     │
│      "labels": {}                                                │
│    }                                                             │
│  }                                                               │
│                                                                 │
│  输出数据流:                                                     │
│  {                                                               │
│    "result_type": "structured" | "raw" | "stream",             │
│    "result": { ... },                                           │
│    "processing_info": {                                         │
│      "agent": "knowledge_retrieval",                            │
│      "version": "v1.2.0",                                       │
│      "processing_time_ms": 125                                 │
│    }                                                             │
│  }                                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 服务发现与注册

| 机制 | 实现 | 说明 |
|-----|------|------|
| **服务注册** | Consul/Nacos | 基础Agent启动时注册 |
| **服务发现** | DNS + SDK | 业务Agent通过SDK发现 |
| **健康检查** | 主动探测 | 每10s检查一次 |
| **元数据** | 版本/标签/权重 | 路由策略依据 |

---

## 6. 文档化要求

### 6.1 必需文档清单

| 文档类型 | 必需 | 说明 |
|---------|------|------|
| **API接口文档** | ✅ | OpenAPI 3.0规范 |
| **SDK使用文档** | ✅ | Python/Java/Go多语言 |
| **部署文档** | ✅ | Kubernetes Helm Chart |
| **运维手册** | ✅ | 监控/告警/故障处理 |
| **示例代码** | ✅ | Jupyter Notebook |
| **变更日志** | ✅ | CHANGELOG.md |

### 6.2 接口文档模板

```yaml
# OpenAPI 3.0 示例
openapi: 3.0.0
info:
  title: 知识检索Agent API
  version: v1.0.0
  description: |
    基础Agent - 知识检索能力
    用于语义检索、相似度匹配、RAG增强
paths:
  /api/v1/knowledge/retrieve:
    post:
      summary: 知识检索
      operationId: retrieve
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/RetrieveRequest'
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/RetrieveResponse'
```

### 6.3 使用限制条件

| Agent | 限制条件 | 超出处理 |
|-------|---------|---------|
| 知识检索Agent | 单次最多返回50条 | 分页获取 |
| 日志分析Agent | 单次最多处理10万条 | 批量分片 |
| 指标查询Agent | 时间范围最大90天 | 缩小范围 |
| 报告生成Agent | 输入JSON最大10MB | 精简数据 |
| 告警通知Agent | 单次最多50个接收人 | 分批发送 |
| 实体识别Agent | 单次最多5000字符 | 分段处理 |

### 6.4 示例代码要求

```python
# 每个基础Agent必须提供以下示例:

# 1. 基础调用示例
example_basic = """
from qaas import KnowledgeRetrievalAgent

agent = KnowledgeRetrievalAgent()
result = agent.retrieve(query="MySQL优化", top_k=5)
"""

# 2. 带错误处理的示例
example_error_handling = """
try:
    result = agent.retrieve(query="MySQL优化", top_k=5)
except AgentTimeoutError:
    print("请求超时，使用降级方案")
except AgentUnavailableError:
    print("服务不可用，切换到备用Agent")
"""

# 3. 异步调用示例
example_async = """
import asyncio
from qaas import KnowledgeRetrievalAgent

async def main():
    agent = KnowledgeRetrievalAgent()
    result = await agent.aretrieve(query="MySQL优化")
"""
```

---

## 7. 典型应用案例

### 7.1 运维场景案例

#### 案例1：异常检测 + 根因分析

```
┌─────────────────────────────────────────────────────────────────┐
│                    异常检测 + 根因分析 协作案例                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: 异常检测Agent检测到异常                                  │
│  └── 调用: 指标查询Agent → 获取异常时刻的指标数据                 │
│                                                                 │
│  Step 2: 异常分类与初步分析                                       │
│  └── 调用: 实体识别Agent → 识别异常涉及的实体                     │
│  └── 调用: 日志分析Agent → 获取异常前后日志                       │
│                                                                 │
│  Step 3: 根因分析                                                │
│  └── 调用: 知识检索Agent → 检索相似历史案例                      │
│  └── 调用: 意图分类Agent → 识别分析意图                          │
│                                                                 │
│  Step 4: 生成报告                                                │
│  └── 调用: 报告生成Agent → 生成RCA报告                           │
│                                                                 │
│  Step 5: 通知                                                    │
│  └── 调用: 告警通知Agent → 通知相关人员                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Python调用示例:**

```python
from qaas import (
    MetricQueryAgent,
    LogAnalysisAgent,
    EntityRecognitionAgent,
    KnowledgeRetrievalAgent,
    ReportGenerationAgent,
    AlertNotificationAgent
)

# 初始化所有Agent
metric_agent = MetricQueryAgent()
log_agent = LogAnalysisAgent()
entity_agent = EntityRecognitionAgent()
knowledge_agent = KnowledgeRetrievalAgent()
report_agent = ReportGenerationAgent()
notify_agent = AlertNotificationAgent()

# Step 1: 获取指标
metrics = metric_agent.query(
    metric_name="cpu_usage",
    labels={"host": "server-01"},
    start_time="2026-04-26T10:00:00Z",
    end_time="2026-04-26T10:30:00Z"
)

# Step 2: 分析日志
logs = log_agent.parse_batch(
    raw_logs=raw_logs,
    format="json"
)

# Step 3: 识别实体
entities = entity_agent.recognize(
    text=f"异常涉及: {','.join([e['name'] for e in entities])}",
    entity_types=["service", "host", "component"]
)

# Step 4: 知识检索
similar_cases = knowledge_agent.retrieve(
    query=f"{alert_title} {alert_content}",
    top_k=3,
    filters={"type": "rca_case"}
)

# Step 5: 生成报告
report = report_agent.generate(
    template="rca_report",
    data={
        "alert": alert,
        "metrics": metrics,
        "logs": logs,
        "entities": entities,
        "similar_cases": similar_cases
    }
)

# Step 6: 发送通知
notify_agent.notify(
    alert={
        "title": f"故障通知: {alert_title}",
        "severity": "P1",
        "content": report
    },
    channels=["dingtalk", "sms"],
    receivers=["ops_team", "oncall"]
)
```

#### 案例2：智能问答

```
┌─────────────────────────────────────────────────────────────────┐
│                    智能问答 协作案例                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户问: "订单服务响应慢如何排查？"                                │
│                                                                 │
│  Step 1: 意图分类                                                │
│  └── 调用: 意图分类Agent → 识别为"故障排查"                      │
│                                                                 │
│  Step 2: 实体识别                                                │
│  └── 调用: 实体识别Agent → 提取"订单服务"                        │
│                                                                 │
│  Step 3: 知识检索                                                │
│  └── 调用: 知识检索Agent → 检索相关运维文档                      │
│                                                                 │
│  Step 4: 日志分析                                                │
│  └── 调用: 日志分析Agent → 分析订单服务最新日志                  │
│                                                                 │
│  Step 5: 指标查询                                                │
│  └── 调用: 指标查询Agent → 查询订单服务性能指标                  │
│                                                                 │
│  Step 6: 生成回答                                                │
│  └── 调用: 文本摘要Agent → 汇总所有信息生成回答                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 案例3：巡检报告自动生成

```
┌─────────────────────────────────────────────────────────────────┐
│                    巡检报告自动生成 案例                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: 指标采集                                                │
│  └── 调用: 指标查询Agent → 批量查询各服务指标                    │
│                                                                 │
│  Step 2: 日志扫描                                                │
│  └── 调用: 日志分析Agent → 扫描ERROR/WARN日志                    │
│                                                                 │
│  Step 3: 告警统计                                                │
│  └── 调用: 告警通知Agent → 查询历史告警统计                      │
│                                                                 │
│  Step 4: 知识检索                                                │
│  └── 调用: 知识检索Agent → 检索近期重大事件                       │
│                                                                 │
│  Step 5: 生成报告                                                │
│  └── 调用: 报告生成Agent → 生成日报/周报/月报                    │
│                                                                 │
│  Step 6: 发送报告                                                │
│  └── 调用: 告警通知Agent → 发送邮件/钉钉                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 其他业务场景

| 场景 | 基础Agent组合 | 说明 |
|-----|-------------|------|
| **故障自愈** | 指标查询+日志分析+告警通知+知识检索 | 自动判断+自动执行 |
| **容量规划** | 指标查询+趋势分析+报告生成 | 历史数据+预测 |
| **安全审计** | 日志分析+实体识别+意图分类 | 日志异常+威胁识别 |
| **性能优化** | 指标查询+日志分析+知识检索+报告生成 | 根因+建议 |
| **变更评估** | 指标查询+日志分析+知识检索 | 变更前/后对比 |

---

## 8. 附录

### 8.1 术语表

| 术语 | 定义 |
|-----|------|
| **基础Agent** | 具备通用能力、可被复用的最小Agent单元 |
| **业务Agent** | 基于基础Agent构建的垂直业务Agent |
| **Agent网关** | 统一接入层，负责负载均衡、限流、监控 |
| **RAG** | Retrieval Augmented Generation，检索增强生成 |

### 8.2 参考文档

| 文档 | 路径 |
|-----|------|
| Agent总览 | [AGENTS.md](../../AGENTS.md) |
| 迁移策略 | [MIGRATION_STRATEGY.md](./MIGRATION_STRATEGY.md) |
| 异常检测Agent需求 | [AGENTS/ANOMALY_DETECTION_AGENT_REQUIREMENTS.md](./AGENTS/ANOMALY_DETECTION_AGENT_REQUIREMENTS.md) |
| 根因分析Agent需求 | [AGENTS/ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md](./AGENTS/ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md) |
| 影子模式设计 | [AGENTS/AGENT_SHADOW_MODE_DESIGN.md](./AGENTS/AGENT_SHADOW_MODE_DESIGN.md) |

### 8.3 反馈与建议

如有疑问或建议，请联系质量智能中台团队。

| 联系方式 | 说明 |
|---------|------|
| Slack | #quality-platform |
| 邮件 | quality-platform@example.com |
| 文档 | [内部Wiki](https://wiki.example.com/quality-platform) |
