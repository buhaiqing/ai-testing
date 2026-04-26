# 运维Agent错误码与异常处理规范

| 属性 | 值 |
|-----|-----|
| **文档类型** | IMPL |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | 质量智能中台团队 |
| **关联文档** | [ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md](./ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md), [ANOMALY_DETECTION_AGENT_REQUIREMENTS.md](./ANOMALY_DETECTION_AGENT_REQUIREMENTS.md), [AGENT_SHADOW_MODE_DESIGN.md](./AGENT_SHADOW_MODE_DESIGN.md) |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.0.0 | 2026-04-26 | 初始版本 | 质量智能中台团队 |

---

## 1. 概述

### 1.1 文档目的

本规范定义了质量智能中台所有运维Agent应遵循的**统一错误码和异常处理标准**，包括：
- 错误码体系设计
- 异常分类与处理策略
- 错误码与HTTP状态码映射
- 错误日志规范
- 告警触发规则

### 1.2 错误码设计原则

| 原则 | 说明 |
|-----|------|
| **层次分明** | 错误码按模块分类，便于快速定位问题 |
| **自描述** | 错误码本身应携带足够信息，无需额外查询 |
| **可扩展** | 预留扩展空间，支持新增错误类型 |
| **国际化** | 错误消息支持多语言，默认为中文 |

---

## 2. 错误码体系

### 2.1 错误码结构

```
ERR_{模块}_{类型}_{序号}

示例：
ERR_RCA_DATA_001  - 根因分析模块，数据类错误，第001号
ERR_ANOMALY_LLM_002 - 异常检测模块，LLM类错误，第002号
ERR_SHADOW_COMP_003 - 影子模式模块，对比类错误，第003号
```

### 2.2 模块划分

| 模块代码 | 模块名称 | 适用Agent |
|---------|---------|----------|
| `RCA` | 根因分析模块 | 根因分析Agent |
| `ANOMALY` | 异常检测模块 | 异常检测Agent |
| `ALERT` | 告警收敛模块 | 告警收敛Agent |
| `CAPACITY` | 容量规划模块 | 容量规划Agent |
| `LOG` | 日志分析模块 | 日志分析Agent |
| `CHANGE` | 变更管理模块 | 变更管理Agent |
| `PREDICT` | 故障预测模块 | 故障预测Agent |
| `KNOWLEDGE` | 知识推荐模块 | 知识推荐Agent |
| `SHADOW` | 影子模式模块 | 所有Agent |
| `COMMON` | 公共模块 | 所有Agent |

### 2.3 错误类型

| 类型代码 | 类型名称 | 说明 |
|---------|---------|------|
| `DATA` | 数据类错误 | 数据采集、存储、校验相关错误 |
| `LLM` | LLM类错误 | LLM服务调用相关错误 |
| `KG` | 知识图谱类错误 | 知识图谱查询、推理相关错误 |
| `CAUSAL` | 因果推断类错误 | 因果分析相关错误 |
| `MODEL` | 模型类错误 | 模型加载、预测相关错误 |
| `TIMEOUT` | 超时类错误 | 服务调用超时 |
| `NETWORK` | 网络类错误 | 网络通信相关错误 |
| `VALIDATE` | 校验类错误 | 参数校验失败 |
| `AUTH` | 认证授权类错误 | 权限相关错误 |
| `SYSTEM` | 系统类错误 | 系统级别错误 |
| `COMP` | 对比类错误 | 影子模式对比相关错误 |
| `CONFIG` | 配置类错误 | 配置相关错误 |

---

## 3. 错误码详细定义

### 3.1 公共错误码 (COMMON)

