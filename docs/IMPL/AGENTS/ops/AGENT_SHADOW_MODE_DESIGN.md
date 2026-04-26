# 运维Agent通用影子模式设计规范

| 属性 | 值 |
|-----|-----|
| **文档类型** | IMPL |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | 质量智能中台团队 |
| **关联文档** | [AGENTS.md](../../../AGENTS.md), [MIGRATION_STRATEGY.md](../../MIGRATION_STRATEGY.md) |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.0.0 | 2026-04-26 | 初始版本 | 质量智能中台团队 |

---

## 1. 概述

### 1.1 影子模式定义

> **影子模式（Shadow Mode）** 是一种**零风险AI验证策略**，其核心原理是：在不改变现有业务流程和不影响用户的前提下，将AI Agent与传统流程进行**并行部署**，AI处理结果**仅用于内部对比分析**，不返回给用户。

### 1.2 文档目的

本规范定义了质量智能中台所有运维Agent应遵循的**通用影子模式设计标准**，包括：
- 数据存储架构
- 对比分析机制
- 准入/退出标准
- 监控指标体系

各具体Agent的影子模式实现应基于本规范进行扩展。

### 1.3 适用范围

| Agent类型 | 是否适用 | 影子模式周期 |
|----------|---------|------------|
| 异常检测Agent | 是 | 4周 |
| 根因分析Agent | 是 | 4周 |
| 告警收敛Agent | 是 | 3周 |
| 容量规划Agent | 是 | 3周 |
| 日志分析Agent | 是 | 3周 |
| 变更管理Agent | 是 | 4周 |
| 故障预测Agent | 是 | 4周 |
| 知识推荐Agent | 是 | 2周 |

---

## 2. 数据存储架构

### 2.1 双路数据存储设计

```
┌─────────────────────────────────────────────────────────────────┐
│                    影子模式数据存储架构                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   用户请求 ──▶ [流量镜像] ──┬──▶ 传统流程 ──▶ 存储【结果A】    │
│                            │                      ▲              │
│                            │                       │              │
│                            └──▶ AI Agent ──▶ 存储【结果B】    │
│                                               │                 │
│                                               ▼                 │
│                                        ┌─────────────┐          │
│                                        │   对比器     │          │
│                                        │ 比对A vs B  │          │
│                                        └─────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 存储内容对照表

| 存储项 | 内容 | 用途 | 保留策略 |
|-------|------|------|---------|
| **结果A（传统流程）** | 实际返回给用户的完整结果 | 对照基准 | 长期保留90天+ |
| **结果B（AI流程）** | AI并行处理的结果 | 与A对比 | 长期保留90天+ |
| **对比差异** | 准确率差、延迟差、覆盖率差 | 量化评估 | 聚合存储 |
| **原始请求** | 用户发送的完整请求体 | 复现/追溯 | 长期保留90天+ |
| **元数据** | 请求ID、时间戳、来源、用户标识(脱敏) | 关联分析 | 长期保留 |

### 2.3 存储分层设计

| 层级 | 存储介质 | 数据类型 | 保留周期 | 查询场景 |
|-----|---------|---------|---------|---------|
| **热层** | PostgreSQL (JSONB) | 对比结果、告警 | 7-30天 | 实时查询、异常追溯 |
| **温层** | ClickHouse | 聚合指标、场景分析 | 30-90天 | OLAP分析、报表 |
| **冷层** | 对象存储 (S3/OBS) + Parquet | 原始数据归档 | 90天-永久 | 模型训练、合规审计 |

### 2.4 数据写入策略

```python
# 异步写入，避免影响主流程
class ShadowStorageStrategy:

    def save_shadow_comparison(self, request_id: str, primary: Any, shadow: Any):
        # Stage 1: 同步写入对比结果（低延迟查询用）
        comparison_record = self._build_comparison_record(request_id, primary, shadow)
        self.postgres.insert(comparison_record)  # 同步，< 5ms

        # Stage 2: 异步写入原始数据（高吞吐）
        raw_record = self._build_raw_record(request_id, primary, shadow)
        self.message_queue.send("shadow_raw", raw_record)  # 异步

        # Stage 3: 触发对比分析（异步）
        self.message_queue.send("shadow_compare", {
            "request_id": request_id,
            "primary_id": primary_record_id,
            "shadow_id": shadow_record_id
        })
