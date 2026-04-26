# 运维Agent模型热更新与可维护性设计规范

| 属性 | 值 |
|-----|-----|
| **文档类型** | IMPL |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | 质量智能中台团队 |
| **关联文档** | [ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md](./ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md), [ANOMALY_DETECTION_AGENT_REQUIREMENTS.md](./ANOMALY_DETECTION_AGENT_REQUIREMENTS.md), [CAPACITY_PLANNING_AND_COST_SPEC.md](./CAPACITY_PLANNING_AND_COST_SPEC.md) |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.0.0 | 2026-04-26 | 初始版本 | 质量智能中台团队 |

---

## 1. 概述

### 1.1 文档目的

本规范定义了质量智能中台运维Agent的**模型热更新和可维护性设计标准**，包括：
- 模型版本管理
- 热更新机制
- 灰度发布策略
- 配置管理
- 监控告警

### 1.2 设计原则

| 原则 | 说明 |
|-----|------|
| **零停机** | 模型更新不中断服务 |
| **可回滚** | 支持问题版本快速回滚 |
| **可观测** | 更新过程全程可监控 |
| **可灰度** | 支持流量渐进式切换 |
| **配置化** | 行为通过配置控制，不改代码 |

---

## 2. 模型版本管理

### 2.1 版本命名规范

```
{agent_type}_{model_type}_{version}_{YYYYMMDD}

示例：
rca_knowledge_graph_v1.8.0_20260426
rca_causal_inference_v2.1.0_20260426
anomaly_prophet_v2.1.0_20260426
anomaly_isolation_forest_v1.5.0_20260426
```

### 2.2 版本元数据

```json
{
  "model_id": "rca_knowledge_graph_v1.8.0_20260426",
  "agent_type": "rca",
  "model_type": "knowledge_graph",
  "version": "v1.8.0",
  "build_date": "2026-04-26",
  "base_model": "GraphSAGE",
  "framework": "PyTorch 2.0",
  "model_size_mb": 256,
  "input_schema": {
    "node_types": ["service", "component", "metric", "alert"],
    "edge_types": ["depends_on", "calls", "monitors"]
  },
  "output_schema": {
    "root_cause": "string",
    "confidence": "float",
    "reasoning_chain": ["string"]
  },
  "performance_metrics": {
    "accuracy": 0.92,
    "recall": 0.95,
    "latency_p99_ms": 150
  },
  "sla_requirements": {
    "max_loading_time_ms": 5000,
    "min_throughput_qps": 10
  },
  "dependencies": {
    "python_packages": ["torch==2.0.0", "dgl==1.0.0"],
    "system_libraries": ["libtorch"]
  },
  "deployment_config": {
    "replicas": 3,
    "resources": {
      "cpu": "4核",
      "memory": "8Gi",
      "gpu": "0"
    }
  },
  "validation_status": "passed",
  "canary_traffic_percent": 10
}
```

### 2.3 模型注册表

```python
class ModelRegistry:
    """模型注册表"""

    def __init__(self, storage: ModelStorage):
        self.storage = storage

    def register_model(self, model_metadata: ModelMetadata) -> str:
        """注册新模型"""
        model_id = model_metadata.model_id

        # 验证模型包完整性
        self._validate_model_package(model_id)

        # 验证模型性能指标
        self._validate_performance(model_metadata)

        # 注册到数据库
        self.db.models.insert(model_metadata)

        # 上传模型到存储
        self.storage.upload(model_id, model_metadata.model_path)

        return model_id

    def get_latest_version(self, agent_type: str, model_type: str) -> ModelMetadata:
        """获取最新版本"""
        return self.db.models.find_one(
            agent_type=agent_type,
            model_type=model_type,
            order_by="-build_date"
        )

    def get_stable_version(self, agent_type: str, model_type: str) -> ModelMetadata:
        """获取稳定版本（经过生产验证）"""
        return self.db.models.find_one(
            agent_type=agent_type,
            model_type=model_type,
            validation_status="production_verified",
            order_by="-build_date"
        )

    def list_versions(self, agent_type: str, model_type: str) -> List[ModelMetadata]:
        """列出所有版本"""
        return self.db.models.find(
            agent_type=agent_type,
            model_type=model_type,
            order_by="-build_date"
        )
```

