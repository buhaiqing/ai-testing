# 运维Agent SDK与客户端集成规范

| 属性 | 值 |
|-----|-----|
| **文档类型** | IMPL |
| **版本** | v1.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | 质量智能中台团队 |
| **关联文档** | [ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md](./ROOT_CAUSE_ANALYSIS_AGENT_REQUIREMENTS.md), [ANOMALY_DETECTION_AGENT_REQUIREMENTS.md](./ANOMALY_DETECTION_AGENT_REQUIREMENTS.md), [ERROR_CODE_AND_EXCEPTION_SPEC.md](./ERROR_CODE_AND_EXCEPTION_SPEC.md) |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v1.0.0 | 2026-04-26 | 初始版本 | 质量智能中台团队 |

---

## 1. 概述

### 1.1 文档目的

本规范定义了质量智能中台运维Agent的**SDK和客户端集成标准**，包括：
- SDK架构设计
- 多语言客户端规范
- 集成代码示例
- 最佳实践

### 1.2 SDK设计原则

| 原则 | 说明 |
|-----|------|
| **简洁易用** | API设计直观，减少集成成本 |
| **幂等性** | 支持安全重试，不产生副作用 |
| **可观测性** | 内置 tracing、logging、metrics |
| **向后兼容** | 版本升级不影响已有集成 |
| **多语言支持** | 提供Python、Java、Go、JavaScript等主流语言SDK |

---

## 2. SDK架构设计

### 2.1 SDK架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端SDK                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    SDK Core Layer                            ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          ││
│  │  │ HTTP Client │ │ Auth Module │ │ Retry Logic │          ││
│  │  └─────────────┘ └─────────────┘ └─────────────┘          ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Agent Service Layer                       ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          ││
│  │  │ RCA Client  │ │ Anomaly     │ │ Alert       │          ││
│  │  │             │ │ Client      │ │ Client      │          ││
│  │  └─────────────┘ └─────────────┘ └─────────────┘          ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                    │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Observability Layer                       ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          ││
│  │  │ Tracing     │ │ Metrics     │ │ Logging     │          ││
│  │  └─────────────┘ └─────────────┘ └─────────────┘          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 核心组件

| 组件 | 职责 |
|-----|------|
| **HTTP Client** | 统一HTTP通信，支持连接池、超时、重试 |
| **Auth Module** | API Key管理、Token刷新 |
| **Retry Logic** | 指数退避、熔断器模式 |
| **Circuit Breaker** | 故障隔离、快速失败 |
| **Rate Limiter** | 限流控制 |
| **Tracing** | 请求链路追踪 |
| **Metrics** | 性能指标采集 |
| **Logging** | 结构化日志输出 |

---

## 3. Python SDK规范

### 3.1 安装与初始化

```bash
pip install aiops-sdk
```

```python
from aiops import AiopsClient
from aiops.config import ClientConfig

# 初始化客户端
config = ClientConfig(
    api_key="your-api-key",
    base_url="https://api.aiops.example.com",
    timeout=30,
    max_retries=3,
    enable_tracing=True
)

client = AiopsClient(config)
```

### 3.2 根因分析客户端

```python
from aiops.rca import RCAClient
from aiops.rca.models import AnalysisRequest, ContextWindow

# 创建RCA客户端
rca_client = client.rca()

# 触发根因分析
request = AnalysisRequest(
    fault_id="fault_001",
    alert_ids=["alert_001", "alert_002"],
    fault_time=1714123456789,
    mode="shadow",
    context=ContextWindow(
        pre_fault_minutes=120,
        post_fault_minutes=30,
        max_alerts=100,
        include_logs=True,
        include_changes=True
    )
)

# 同步调用
response = rca_client.trigger_analysis(request)
print(f"Report ID: {response.report_id}")
print(f"Confidence: {response.root_cause.confidence}")

# 异步调用（返回future）
future = rca_client.trigger_analysis_async(request)
result = future.result(timeout=60)

# 订阅分析进度
for progress in rca_client.subscribe_analysis(request):
    print(f"Stage: {progress.stage}, Progress: {progress.percent}%")
```

### 3.3 异常检测客户端