```

### 2.5 无基准场景的处理方案

#### 2.5.1 场景定义

以下情况不存在传统流程作为对照基准：

| 场景类型 | 说明 | 适用Agent |
|---------|------|----------|
| **无历史流程** | 公司从未有过自动化分析流程，均为人工分析 | 根因分析Agent |
| **规则引擎替代** | 传统流程为简单阈值规则，无法作为有效基准 | 异常检测Agent |
| **人工专家经验** | 依赖专家经验的场景，无结构化输出 | 告警收敛Agent |

#### 2.5.2 替代基准方案

```
┌─────────────────────────────────────────────────────────────────┐
│                    无基准场景下的对比策略                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  方案A: 基于历史标注数据的对比                                     │
│  ├── 利用已标注的历史故障案例作为Ground Truth                     │
│  ├── 定期人工标注新案例扩充标注库                                 │
│  └── 适用场景: 有历史数据积累的故障分析                           │
│                                                                 │
│  方案B: 多Agent交叉验证                                          │
│  ├── 不同Agent版本之间的结果对比                                  │
│  ├── 新版本 vs 稳定版本                                          │
│  └── 适用场景: 模型迭代升级验证                                   │
│                                                                 │
│  方案C: 基于规则的伪基准                                         │
│  ├── 用规则引擎生成"基线结果"进行对比                            │
│  ├── 适用于: 简单阈值可覆盖的场景                                │
│  └── 适用场景: 异常检测Agent的初步验证                           │
│                                                                 │
│  方案D: 人工抽检对比                                             │
│  ├── 定期人工复盘，抽样与AI结果对比                               │
│  ├── 按比例抽检(建议5%-10%)                                      │
│  └── 适用场景: 所有无基准场景的长期验证                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 2.5.3 各方案详细设计

**方案A: 历史标注数据对比**

```python
class HistoricalAnnotationComparator:
    """基于历史标注数据的对比器"""

    def __init__(self, annotation_db: AnnotationDatabase):
        self.annotation_db = annotation_db
        self.min_annotation_count = 100  # 最小标注样本量

    def compare_with_ground_truth(self, ai_result: Any, fault_id: str) -> ComparisonResult:
        # 1. 获取该故障的人工标注结果
        ground_truth = self.annotation_db.get_annotation(fault_id)

        if ground_truth is None:
            # 无标注，跳过对比但不报错
            return ComparisonResult(
                request_id=fault_id,
                has_ground_truth=False,
                accuracy_estimated=None
            )

        # 2. 计算与标注的一致性
        accuracy = self._calculate_annotation_match(ai_result, ground_truth)

        return ComparisonResult(
            request_id=fault_id,
            has_ground_truth=True,
            accuracy_estimated=accuracy,
            confidence=self._calculate_confidence(ground_truth)
        )

    def _calculate_confidence(self, annotation: Annotation) -> float:
        """根据标注质量计算置信度"""
        base_confidence = 0.8
        if annotation.is_expert_verified:
            base_confidence += 0.15
        if annotation.has_feedback:
            base_confidence += 0.05
        return min(base_confidence, 1.0)
```

**方案B: 多版本交叉验证**

```python
class MultiVersionComparator:
    """多Agent版本交叉验证"""

    def __init__(self, version_manager: VersionManager):
        self.version_manager = version_manager
        self.stable_version = version_manager.get_stable_version()

    def cross_validate(self, new_result: Any, stable_result: Any) -> ComparisonResult:
        # 计算新版本与稳定版本的一致率
        consistency_rate = self._calculate_consistency(new_result, stable_result)

        # 一致性阈值判断
        if consistency_rate >= 0.90:
            status = "pass"
        elif consistency_rate >= 0.80:
            status = "warning"
        else:
            status = "fail"

        return ComparisonResult(
            consistency_rate=consistency_rate,
            validation_status=status,
            stable_version=self.stable_version
        )
```

**方案C: 规则引擎伪基准**

```python
class RuleBasedBaselineComparator:
    """基于规则引擎的伪基准对比"""

    def __init__(self, rule_engine: RuleEngine):
        self.rule_engine = rule_engine

    def compare_with_rules(self, ai_result: Any) -> ComparisonResult:
        # 1. 用规则引擎生成基线结果
        baseline_result = self.rule_engine.analyze(ai_result.input_data)

        # 2. 对比AI结果与规则结果
        overlap_ratio = self._calculate_overlap(ai_result, baseline_result)

        # 3. 分析差异原因
        diff_analysis = self._analyze_diff_reason(ai_result, baseline_result)

        return ComparisonResult(
            baseline_type="rule_engine",
            overlap_ratio=overlap_ratio,
            diff_analysis=diff_analysis,
            is_consistent=(overlap_ratio >= 0.75)
        )
```