---

## 3. 热更新机制

### 3.1 热更新原理

```
┌─────────────────────────────────────────────────────────────────┐
│                      模型热更新流程                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. 模型上传                                                    │
│     新模型文件上传到共享存储 / 对象存储                           │
│                                                                 │
│  2. 模型预热                                                    │
│     后台加载新模型到内存，验证预测正确性                          │
│                                                                 │
│  3. 流量切换                                                    │
│     通过配置中心下发路由规则，渐进式切换流量                      │
│                                                                 │
│  4. 旧模型卸载                                                  │
│     确认新模型稳定后，卸载旧模型释放内存                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 热更新实现

```python
class ModelHotUpdater:
    """模型热更新器"""

    def __init__(self, config_center: ConfigCenter):
        self.config_center = config_center
        self.current_model = None
        self.new_model = None

    async def hot_update(self, new_model_id: str, canary_percent: int = 10):
        """
        执行热更新

        Args:
            new_model_id: 新模型ID
            canary_percent: 金丝雀流量比例
        """
        # Step 1: 加载新模型
        logger.info(f"Loading new model: {new_model_id}")
        self.new_model = await self._load_model(new_model_id)

        # Step 2: 预热验证
        await self._warmup_model(self.new_model)
        validation_result = await self._validate_model(self.new_model)
        if not validation_result.is_valid:
            raise ModelValidationError(f"Model validation failed: {validation_result.errors}")

        # Step 3: 金丝雀发布
        logger.info(f"Starting canary deployment: {canary_percent}% traffic")
        await self._update_routing(canary_percent, new_model_id)

        # Step 4: 全量发布
        await self._full_deployment(new_model_id)

        # Step 5: 卸载旧模型
        await self._unload_old_model()

    async def _load_model(self, model_id: str) -> Model:
        """加载模型到内存"""
        model_path = self.storage.get_model_path(model_id)
        model = ModelLoader.load(model_path)
        return model

    async def _warmup_model(self, model: Model):
        """预热模型"""
        warmup_requests = self._generate_warmup_requests(count=100)
        for req in warmup_requests:
            await model.predict(req)
        logger.info("Model warmup completed")

    async def _validate_model(self, model: Model) -> ValidationResult:
        """验证模型正确性"""
        test_cases = self._load_test_cases()
        results = []
        for case in test_cases:
            pred = await model.predict(case.input)
            results.append(self._compare(pred, case.expected))

        accuracy = sum(results) / len(results)
        return ValidationResult(
            is_valid=accuracy >= 0.95,
            accuracy=accuracy,
            errors=[r for r in results if not r]
        )

    async def _update_routing(self, canary_percent: int, new_model_id: str):
        """更新路由配置"""
        config = {
            "model_routing": {
                "default": self.current_model.model_id,
                "canary": new_model_id,
                "canary_percent": canary_percent
            }
        }
        await self.config_center.update_config("model_routing", config)
        logger.info(f"Routing updated: canary_percent={canary_percent}%")
```

### 3.3 金丝雀发布策略

```yaml
canary_deployment:
  stages:
    - name: "初始验证"
      traffic_percent: 5
      duration_minutes: 10
      success_criteria:
        error_rate: < 1%
        latency_p99: < 500ms

    - name: "扩大验证"
      traffic_percent: 20
      duration_minutes: 30
      success_criteria:
        error_rate: < 0.5%
        latency_p99: < 300ms
        accuracy: > 90%

    - name: "全量发布"
      traffic_percent: 100
      duration_minutes: 60
      success_criteria:
        error_rate: < 0.1%
        latency_p99: < 200ms

  rollback:
    enabled: true
    trigger_conditions:
      error_rate_spike: > 5%
      latency_spike: > 2x baseline
      accuracy_drop: > 10%
    auto_rollback_percent: 100
```

---

## 4. 灰度发布策略

### 4.1 灰度发布流程

```
┌─────────────────────────────────────────────────────────────────┐
│                      灰度发布流程                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐                                               │
│  │  内部测试    │  5% 内部用户 + 自动化测试                     │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  灰度一     │  10% 生产流量 + 特定用户群                     │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  灰度二     │  30% 生产流量                                 │
│  └──────┬──────┘                                               │
│         │                                                       │
│         ▼                                                       │
│  ┌─────────────┐                                               │
│  │  全量发布   │  100% 流量                                   │
│  └─────────────┘                                               │
│                                                                 │
│  每阶段间隔: 4-24小时（根据问题发现情况调整）                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 灰度配置