```python
from aiops.anomaly import AnomalyClient
from aiops.anomaly.models import MetricRequest, DetectionMode

# 创建异常检测客户端
anomaly_client = client.anomaly()

# 提交单个指标
metric = MetricRequest(
    metric_name="cpu_usage",
    value=85.5,
    timestamp=1714123456789,
    tags={"host": "server-01", "env": "prod"},
    source_type="prometheus",
    mode=DetectionMode.SHADOW
)

response = anomaly_client.submit_metric(metric)
print(f"Anomaly Detected: {response.anomaly_detected}")

# 批量提交指标
metrics = [
    MetricRequest(metric_name="cpu_usage", value=85.5, timestamp=..., tags=...),
    MetricRequest(metric_name="memory_usage", value=72.3, timestamp=..., tags=...),
]
batch_response = anomaly_client.batch_submit_metrics(metrics)
print(f"Processed: {batch_response.total}, Anomalies: {batch_response.anomaly_count}")

# 订阅异常事件
for event in anomaly_client.subscribe_anomalies():
    print(f"Anomaly Alert: {event.details.severity} - {event.details.anomaly_type}")
```

### 3.4 错误处理

```python
from aiops.exceptions import (
    AiopsException,
    ValidationError,
    AuthenticationError,
    RateLimitError,
    ServiceUnavailableError,
    AnalysisTimeoutError
)

try:
    response = rca_client.trigger_analysis(request)
except ValidationError as e:
    print(f"参数错误: {e.code}, {e.message}")
    print(f"详情: {e.details}")
except AuthenticationError as e:
    print(f"认证失败: {e.code}")
    # 刷新token或检查api key
except RateLimitError as e:
    print(f"限流: 剩余 {e.details['retry_after']} 秒")
    time.sleep(e.details['retry_after'])
    # 重试
except AnalysisTimeoutError as e:
    print(f"分析超时: {e.details['timeout']}s")
    # 增加超时或减少数据量
except ServiceUnavailableError as e:
    print(f"服务不可用: {e.code}")
    # 降级处理或等待恢复
except AiopsException as e:
    print(f"其他错误: {e.code}, {e.message}")
    # 通用错误处理
```

### 3.5 重试与熔断

```python
from aiops.circuit_breaker import CircuitBreaker, CircuitState

# 配置熔断器
circuit_breaker = CircuitBreaker(
    failure_threshold=5,      # 5次失败后打开
    recovery_timeout=60,      # 60秒后尝试半开
    expected_exception=ServiceUnavailableError
)

@circuit_breaker
def call_with_circuit_breaker():
    return rca_client.trigger_analysis(request)

# 检查熔断器状态
print(f"Circuit State: {circuit_breaker.state}")
print(f"Failure Count: {circuit_breaker.failure_count}")
```

### 3.6 完整集成示例

```python
"""
完整集成示例：故障自愈流程
"""
import logging
from aiops import AiopsClient
from aiops.rca.models import AnalysisRequest, ContextWindow
from aiops.exceptions import AnalysisTimeoutError, ServiceUnavailableError

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class FaultAutoHealing:
    def __init__(self, api_key: str):
        self.client = AiopsClient(api_key=api_key)
        self.rca = self.client.rca()
        self.anomaly = self.client.anomaly()

    def process_fault(self, fault_id: str, alert_ids: list):
        try:
            # Step 1: 触发根因分析
            logger.info(f"Starting RCA for fault: {fault_id}")
            request = AnalysisRequest(
                fault_id=fault_id,
                alert_ids=alert_ids,
                fault_time=1714123456789,
                mode="shadow",
                context=ContextWindow()
            )

            response = self.rca.trigger_analysis(request)

            # Step 2: 检查置信度
            if response.root_cause.confidence >= 0.85:
                logger.info(f"High confidence root cause: {response.root_cause.specific_issue}")

                # Step 3: 自动执行修复建议（高置信度）
                for suggestion in response.repair_suggestions:
                    if suggestion.priority == "P0":
                        logger.info(f"Executing P0 action: {suggestion.action}")
                        self.execute_repair(suggestion)
            else:
                logger.warning(f"Low confidence ({response.root_cause.confidence}), manual review needed")

            # Step 4: 发送通知
            self.send_notification(response)

            return response

        except AnalysisTimeoutError as e:
            logger.error(f"Analysis timeout: {e}")
            return None
        except ServiceUnavailableError as e:
            logger.error(f"Service unavailable: {e}")
            return None

    def execute_repair(self, suggestion):
        """执行修复操作"""
        for step in suggestion.steps:
            logger.info(f"Executing: {step}")

    def send_notification(self, response):
        """发送通知"""
        logger.info(f"Sending notification for report: {response.report_id}")


# 使用示例
if __name__ == "__main__":
    client = FaultAutoHealing(api_key="your-api-key")
    result = client.process_fault(
        fault_id="fault_001",
        alert_ids=["alert_001", "alert_002"]
    )
```