| 错误码 | HTTP状态码 | 错误名称 | 说明 | 处理建议 |
|-------|-----------|---------|------|---------|
| `COMMON_SUCCESS` | 200 | 成功 | 请求成功 | - |
| `COMMON_VALIDATE_001` | 400 | 缺少必填参数 | 缺少必填参数 `{param_name}` | 检查请求参数 |
| `COMMON_VALIDATE_002` | 400 | 参数格式错误 | 参数 `{param_name}` 格式不正确 | 检查参数格式 |
| `COMMON_VALIDATE_003` | 400 | 参数值超出范围 | 参数 `{param_name}` 值超出范围 | 检查参数取值范围 |
| `COMMON_AUTH_001` | 401 | 认证失败 | API Key无效或已过期 | 检查认证信息 |
| `COMMON_AUTH_002` | 403 | 权限不足 | 无权限访问该资源 | 申请相应权限 |
| `COMMON_AUTH_003` | 403 | 请求被拒绝 | 请求被安全策略拦截 | 联系安全团队 |
| `COMMON_NETWORK_001` | 502 | 服务不可达 | 无法连接到后端服务 | 检查网络连接 |
| `COMMON_NETWORK_002` | 504 | 请求超时 | 后端服务响应超时 | 增加超时时间或检查服务状态 |
| `COMMON_SYSTEM_001` | 500 | 内部错误 | 系统内部错误 | 查看错误详情，联系运维 |
| `COMMON_SYSTEM_002` | 503 | 服务降级 | 服务处于降级运行状态 | 等待服务恢复 |
| `COMMON_SYSTEM_003` | 503 | 服务不可用 | 服务暂时不可用 | 稍后重试 |

### 3.2 根因分析模块错误码 (RCA)

| 错误码 | HTTP状态码 | 错误名称 | 说明 | 处理建议 |
|-------|-----------|---------|------|---------|
| `RCA_DATA_001` | 400 | 告警数据缺失 | 告警数据为空或格式错误 | 检查告警ID是否有效 |
| `RCA_DATA_002` | 400 | 故障时间无效 | 故障时间格式错误或超出范围 | 检查故障时间参数 |
| `RCA_DATA_003` | 500 | 指标数据获取失败 | 无法从Prometheus获取指标数据 | 检查Prometheus连接 |
| `RCA_DATA_004` | 500 | 拓扑数据获取失败 | 无法从服务注册中心获取拓扑 | 检查服务注册中心连接 |
| `RCA_DATA_005` | 500 | 日志数据获取失败 | 无法从Loki获取日志数据 | 检查Loki连接 |
| `RCA_DATA_006` | 400 | 数据质量不达标 | 数据完整性低于阈值 `{threshold}` | 等待数据补充或手动触发 |
| `RCA_KG_001` | 500 | 知识图谱不可用 | 知识图谱服务连接失败 | 检查知识图谱服务状态 |
| `RCA_KG_002` | 500 | 知识图谱查询超时 | 知识图谱查询超过 `{timeout}s` | 增加超时时间 |
| `RCA_KG_003` | 404 | 实体未找到 | 知识图谱中未找到实体 `{entity}` | 检查拓扑数据是否同步 |
| `RCA_LLM_001` | 500 | LLM服务不可用 | LLM推理服务不可用 | 检查LLM服务状态 |
| `RCA_LLM_002` | 504 | LLM推理超时 | LLM推理超过 `{timeout}s` | 增加超时时间 |
| `RCA_LLM_003` | 500 | LLM响应格式错误 | LLM返回格式无法解析 | 检查Prompt模板 |
| `RCA_LLM_004` | 500 | LLM令牌超限 | 请求token数超过模型上限 | 减少输入数据量 |
| `RCA_CAUSAL_001` | 500 | 因果推断失败 | 因果图构建失败 | 检查输入数据质量 |
| `RCA_CAUSAL_002` | 500 | 因果推断超时 | 因果推断计算超时 | 减少分析维度 |
| `RCA_MODEL_001` | 500 | 模型加载失败 | GNN模型加载失败 | 检查模型文件 |
| `RCA_MODEL_002` | 500 | 模型预测失败 | 模型推理过程出错 | 查看详细错误日志 |
| `RCA_TIMEOUT_001` | 504 | 分析超时 | 整体分析超过 `{timeout}s` | 增加超时时间或减少数据量 |
| `RCA_CONFIG_001` | 400 | 上下文窗口配置无效 | 时间窗口参数配置错误 | 检查context_window参数 |

### 3.3 异常检测模块错误码 (ANOMALY)