#### 2.5.4 准入标准计算方法

| 指标 | 有基准场景 | 无基准场景 | 测量方法 |
|-----|-----------|-----------|---------|
| **准确率** | AI结果与基准一致率 | 与人工标注一致率 | 人工复盘/总分析数 |
| **召回率** | 基准检出的/AI检出的 | 标注案例召回率 | 人工确认召回/总相关 |
| **假阳性率** | 错误判定/总判定 | 标注外判定/总判定 | 人工确认误判/总判定 |
| **Top-N命中率** | N=3候选含真实根因 | N=3候选含标注根因 | 含目标的比例 |

#### 2.5.5 混合基准模式

在实际部署中，可能同时存在有基准和无基准场景，应采用混合基准模式：

```python
class HybridBaselineStrategy:
    """混合基准策略"""

    def __init__(self):
        self.primary_comparator = None  # 有基准时使用
        self.fallback_comparators = []  # 无基准时的替代方案

    def select_comparator(self, scenario: str) -> Comparator:
        if scenario == "has_baseline":
            return self.primary_comparator
        elif scenario == "historical_annotation":
            return self.fallback_comparators[0]
        elif scenario == "multi_version":
            return self.fallback_comparators[1]
        elif scenario == "rule_baseline":
            return self.fallback_comparators[2]
        else:
            return self.fallback_comparators[3]  # 人工抽检

    def generate_comparison_report(self, results: List[ComparisonResult]) -> ShadowReport:
        """生成综合对比报告"""
        report = ShadowReport()

        # 合并所有对比结果
        for result in results:
            report.merge(result)

        # 计算综合指标
        report.overall_accuracy = self._calculate_weighted_accuracy(results)
        report.confidence_level = self._assess_confidence(results)

        return report
```

---

## 3. 对比分析机制

### 3.1 对比器设计

```python
@dataclass
class ComparisonResult:
    """对比结果"""
    request_id: str
    accuracy_diff: float       # 准确率差异 (-1 ~ 1)
    latency_diff_ratio: float  # 延迟差异倍数
    coverage_diff: float       # 覆盖率差异
    is_anomaly: bool           # 是否异常
    anomaly_reason: Optional[str]

class ResultComparator:
    """
    结果对比器 - 核心智能组件
    """

    def __init__(self, config: 'ComparisonConfig'):
        self.config = config
        self.threshold_accuracy_drop = 0.10   # 准确率下降10%触发告警
        self.threshold_latency_multiplier = 2.0  # 延迟2倍触发告警

    def compare(self, primary_result: Any, shadow_result: Any) -> ComparisonResult:
        # 计算各项差异指标
        accuracy_diff = self._calculate_accuracy_diff(primary_result, shadow_result)
        latency_diff_ratio = self._calculate_latency_diff(primary_result, shadow_result)
        coverage_diff = self._calculate_coverage_diff(primary_result, shadow_result)

        # 判断是否异常
        is_anomaly, anomaly_reason = self._detect_anomaly(
            accuracy_diff, latency_diff_ratio
        )

        return ComparisonResult(
            request_id=primary_result.request_id,
            accuracy_diff=accuracy_diff,
            latency_diff_ratio=latency_diff_ratio,
            coverage_diff=coverage_diff,
            is_anomaly=is_anomaly,
            anomaly_reason=anomaly_reason
        )
```

### 3.2 告警触发规则

| 告警级别 | 触发条件 | 动作 | 通知方式 |
|---------|---------|------|---------|
| **Critical** | 准确率下降 > 15% | 立即暂停AI流程 | 电话+短信+钉钉 |
| **Warning** | 准确率下降 > 10% | 记录告警，人工关注 | 钉钉 |
| **Info** | 准确率下降 > 5% | 记录日志，持续观察 | 邮件 |

---

## 4. 准入与退出标准

### 4.1 通用准入标准

| 指标 | 通用标准 | 说明 |
|-----|---------|------|
| **AI准确率** | > 85% | AI结果与标准结果的一致率 |
| **假阳性率** | < 10% | 错误判定为异常的比例 |
| **系统级Bug** | 0 | 阻塞性问题数 |
| **运行时间** | >= 2周 | 影子模式最小运行周期 |

### 4.2 Agent特定准入标准