```python
@dataclass
class CanaryConfig:
    """金丝雀配置"""
    model_id: str
    traffic_percent: int
    user_filter: Optional[UserFilter] = None
    region_filter: Optional[RegionFilter] = None
    duration_minutes: int = 60
    success_criteria: SuccessCriteria

@dataclass
class UserFilter:
    user_ids: List[str]  # 指定用户ID
    user_tags: List[str]  # 用户标签
    user_percent: float  # 用户百分比

@dataclass
class RegionFilter:
    allowed_regions: List[str]  # 允许的区域
    blocked_regions: List[str]  # 屏蔽的区域

@dataclass
class SuccessCriteria:
    max_error_rate: float = 0.01
    max_latency_p99_ms: float = 500
    min_accuracy: float = 0.90
    min_success_rate: float = 0.99
```

### 4.3 灰度监控

```yaml
canary_monitoring:
  metrics:
    - name: "canary_error_rate"
      query: "sum(rate(requests_total{canary='true'}[5m])) / sum(rate(requests_total{canary='true'}[5m]))"
      alert_threshold: 0.01

    - name: "canary_latency_p99"
      query: "histogram_quantile(0.99, sum(rate(request_duration_seconds_bucket{canary='true'}[5m])) by (le))"
      alert_threshold: 0.5

    - name: "canary_accuracy"
      query: "canary_correct_predictions / canary_total_predictions"
      alert_threshold: 0.90

    - name: "canary_vs_baseline_diff"
      query: "canary_error_rate - baseline_error_rate"
      alert_threshold: 0.005

  comparison:
    enabled: true
    baseline_version: "previous_stable_version"
    min_sample_size: 1000
    statistical_test: "chi_square"
```

---

## 5. 配置管理

### 5.1 配置分类

| 配置类型 | 存储位置 | 修改方式 | 热更新 |
|---------|---------|---------|-------|
| **模型配置** | 模型元数据库 | 重新部署 | 否 |
| **运行时配置** | 配置中心 | 动态下发 | 是 |
| **业务规则** | 配置中心 | 动态下发 | 是 |
| **阈值配置** | 配置中心 | 动态下发 | 是 |

### 5.2 运行时配置

```yaml
# 根因分析Agent运行时配置
rca_runtime_config:
  version: "v1.8.0"

  # 推理器配置
  inferencers:
    knowledge_graph:
      enabled: true
      weight: 0.35
      timeout_ms: 5000
      retry_count: 2

    causal_inference:
      enabled: true
      weight: 0.35
      timeout_ms: 3000
      retry_count: 2

    llm_reasoning:
      enabled: true
      weight: 0.30
      timeout_ms: 30000
      retry_count: 1
      model_name: "qwen-72b"

  # 置信度阈值
  confidence_thresholds:
    high: 0.85
    medium: 0.60
    low: 0.00

  # 数据收集配置
  data_collection:
    pre_fault_minutes: 120
    post_fault_minutes: 30
    max_alerts: 100
    alert_timeout_seconds: 30
    metric_timeout_seconds: 60

  # 降级策略配置
  degradation:
    enabled: true
    auto_degrade_on_llm_failure: true
    degrade_to_two_inferencers: true
    confidence_penalty_factor: 0.9
```

### 5.3 配置热更新

```python
class ConfigHotUpdater:
    """配置热更新器"""

    def __init__(self, config_center: ConfigCenter):
        self.config_center = config_center
        self.current_config = {}
        self.config_listeners = []

    async def update_config(self, key: str, value: Any):
        """动态更新配置"""
        old_value = self.current_config.get(key)

        # 验证配置变更
        self._validate_config_change(key, old_value, value)

        # 更新配置中心
        await self.config_center.update(key, value)

        # 通知所有监听器
        for listener in self.config_listeners:
            await listener.on_config_changed(key, old_value, value)

        # 记录审计日志
        self._log_config_change(key, old_value, value)

    def add_listener(self, listener: ConfigChangeListener):
        """添加配置变更监听器"""
        self.config_listeners.append(listener)

class ConfigChangeListener(ABC):
    """配置变更监听器基类"""

    @abstractmethod
    async def on_config_changed(self, key: str, old_value: Any, new_value: Any):
        pass
```