| 错误码 | HTTP状态码 | 错误名称 | 说明 | 处理建议 |
|-------|-----------|---------|------|---------|
| `ANOMALY_DATA_001` | 400 | 指标数据格式错误 | 指标数据JSON格式不正确 | 检查数据格式 |
| `ANOMALY_DATA_002` | 400 | 指标名称无效 | 指标名称 `{metric_name}` 不在白名单 | 检查指标名称 |
| `ANOMALY_DATA_003` | 400 | 指标值类型错误 | 指标值必须为数值类型 | 检查数据类型 |
| `ANOMALY_DATA_004` | 500 | 数据采集失败 | 从 `{source}` 采集数据失败 | 检查数据源连接 |
| `ANOMALY_MODEL_001` | 500 | Prophet模型加载失败 | Prophet模型加载失败 | 检查模型文件 |
| `ANOMALY_MODEL_002` | 500 | IsolationForest模型加载失败 | IsolationForest模型加载失败 | 检查模型文件 |
| `ANOMALY_MODEL_003` | 500 | LSTM模型加载失败 | LSTM模型加载失败 | 检查模型文件 |
| `ANOMALY_MODEL_004` | 500 | 模型预测异常 | 模型预测过程出现异常 | 查看详细错误日志 |
| `ANOMALY_MODEL_005` | 500 | 模型更新失败 | 增量模型更新失败 | 触发全量更新 |
| `ANOMALY_THRESHOLD_001` | 400 | 阈值配置无效 | 异常阈值配置超出合理范围 | 检查阈值配置 |
| `ANOMALY_TIMEOUT_001` | 504 | 检测超时 | 单指标检测超过 `{timeout}s` | 增加超时时间 |

### 3.4 影子模式模块错误码 (SHADOW)

| 错误码 | HTTP状态码 | 错误名称 | 说明 | 处理建议 |
|-------|-----------|---------|------|---------|
| `SHADOW_COMP_001` | 500 | 对比器不可用 | 影子对比器服务不可用 | 检查对比器服务状态 |
| `SHADOW_COMP_002` | 500 | 对比计算失败 | 对比指标计算失败 | 查看详细错误日志 |
| `SHADOW_COMP_003` | 500 | 对比超时 | 对比计算超过 `{timeout}s` | 增加超时时间 |
| `SHADOW_STORE_001` | 500 | 结果存储失败 | 影子结果写入失败 | 检查存储服务状态 |
| `SHADOW_STORE_002` | 500 | 结果读取失败 | 影子结果读取失败 | 检查存储服务状态 |
| `SHADOW_BASELINE_001` | 400 | 基准数据缺失 | 无基准数据可用于对比 | 检查基准配置 |
| `SHADOW_BASELINE_002` | 400 | 基准版本不匹配 | 基准版本与当前版本不一致 | 更新基准数据 |
| `SHADOW_ANNOTATION_001` | 400 | 标注数据格式错误 | 人工标注数据格式不正确 | 检查标注数据格式 |
| `SHADOW_ANNOTATION_002` | 404 | 标注不存在 | 指定的标注 `{id}` 不存在 | 检查标注ID |

---

## 4. 异常分类与处理策略

### 4.1 异常分类

```python
class AgentException(Exception):
    """Agent异常基类"""
    def __init__(self, code: str, message: str, details: dict = None):
        self.code = code
        self.message = message
        self.details = details or {}

class RetryableException(AgentException):
    """可重试异常 - 网络超时、临时不可用等"""
    pass

class NonRetryableException(AgentException):
    """不可重试异常 - 参数错误、权限问题等"""
    pass

class DegradableException(AgentException):
    """可降级异常 - 部分功能可用"""
    pass
```

### 4.2 异常处理策略

| 异常类型 | 处理策略 | 重试次数 | 重试间隔 |
|---------|---------|---------|---------|
| **网络超时** | 自动重试 | 3次 | 指数退避(1s, 2s, 4s) |
| **服务不可用** | 自动重试 + 降级 | 3次 | 指数退避(2s, 4s, 8s) |
| **LLM超时** | 降级到双推理模式 | 不重试 | - |
| **数据缺失** | 降级分析精度 | 不重试 | - |
| **参数错误** | 立即返回错误 | 0次 | - |
| **权限错误** | 立即返回错误 | 0次 | - |
| **系统内部错误** | 记录日志 + 告警 | 1次 | 5s |

### 4.3 降级异常处理流程