---

## 4. Java SDK规范

### 4.1 Maven依赖

```xml
<dependency>
    <groupId>com.aiops</groupId>
    <artifactId>aiops-sdk-java</artifactId>
    <version>1.0.0</version>
</dependency>
```

### 4.2 客户端初始化

```java
import com.aiops.sdk.AiopsClient;
import com.aiops.sdk.config.ClientConfig;

public class AiopsClientExample {
    public static void main(String[] args) {
        ClientConfig config = ClientConfig.builder()
            .apiKey("your-api-key")
            .baseUrl("https://api.aiops.example.com")
            .timeout(Duration.ofSeconds(30))
            .maxRetries(3)
            .enableTracing(true)
            .build();

        AiopsClient client = new AiopsClient(config);
    }
}
```

### 4.3 根因分析调用

```java
import com.aiops.sdk.rca.RCAClient;
import com.aiops.sdk.rca.model.*;

public class RCAExample {
    public void triggerAnalysis(AiopsClient client) {
        RCAClient rcaClient = client.rca();

        AnalysisRequest request = AnalysisRequest.builder()
            .faultId("fault_001")
            .alertIds(Arrays.asList("alert_001", "alert_002"))
            .faultTime(1714123456789L)
            .mode(AnalysisMode.SHADOW)
            .context(ContextWindow.builder()
                .preFaultMinutes(120)
                .postFaultMinutes(30)
                .maxAlerts(100)
                .includeLogs(true)
                .includeChanges(true)
                .build())
            .build();

        // 同步调用
        AnalysisResponse response = rcaClient.triggerAnalysis(request);
        System.out.println("Report ID: " + response.getReportId());
        System.out.println("Confidence: " + response.getRootCause().getConfidence());

        // 异步调用
        CompletableFuture<AnalysisResponse> future = rcaClient.triggerAnalysisAsync(request);
        future.thenAccept(r -> System.out.println("Async result: " + r.getReportId()));
    }
}
```

---

## 5. Go SDK规范

### 5.1 安装

```bash
go get github.com/aiops/sdk-go
```

### 5.2 客户端初始化

```go
package main

import (
    "context"
    "time"

    aiops "github.com/aiops/sdk-go"
)

func main() {
    client, err := aiops.NewClient(
        aiops.WithAPIKey("your-api-key"),
        aiops.WithBaseURL("https://api.aiops.example.com"),
        aiops.WithTimeout(30*time.Second),
        aiops.WithMaxRetries(3),
    )
    if err != nil {
        panic(err)
    }
    defer client.Close()
}
```

### 5.3 根因分析调用

```go
package main

import (
    "context"
    "fmt"

    aiops "github.com/aiops/sdk-go"
)

func main() {
    client, _ := aiops.NewClient(aiops.WithAPIKey("your-api-key"))

    ctx := context.Background()
    request := &aiops.AnalysisRequest{
        FaultID:   "fault_001",
        AlertIDs:  []string{"alert_001", "alert_002"},
        FaultTime: 1714123456789,
        Mode:      aiops.ModeShadow,
        Context: &aiops.ContextWindow{
            PreFaultMinutes:  120,
            PostFaultMinutes: 30,
            MaxAlerts:       100,
            IncludeLogs:     true,
            IncludeChanges:  true,
        },
    }

    response, err := client.RCA.TriggerAnalysis(ctx, request)
    if err != nil {
        panic(err)
    }

    fmt.Printf("Report ID: %s\n", response.ReportID)
    fmt.Printf("Confidence: %.2f\n", response.RootCause.Confidence)
}
```

---

## 6. JavaScript/TypeScript SDK规范

### 6.1 安装

```bash
npm install @aiops/sdk
```

### 6.2 客户端初始化

```typescript
import { AiopsClient } from '@aiops/sdk';

const client = new AiopsClient({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.aiops.example.com',
  timeout: 30000,
  maxRetries: 3,
  enableTracing: true,
});
```

### 6.3 根因分析调用