---

## 6. 监控与告警

### 6.1 模型监控指标

| 指标名称 | 类型 | 说明 | 告警阈值 |
|---------|-----|------|---------|
| `model_loading_time_seconds` | Gauge | 模型加载耗时 | > 30s |
| `model_prediction_latency_seconds` | Histogram | 模型预测延迟 | P99 > 2s |
| `model_prediction_errors_total` | Counter | 预测错误数 | 增长率 > 10% |
| `model_accuracy_current` | Gauge | 当前准确率 | < 0.85 |
| `model_version_mismatch_total` | Counter | 版本不匹配次数 | > 0 |
| `model_stale_predictions_total` | Counter | 过期模型预测次数 | > 0 |

### 6.2 灰度监控仪表板

```json
{
  "dashboard": "模型灰度发布监控",
  "panels": [
    {
      "title": "新旧模型对比 - 错误率",
      "type": "timeseries",
      "targets": [
        {
          "expr": "error_rate{version=\"new\"}",
          "legendFormat": "新模型"
        },
        {
          "expr": "error_rate{version=\"old\"}",
          "legendFormat": "旧模型"
        }
      ]
    },
    {
      "title": "新旧模型对比 - 延迟",
      "type": "timeseries",
      "targets": [
        {
          "expr": "latency_p99{version=\"new\"}",
          "legendFormat": "新模型 P99"
        },
        {
          "expr": "latency_p99{version=\"old\"}",
          "legendFormat": "旧模型 P99"
        }
      ]
    },
    {
      "title": "流量分布",
      "type": "piechart",
      "targets": [
        {
          "expr": "sum by (version) (requests_total)"
        }
      ]
    },
    {
      "title": "模型准确率对比",
      "type": "gauge",
      "targets": [
        {
          "expr": "accuracy{version=\"new\"}",
          "legendFormat": "新模型"
        },
        {
          "expr": "accuracy{version=\"old\"}",
          "legendFormat": "旧模型"
        }
      ]
    }
  ]
}
```

### 6.3 更新告警规则

```yaml
model_update_alerts:
  - name: "模型加载失败"
    condition: "model_loading_errors_total > 0"
    severity: "critical"
    action: "立即通知 + 自动回滚"

  - name: "新模型错误率飙升"
    condition: "error_rate{version='new'} > 3 * error_rate{version='old'}"
    severity: "critical"
    action: "立即通知 + 自动回滚"

  - name: "新模型延迟过高"
    condition: "latency_p99{version='new'} > 2 * latency_p99{version='old'}"
    severity: "warning"
    action: "通知 + 继续监控"

  - name: "流量切换异常"
    condition: "sum(requests_total{version='new'}) / sum(requests_total) > target_percent + 0.1"
    severity: "warning"
    action: "通知 + 检查配置"

  - name: "版本不一致"
    condition: "count(distinct(model_version)) > 2"
    severity: "warning"
    action: "通知 + 清理旧版本"
```

---

## 7. 回滚机制

### 7.1 回滚触发条件

```yaml
rollback:
  auto_rollback:
    enabled: true
    triggers:
      - name: "错误率超标"
        condition: "error_rate_new > 0.05"
        action: "立即回滚"

      - name: "延迟超标"
        condition: "latency_p99_new > latency_p99_old * 3"
        action: "立即回滚"

      - name: "准确率下降"
        condition: "accuracy_new < accuracy_old - 0.1"
        action: "立即回滚"

      - name: "服务不可用"
        condition: "model_health_check_failed == true"
        action: "立即回滚"

  manual_rollback:
    enabled: true
    approval_required: true
    timeout_minutes: 30
```

### 7.2 回滚流程