```
异常发生
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│ 判断异常类型                                                    │
├─────────────────────────────────────────────────────────────┤
│                                                                  │
│  可重试? ──是──▶ 执行重试策略                                    │
│     │                  │                                        │
│     │                  ▼                                        │
│     │            重试成功?                                      │
│     │               │    │                                     │
│     否            是    否                                       │
│     │            │      │                                      │
│     ▼            ▼      ▼                                      │
│ ┌─────────┐   继续   触发降级 ──▶ 降级异常                       │
│ │不可重试 │          │                                         │
│ └─────────┘          ▼                                         │
│                      降级成功?                                  │
│                        │    │                                   │
│                       是    否                                   │
│                        │      │                                 │
│                        ▼      ▼                                 │
│                    记录元数据   触发人工介入                      │
│                                                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. 错误响应格式

### 5.1 标准错误响应

```json
{
  "success": false,
  "error": {
    "code": "RCA_LLM_002",
    "name": "LLM推理超时",
    "message": "LLM推理超过30s",
    "details": {
      "timeout_seconds": 30,
      "actual_duration_ms": 30002,
      "suggested_action": "增加超时时间或检查LLM服务状态"
    },
    "trace_id": "trace_20260426_001",
    "timestamp": "2026-04-26T10:30:00Z"
  }
}
```

### 5.2 批量操作错误响应

```json
{
  "success": false,
  "error": {
    "code": "COMMON_BATCH_PARTIAL_FAILURE",
    "name": "批量操作部分失败",
    "message": "批量操作中部分请求失败",
    "details": {
      "total": 10,
      "success_count": 7,
      "failure_count": 3,
      "failures": [
        {
          "index": 2,
          "code": "RCA_DATA_001",
          "message": "告警数据缺失"
        },
        {
          "index": 5,
          "code": "ANOMALY_DATA_002",
          "message": "指标名称无效: invalid_metric"
        },
        {
          "index": 8,
          "code": "RCA_TIMEOUT_001",
          "message": "分析超时"
        }
      ]
    },
    "trace_id": "trace_20260426_002",
    "timestamp": "2026-04-26T10:30:00Z"
  }
}
```

---

## 6. 错误日志规范

### 6.1 日志格式

```json
{
  "timestamp": "2026-04-26T10:30:00.123Z",
  "level": "ERROR",
  "service": "rca-agent",
  "version": "v1.8.0",
  "trace_id": "trace_20260426_001",
  "span_id": "span_001",
  "code": "RCA_LLM_002",
  "message": "LLM推理超时",
  "context": {
    "fault_id": "fault_001",
    "alert_ids": ["alert_001", "alert_002"],
    "推理器": "llm",
    "超时时间": 30
  },
  "stack_trace": "...",
  "recommendation": "检查LLM服务状态或增加超时时间"
}
```

### 6.2 日志级别规范

| 级别 | 使用场景 | 示例 |
|-----|---------|------|
| **DEBUG** | 调试信息，仅开发环境 | 函数入口/出口、变量值 |
| **INFO** | 正常流程信息 | 分析开始/结束、数据量统计 |
| **WARNING** | 异常但可处理 | 降级触发、数据质量警告 |
| **ERROR** | 错误但未崩溃 | LLM超时（已降级）、单次请求失败 |
| **CRITICAL** | 严重错误，服务不可用 | 数据库不可用、模型加载失败 |

### 6.3 日志保留策略

| 级别 | 保留周期 | 存储介质 |
|-----|---------|---------|
| DEBUG | 3天 | 本地文件 |
| INFO | 7天 | Elasticsearch |
| WARNING | 30天 | Elasticsearch |
| ERROR | 90天 | Elasticsearch + S3归档 |
| CRITICAL | 180天 | Elasticsearch + S3归档 |

---

## 7. 告警触发规则

### 7.1 错误率告警

| 告警级别 | 触发条件 | 动作 | 通知方式 |
|---------|---------|------|---------|
| **Critical** | 5分钟内错误率 > 10% | 立即暂停受影响功能 | 电话+短信+钉钉 |
| **Warning** | 5分钟内错误率 > 5% | 记录告警，持续观察 | 钉钉 |
| **Info** | 5分钟内错误率 > 2% | 记录日志 | 邮件 |

### 7.2 特定错误告警

| 错误类型 | 触发条件 | 告警级别 | 通知方式 |
|---------|---------|---------|---------|
| **LLM服务不可用** | 持续 > 1分钟 | Critical | 电话+短信+钉钉 |
| **知识图谱不可用** | 持续 > 5分钟 | Warning | 钉钉 |
| **数据存储失败** | 连续3次 | Critical | 电话+短信+钉钉 |
| **降级模式触发** | 每触发1次 | Info | 邮件 |
| **影子对比器不可用** | 持续 > 10分钟 | Warning | 钉钉 |

### 7.3 系统资源告警

| 资源 | Warning阈值 | Critical阈值 | 通知方式 |
|-----|-----------|------------|---------|
| CPU使用率 | > 70% | > 90% | 钉钉 |
| 内存使用率 | > 75% | > 90% | 钉钉 |
| 磁盘使用率 | > 80% | > 90% | 钉钉+短信 |
| Kafka消费延迟 | > 10000条 | > 50000条 | 钉钉 |
| PostgreSQL连接数 | > 80%上限 | > 95%上限 | 钉钉+短信 |

---

## 8. SLO/SLA映射

### 8.1 各Agent SLO定义

| Agent | SLO指标 | 目标值 | SLA目标 |
|-------|--------|-------|--------|
| **根因分析Agent** | 分析成功率 | >= 98% | >= 95% |
| **根因分析Agent** | 分析耗时P99 | < 5分钟 | < 8分钟 |
| **根因分析Agent** | 服务可用性 | >= 99.5% | >= 99.0% |
| **异常检测Agent** | 检测成功率 | >= 99% | >= 98% |
| **异常检测Agent** | 检测延迟P99 | < 2秒 | < 5秒 |
| **异常检测Agent** | 服务可用性 | >= 99.9% | >= 99.5% |

### 8.2 违约处理

| 违约级别 | 触发条件 | 处理方式 |
|---------|---------|---------|
| **轻微** | SLO未达标持续1小时 | 发送报告给运维团队负责人 |
| **中等** | SLO未达标持续4小时 | 升级通知给部门负责人 |
| **严重** | SLO未达标持续8小时或SLA违约 | 触发应急响应流程 |

---

## 9. 健康检查接口

### 9.1 健康检查端点

```yaml
GET /health
GET /health/live
GET /health/ready
GET /health/detailed
```

### 9.2 响应格式

```json
{
  "status": "healthy",
  "timestamp": "2026-04-26T10:30:00Z",
  "version": "v1.8.0",
  "uptime_seconds": 86400,
  "checks": {
    "database": "healthy",
    "cache": "healthy",
    "llm_service": "degraded",
    "knowledge_graph": "healthy",
    "kafka": "healthy"
  },
  "metrics": {
    "requests_total": 10000,
    "errors_total": 50,
    "error_rate": 0.005
  }
}
```

---

## 10. 附录

### 10.1 错误码快速查询表

| 错误码 | 模块 | 类型 | 严重程度 |
|-------|-----|------|---------|
| COMMON_\* | 公共 | 多种 | 根据具体错误 |
| RCA_\* | 根因分析 | 多种 | 根据具体错误 |
| ANOMALY_\* | 异常检测 | 多种 | 根据具体错误 |
| SHADOW_\* | 影子模式 | 多种 | 根据具体错误 |

### 10.2 错误处理代码模板

```python
from typing import TypeVar, Optional
from functools import wraps
import time
import logging

T = TypeVar('T')

logger = logging.getLogger(__name__)

def agent_retry(max_retries: int = 3, base_delay: float = 1.0):
    """重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except RetryableException as e:
                    last_exception = e
                    if attempt < max_retries:
                        delay = base_delay * (2 ** attempt)
                        logger.warning(f"Retryable error, retrying in {delay}s: {e}")
                        time.sleep(delay)
                    else:
                        logger.error(f"Max retries exceeded: {e}")
                except NonRetryableException as e:
                    logger.error(f"Non-retryable error: {e}")
                    raise
            raise last_exception
        return wrapper
    return decorator

@agent_retry(max_retries=3, base_delay=2.0)
def analyze_with_retry(fault_id: str) -> AnalysisResult:
    """带重试的分析请求"""
    # 分析逻辑
    pass
```