```typescript
import { AnalysisMode, ContextWindow } from '@aiops/sdk';

async function triggerAnalysis() {
  const rcaClient = client.rca();

  const request = {
    faultId: 'fault_001',
    alertIds: ['alert_001', 'alert_002'],
    faultTime: 1714123456789,
    mode: AnalysisMode.SHADOW,
    context: new ContextWindow({
      preFaultMinutes: 120,
      postFaultMinutes: 30,
      maxAlerts: 100,
      includeLogs: true,
      includeChanges: true,
    }),
  };

  try {
    // 同步调用
    const response = await rcaClient.triggerAnalysis(request);
    console.log(`Report ID: ${response.reportId}`);
    console.log(`Confidence: ${response.rootCause.confidence}`);

    // 订阅进度
    for await (const progress of rcaClient.subscribeAnalysis(request)) {
      console.log(`Stage: ${progress.stage}, Progress: ${progress.percent}%`);
    }
  } catch (error) {
    if (error.code === 'RCA_TIMEOUT_001') {
      console.error('Analysis timeout');
    } else {
      throw error;
    }
  }
}
```

---

## 7. API集成规范

### 7.1 认证方式

| 方式 | 说明 | 适用场景 |
|-----|------|---------|
| **API Key** | 在请求头中传递`X-API-Key` | 客户端SDK集成 |
| **OAuth 2.0** | 支持Token刷新 | 企业级应用 |
| **JWT** | 支持短期Token | Web应用 |

**API Key认证示例：**
```http
GET /api/v1/rca/analysis HTTP/1.1
Host: api.aiops.example.com
X-API-Key: your-api-key
Content-Type: application/json
X-Request-ID: unique-request-id
X-Trace-ID: trace-id-for-correlation
```

### 7.2 请求限制

| 限制类型 | 限制值 | 说明 |
|---------|-------|------|
| **QPS限制** | 100 QPS/用户 | 按API Key区分 |
| **并发限制** | 50 并发/用户 | 同时进行的请求数 |
| **日请求量** | 100,000/日 | 超出后限流 |
| **单次超时** | 60秒 | 超过后自动断开 |

### 7.3 限流响应

```json
{
  "success": false,
  "error": {
    "code": "COMMON_RATE_LIMIT",
    "name": "请求频率超限",
    "message": "每秒请求数超过限制",
    "details": {
      "limit": 100,
      "remaining": 0,
      "retry_after": 1
    }
  }
}
```

**响应头：**
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1714123460
Retry-After: 1
```

---

## 8. 集成检查清单

### 8.1 集成前检查

| 检查项 | 说明 | 状态 |
|-------|------|------|
| API Key配置 | 已获取有效的API Key | □ |
| 网络连通性 | 能访问API服务 | □ |
| 权限配置 | API Key有所需权限 | □ |
| 超时配置 | 配置合理的超时时间 | □ |
| 重试机制 | 实现指数退避重试 | □ |
| 错误处理 | 处理所有错误码 | □ |

### 8.2 集成后检查

| 检查项 | 说明 | 状态 |
|-------|------|------|
|  Tracing | 请求链路可追踪 | □ |
|  Logging | 日志格式正确输出 | □ |
|  Metrics | 关键指标已采集 | □ |
|  限流处理 | 正确处理429响应 | □ |
|  健康检查 | 健康检查接口正常 | □ |
|  灰度发布 | 支持版本切换 | □ |

---

## 9. 常见问题

### 9.1 FAQ

| 问题 | 答案 |
|-----|------|
| **Q: 如何处理限流？** | 使用指数退避重试，检查`Retry-After`响应头 |
| **Q: 如何保证幂等性？** | 使用`X-Idempotency-Key`请求头 |
| **Q: 如何处理超时？** | 设置合理超时，实现重试机制 |
| **Q: 如何调试问题？** | 启用`X-Debug-Mode`请求头获取详细日志 |
| **Q: 如何切换环境？** | 通过`base_url`配置切换测试/生产环境 |

### 9.2 幂等性示例

```python
import uuid

# 使用幂等Key
idempotency_key = str(uuid.uuid4())
response = rca_client.trigger_analysis(
    request,
    headers={"X-Idempotency-Key": idempotency_key}
)
```

---

## 10. 版本兼容矩阵

| SDK版本 | API版本 | 支持的Python版本 | 支持的Java版本 |
|-------|--------|----------------|---------------|
| 1.0.x | v1 | 3.8, 3.9, 3.10, 3.11 | 8, 11, 17 |
| 1.1.x | v1 | 3.8, 3.9, 3.10, 3.11, 3.12 | 8, 11, 17, 21 |
| 2.0.x | v2 | 3.9, 3.10, 3.11, 3.12 | 11, 17, 21 |