| Agent类型 | 额外准入标准 | 影子周期 |
|----------|------------|---------|
| 异常检测Agent | 漏报率 < 5% | 4周 |
| 根因分析Agent | Top-3命中率 > 95% | 4周 |
| 告警收敛Agent | 收敛准确率 > 80% | 3周 |
| 容量规划Agent | 预测误差 < 15% | 3周 |
| 故障预测Agent | 预警准确率 > 75% | 4周 |
| 知识推荐Agent | 推荐采纳率 > 50% | 2周 |

### 4.3 退出标准

```
退出条件:
├── ✅ 准入标准全部满足
├── ✅ 影子模式运行 >= 最小周期
├── ✅ 无P1/P2级问题
└── ✅ 人工评审通过

退出流程:
1. 系统自动生成退出报告
2. 运维团队负责人审批
3. 灰度开启10%流量（金丝雀阶段）
4. 持续监控4周后全量
```

---

## 5. 监控指标体系

### 5.1 核心监控指标

| 指标类别 | 指标名称 | 计算方式 | 告警阈值 |
|---------|---------|---------|---------|
| **准确性** | 准确率 | 一致数/总数 | < 85% |
| **准确性** | 召回率 | 检出数/应检出数 | < 90% |
| **准确性** | F1-Score | 2P*R/(P+R) | < 85% |
| **性能** | 检测延迟P99 | 99分位延迟 | > 2x传统 |
| **稳定性** | 可用性 | 运行时间/总时间 | < 99% |
| **稳定性** | 数据完整率 | 成功数/总数 | < 99.5% |

### 5.2 Dashboard设计

```
┌─────────────────────────────────────────────────────────────────┐
│                    影子模式监控Dashboard                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │
│  │ 准确率趋势   │ │ 延迟对比    │ │ 异常率趋势  │              │
│  │ [折线图]    │ │ [对比图]    │ │ [折线图]    │              │
│  └─────────────┘ └─────────────┘ └─────────────┘              │
│                                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐              │
│  │ 流量分布    │ │ 资源消耗    │ │ 对比数据量  │              │
│  │ [饼图]      │ │ [柱状图]    │ │ [数字卡片]  │              │
│  └─────────────┘ └─────────────┘ └─────────────┘              │
│                                                                 │
│  最近告警:                                                     │
│  ├── 14:30 - 异常检测准确率下降至82% (< 85%)                  │
│  ├── 14:25 - 根因分析P99延迟超过2x传统                        │
│  └── 14:20 - 数据完整率降至99.2% (< 99.5%)                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. 数据对比表结构

### 6.1 PostgreSQL表结构

```sql
-- 影子模式对比结果表
CREATE TABLE shadow_comparisons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    comparison_id VARCHAR(64) UNIQUE NOT NULL,
    agent_type VARCHAR(32) NOT NULL,
    request_id VARCHAR(64) NOT NULL,
    detection_mode VARCHAR(16) NOT NULL,  -- 'shadow' or 'production'

    -- 对比结果
    primary_result JSONB NOT NULL,         -- 传统流程结果
    shadow_result JSONB NOT NULL,           -- AI流程结果
    comparison_result JSONB,                 -- 对比分析结果

    -- 差异指标
    accuracy_diff FLOAT,
    latency_diff_ratio FLOAT,
    coverage_diff FLOAT,

    -- 元数据
    is_anomaly BOOLEAN DEFAULT FALSE,
    anomaly_reason TEXT,
    severity VARCHAR(8),

    -- 时间戳
    created_at TIMESTAMPTZ DEFAULT NOW(),
    expires_at TIMESTAMPTZ,  -- TTL过期时间

    -- 索引
    INDEX idx_agent_type (agent_type),
    INDEX idx_created_at (created_at DESC),
    INDEX idx_is_anomaly (is_anomaly) WHERE is_anomaly = TRUE
) PARTITION BY RANGE (created_at);

-- 按月分区
CREATE TABLE shadow_comparisons_2026_04 PARTITION OF shadow_comparisons
    FOR VALUES FROM ('2026-04-01') TO ('2026-05-01');