```python
class ModelRollback:
    """模型回滚器"""

    def __init__(self, config_center: ConfigCenter, model_registry: ModelRegistry):
        self.config_center = config_center
        self.model_registry = model_registry

    async def rollback_to_previous(self, reason: str):
        """回滚到上一版本"""
        previous_version = await self.model_registry.get_previous_version()

        logger.warning(f"Starting rollback to {previous_version.version}, reason: {reason}")

        # Step 1: 通知
        await self._send_rollback_notification(previous_version, reason)

        # Step 2: 切换流量到旧版本
        await self._switch_traffic_to(previous_version.model_id)

        # Step 3: 确认稳定性
        await self._confirm_stability(timeout_seconds=60)

        # Step 4: 清理新版本
        await self._cleanup_version(previous_version.successor_id)

        logger.info(f"Rollback completed: {previous_version.version}")

    async def rollback_to_specific(self, target_version: str, reason: str):
        """回滚到指定版本"""
        target = await self.model_registry.get_version(target_version)
        if not target:
            raise ValueError(f"Version not found: {target_version}")

        logger.warning(f"Rolling back to specific version: {target_version}, reason: {reason}")

        await self._switch_traffic_to(target.model_id)
        await self._confirm_stability(timeout_seconds=60)
        await self._cleanup_version(target.successor_id)
```

### 7.3 回滚后检查清单

```markdown
# 回滚后检查清单

## 1. 流量验证
- [ ] 100%流量切换到旧版本
- [ ] 旧版本错误率为零
- [ ] 旧版本延迟恢复正常

## 2. 监控确认
- [ ] 告警已消除
- [ ] 仪表板显示正常
- [ ] 无新的错误日志

## 3. 问题分析
- [ ] 已记录回滚原因
- [ ] 已创建问题跟踪
- [ ] 已通知相关人员

## 4. 后续行动
- [ ] 新版本修复计划
- [ ] 重新测试计划
- [ ] 回滚时间记录
```

---

## 8. 发布 Checklist

### 8.1 模型发布前检查

| 检查项 | 状态 | 说明 |
|-------|------|------|
| 模型单元测试通过 | □ | 准确率、召回率达标 |
| 模型集成测试通过 | □ | 与Agent服务集成正常 |
| 性能测试通过 | □ | 延迟、吞吐量达标 |
| 内存占用合理 | □ | 不超过配置限制 |
| 模型文件完整 | □ | MD5校验通过 |
| 文档更新 | □ | changelog、API文档 |
| 监控指标注册 | □ | 新指标已添加 |

### 8.2 灰度发布检查

| 检查项 | 状态 | 说明 |
|-------|------|------|
| 灰度流量配置正确 | □ | 百分比符合预期 |
| 用户筛选条件正确 | □ | 区域、标签筛选正确 |
| 监控仪表板就绪 | □ | 新旧对比视图正常 |
| 告警规则配置正确 | □ | 阈值设置合理 |
| 回滚预案准备就绪 | □ | 回滚流程已测试 |
| 值班人员已通知 | □ | 相关人员已到位 |

### 8.3 全量发布检查

| 检查项 | 状态 | 说明 |
|-------|------|------|
| 灰度阶段指标正常 | □ | 错误率、延迟达标 |
| 流量100%切换完成 | □ | 无版本不一致 |
| 旧模型已卸载 | □ | 资源已释放 |
| 旧模型保留策略 | □ | 保留最近N个版本 |
| 监控告警正常 | □ | 无异常告警 |
| 文档更新完成 | □ | 版本记录已更新 |
| 团队通知已发送 | □ | 发布通知已发送 |

---

## 9. 附录

### 9.1 版本兼容性矩阵

| Agent版本 | 最低API版本 | 最低SDK版本 | 最低Python版本 |
|----------|------------|------------|--------------|
| v1.8.x | v1 | 1.0.x | 3.8 |
| v1.9.x | v1 | 1.1.x | 3.8 |
| v2.0.x | v2 | 2.0.x | 3.9 |
| v2.1.x | v2 | 2.0.x | 3.9 |

### 9.2 术语表

| 术语 | 定义 |
|-----|------|
| **热更新** | 在不停止服务的情况下更新模型 |
| **金丝雀发布** | 将新版本逐步放量给少量用户 |
| **灰度发布** | 渐进式发布，先小流量后大流量 |
| **蓝绿部署** | 两套环境交替，新旧切换 |
| **回滚** | 发现问题后恢复到旧版本 |
| **预热** | 新模型上线前用真实请求验证 |