```

### 6.2 ClickHouse聚合表结构

```sql
-- 影子模式聚合指标表 (ClickHouse)
CREATE TABLE shadow_metrics_agg (
    agent_type VARCHAR(32),
    date DATE,
    hour UINT8,

    -- 数量指标
    total_requests UInt64,
    anomaly_count UInt64,
    error_count UInt64,

    -- 准确性指标
    avg_accuracy_diff Float32,
    min_accuracy_diff Float32,
    max_accuracy_diff Float32,

    -- 性能指标
    avg_latency_ratio Float32,
    p50_latency_ratio Float32,
    p99_latency_ratio Float32,

    -- 数据质量
    avg_data_completeness Float32,

    INDEX idx_agent_date (agent_type, date)
) ENGINE = MergeTree()
PARTITION BY (agent_type, toYYYYMM(date))
ORDER BY (agent_type, date, hour);
```

---

## 7. API接口规范

### 7.1 影子模式配置API

```yaml
# 影子模式配置接口
/api/v1/shadow/config:
  GET:
    description: 获取影子模式配置
    response:
      200:
        body:
          {
            "enabled": true,
            "agent_type": "anomaly_detection",
            "traffic_split": {
              "primary": 1.0,
              "shadow": 1.0
            },
            "thresholds": {
              "accuracy_drop": 0.10,
              "latency_multiplier": 2.0
            },
            "retention_days": 90
          }

  PUT:
    description: 更新影子模式配置
    body:
      {
        "enabled": true,
        "thresholds": {
          "accuracy_drop": 0.15
        }
      }
```

### 7.2 影子模式统计API

```yaml
# 影子模式统计接口
/api/v1/shadow/stats:
  GET:
    description: 获取影子模式统计信息
    query_params:
      - agent_type: string
      - start_time: datetime
      - end_time: datetime
    response:
      200:
        body:
          {
            "agent_type": "anomaly_detection",
            "period": {
              "start": "2026-04-01T00:00:00Z",
              "end": "2026-04-26T00:00:00Z"
            },
            "metrics": {
              "total_requests": 1000000,
              "accuracy": 0.92,
              "recall": 0.95,
              "avg_latency_ratio": 1.15,
              "anomaly_rate": 0.05
            },
            "status": "healthy"
          }
```

---

## 8. 安全与合规

### 8.1 数据安全

| 安全措施 | 实现方式 |
|---------|---------|
| **传输加密** | TLS 1.3 |
| **存储加密** | AES-256 |
| **脱敏处理** | 用户标识、IP等敏感字段脱敏 |
| **权限控制** | RBAC + 操作审计 |

### 8.2 合规要求

| 合规项 | 要求 |
|-------|------|
| **数据保留** | 90天+保留，支持导出 |
| **审计日志** | 所有数据访问记录保留1年 |
| **数据删除** | 支持按请求删除个人数据 |
| **跨境传输** | 不跨境，数据存储境内 |

---

## 9. 故障处理

### 9.1 故障场景与处理

| 故障场景 | 影响 | 处理方式 | 恢复时间 |
|---------|------|---------|---------|
| Shadow Store不可用 | 对比数据丢失 | 降级：AI结果仍正常返回，仅停止对比 | < 5min |
| 对比器异常 | 无法计算差异 | 降级：记录原始结果，后续补分析 | < 10min |
| 数据积累延迟 | Dashboard数据滞后 | 降级：显示预估数据，标注"估算" | < 30min |

### 9.2 自动恢复机制

```yaml
auto_recovery:
  enabled: true

  strategies:
    - trigger: "shadow_store_unavailable"
      action: "disable_comparison"
      notification: "critical"

    - trigger: "anomaly_rate_spike"
      action: "reduce_traffic_to_50%"
      notification: "warning"

    - trigger: "data_loss_rate_above_1%"
      action: "pause_shadow_mode"
      notification: "critical"
```

---

## 10. 与各Agent的关联

| Agent需求文档 | 影子模式扩展说明 |
|--------------|----------------|
| [ANOMALY_DETECTION_AGENT_REQUIREMENTS.md](./ANOMALY_DETECTION_AGENT_REQUIREMENTS.md) | 异常检测特有的时序对比、多指标关联对比 |
| [ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md](./ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md) | 根因分析特有的人工标注流程、案例对比 |

---

## 11. 附录

### 11.1 术语表

| 术语 | 定义 |
|-----|------|
| Shadow Mode | 影子模式，AI与传统流程并行运行，结果仅内部对比 |
| Traffic Mirror | 流量镜像，将请求复制到多个处理路径 |
| Ground Truth | 标准答案/对照基准，此处指传统流程结果 |
| Comparator | 对比器，计算AI与传统流程结果差异的组件 |

### 11.2 参考资料

- [MIGRATION_STRATEGY.md](../../MIGRATION_STRATEGY.md) - 迁移策略详细设计
- [AGENTS.md](../../../AGENTS.md) - Agent矩阵定义
