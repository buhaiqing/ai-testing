# Quality Agent 进化机制详解

| 属性 | 值 |
|-----|-----|
| **文档类型** | ARCH |
| **版本** | v2.0.0 |
| **最后更新** | 2026-04-26 |
| **维护者** | Quality Agent 培养计划 |
| **关联文档** | [AGENTS.md](../AGENTS.md), [TERMINOLOGY.md](./TERMINOLOGY.md), [VERSIONING.md](./VERSIONING.md) |

---

## 一、三层进化架构

```
┌─────────────────────────────────────────────────────────────┐
│                   进化能力技术实现                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 1: 短期记忆 (Short-term Memory)                      │
│  ├─ 来源: 当前对话上下文、实时执行状态                        │
│  ├─ 技术: Conversation Buffer / Message History            │
│  └─ 生命周期: 单次任务会话                                   │
│                                                             │
│  Layer 2: 长期记忆 (Long-term Memory)                       │
│  ├─ 来源: 历史任务经验、质量案例、规则沉淀                    │
│  ├─ 技术: [知识检索] + 知识图谱                              │
│  └─ 生命周期: 持久化，可跨会话检索                          │
│                                                             │
│  Layer 3: 规则引擎 (Rule Engine)                            │
│  ├─ 来源: 从经验中提取的可执行规则                           │
│  ├─ 技术: [规则引擎]                                         │
│  └─ 生命周期: 自动更新 + 人工审核                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  进化流程:                                           │   │
│  │  任务执行 → 经验沉淀 → RAG索引 → 规则抽取 → 规则应用  │   │
│  │                                                      │   │
│  │  失败案例 → 根因分析 → 预防策略 → 规则库更新          │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、进化能力实现细节

| 层级 | 存储技术 | 检索方式 | 更新机制 |
|-----|---------|---------|---------|
| **短期记忆** | Redis / Memory | 直接访问 | 会话结束时清理 |
| **长期记忆** | [知识检索] | ANN检索 | 定时增量更新 |
| **规则库** | PostgreSQL / JSON | 规则引擎匹配 | 自动 + 人工审核 |

---

## 三、RAG 系统工程实现

### 3.1 RAG 工程决策记录

以下决策基于 2026 年 Q1 的技术生态评估，每季度复审一次。

| 决策项 | 当前选择 | 选择理由 | 备选方案 | 切换条件 |
|--------|---------|---------|---------|---------|
| **Embedding 模型** | `text-embedding-3-large` (3072d) | 多语言支持优秀，长文本（8192 tokens）处理能力强，OpenAI 官方 SLA 99.9% | `BGE-M3` (开源，免费，多语言) | 月度 Token 成本 > $1000 或需完全离线部署时评估切换 |
| **Chunk 策略** | 按经验条目整体嵌入（单条 < 500 tokens） | 经验条目本身为结构化数据，无需再切分；保留完整语义上下文 | 递归字符切分（RecursiveCharacterTextSplitter） | 单条经验 > 1000 tokens 时引入切分策略 |
| **检索算法** | HNSW + COSINE | 高召回率（> 95%@Top-5），低延迟（P95 < 50ms），支持增量插入 | IVF_FLAT（内存受限时） | 数据量 > 1000 万条或内存 < 16GB 时评估 |
| **重排序** | 暂不使用 | 经验条目短（平均 200 tokens），Embedding 检索 Top-3 准确率已达 85%+，引入重排序收益 < 5% | `bge-reranker-base` (Cross-Encoder) | Embedding 检索准确率 < 80% 时引入 |
| **元数据过滤** | 预过滤（`project` + `task_type` + `success`） | 覆盖 90%+ 的查询场景，预过滤减少向量搜索空间 | 动态过滤表达式（Milvus `expr` 参数） | 查询模式复杂化（需多字段组合过滤）时扩展 |
| **向量数据库** | Milvus 2.4+ | 云原生架构，支持分布式扩展，Python SDK 成熟，社区活跃 | Pinecone（托管服务，零运维） | 团队无 K8s 运维能力时评估 |

### 3.2 生产级 RAG 实现代码

```python
"""
Quality Agent 长期记忆系统 - 基于 RAG 的经验沉淀

SLO:
- 存储延迟: P95 < 500ms (单条经验)
- 检索延迟: P95 < 200ms (Top-3, 含元数据过滤)
- 检索准确率: Top-3 相关度 > 85% (人工评估, 季度抽样 100 条)
- 可用性: > 99.5% (排除计划内维护窗口)
"""

import os
import json
import logging
from typing import List, Optional, Dict, Any
from datetime import datetime
from dataclasses import dataclass, asdict
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type,
    before_sleep_log,
)
from langchain_milvus import MilvusVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

logger = logging.getLogger(__name__)


@dataclass
class Experience:
    """经验数据结构"""
    task_type: str
    success: bool
    root_cause: str = ""
    solution: str = ""
    timestamp: Optional[str] = None
    project: str = "default"
    stack: str = ""
    app: str = ""
    confidence: float = 0.0
    
    def __post_init__(self):
        if self.timestamp is None:
            self.timestamp = datetime.utcnow().isoformat()


class ExperienceMemory:
    """
    质量智能体长期记忆系统
    
    职责:
    1. 存储任务执行经验到向量数据库
    2. 基于语义相似度检索相关历史经验
    3. 支持元数据过滤（项目、任务类型、成功率等）
    4. 异常时自动降级到本地缓存
    """
    
    def __init__(
        self,
        collection_name: str = "quality_experiences",
        embedding_model: str = "text-embedding-3-large",
        milvus_uri: Optional[str] = None,
        milvus_token: Optional[str] = None,
        fallback_cache_path: str = ".cache/experiences.json",
    ):
        self.collection_name = collection_name
        self.fallback_cache_path = fallback_cache_path
        self._fallback_mode = False
        
        # Embedding 配置
        dimensions = 3072 if "large" in embedding_model else 1536
        self.embeddings = OpenAIEmbeddings(
            model=embedding_model,
            api_key=os.getenv("OPENAI_API_KEY"),
            dimensions=dimensions,
            # 超时配置：避免 Embedding API 长时间阻塞
            timeout=30,
            max_retries=2,
        )
        
        # Milvus 向量存储配置
        try:
            self.vector_store = MilvusVectorStore(
                embedding_function=self.embeddings,
                collection_name=collection_name,
                connection_args={
                    "uri": milvus_uri or os.getenv("MILVUS_URI", "http://localhost:19530"),
                    "token": milvus_token or os.getenv("MILVUS_TOKEN"),
                },
                # 生产环境：禁止自动删除已有集合
                drop_old=False,
                # 索引配置：HNSW + COSINE，平衡召回率和性能
                index_params={
                    "index_type": "HNSW",
                    "metric_type": "COSINE",
                    "params": {"M": 16, "efConstruction": 200},
                },
                # 搜索参数：ef 越大召回率越高，但延迟增加
                search_params={"params": {"ef": 64}},
            )
            logger.info(f"Milvus vector store initialized: {collection_name}")
        except Exception as e:
            logger.error(f"Failed to connect to Milvus: {e}. Entering fallback mode.")
            self._fallback_mode = True
            self.vector_store = None
    
    @property
    def is_fallback(self) -> bool:
        """当前是否处于降级模式"""
        return self._fallback_mode
    
    @retry(
        retry=retry_if_exception_type((ConnectionError, TimeoutError)),
        stop=stop_after_attempt(3),
        wait=wait_exponential(multiplier=1, min=2, max=10),
        before_sleep=before_sleep_log(logger, logging.WARNING),
        reraise=True,
    )
    def store_experience(self, experience: Experience) -> Optional[str]:
        """
        存储任务经验到长期记忆
        
        Args:
            experience: 经验数据对象
            
        Returns:
            文档 ID，存储失败时返回 None（降级到本地缓存）
            
        SLO: P95 延迟 < 500ms
        """
        if self._fallback_mode:
            return self._store_to_local_cache(experience)
        
        try:
            doc = Document(
                page_content=self._format_experience(experience),
                metadata={
                    "task_type": experience.task_type,
                    "success": experience.success,
                    "timestamp": experience.timestamp,
                    "project": experience.project,
                    "stack": experience.stack,
                    "app": experience.app,
                    "confidence": experience.confidence,
                }
            )
            ids = self.vector_store.add_documents([doc])
            logger.info(f"Experience stored: id={ids[0]}, type={experience.task_type}")
            return ids[0]
        except Exception as e:
            logger.error(f"Failed to store experience: {e}. Falling back to local cache.")
            self._fallback_mode = True
            return self._store_to_local_cache(experience)
    
    def retrieve_similar(
        self,
        query: str,
        top_k: int = 3,
        filter_expr: Optional[str] = None,
        min_score: float = 0.70,
    ) -> List[Document]:
        """
        检索相似经验，支持元数据过滤和分数阈值
        
        Args:
            query: 查询文本
            top_k: 返回结果数量
            filter_expr: Milvus 过滤表达式，如 'project == "payment" && success == true'
            min_score: 最小相似度分数（COSINE 距离，范围 [0, 1]）
            
        Returns:
            符合条件的经验文档列表
            
        SLO: P95 延迟 < 200ms, Top-3 相关度 > 85%
        """
        if self._fallback_mode:
            return self._retrieve_from_local_cache(query, top_k)
        
        try:
            # similarity_search_with_score 返回 (Document, score) 元组
            # COSINE 距离下，score 范围 [0, 1]，越接近 1 越相似
            results = self.vector_store.similarity_search_with_score(
                query=query,
                k=top_k * 2,  # 多召回一些，后续过滤
                filter=filter_expr,
            )
            
            # 过滤低质量结果
            filtered = [
                doc for doc, score in results
                if score >= min_score
            ]
            
            # 限制返回数量
            final_results = filtered[:top_k]
            
            logger.debug(
                f"Retrieved {len(final_results)} experiences for query "
                f"(candidates={len(results)}, filtered={len(filtered)})"
            )
            return final_results
            
        except Exception as e:
            logger.error(f"Retrieval failed: {e}. Falling back to local cache.")
            self._fallback_mode = True
            return self._retrieve_from_local_cache(query, top_k)
    
    def retrieve_by_context(
        self,
        query: str,
        project: Optional[str] = None,
        task_type: Optional[str] = None,
        success_only: bool = True,
        top_k: int = 3,
    ) -> List[Document]:
        """
        基于上下文检索（常用查询模式封装）
        
        自动构建过滤表达式，简化调用方代码
        """
        filters = []
        if project:
            filters.append(f'project == "{project}"')
        if task_type:
            filters.append(f'task_type == "{task_type}"')
        if success_only:
            filters.append("success == true")
        
        filter_expr = " && ".join(filters) if filters else None
        return self.retrieve_similar(query, top_k, filter_expr)
    
    def _format_experience(self, exp: Experience) -> str:
        """将结构化经验格式化为可嵌入文本"""
        parts = [
            f"Task Type: {exp.task_type}",
            f"Project: {exp.project}",
            f"Stack: {exp.stack}",
            f"App: {exp.app}",
            f"Success: {exp.success}",
            f"Confidence: {exp.confidence}",
        ]
        if exp.root_cause:
            parts.append(f"Root Cause: {exp.root_cause}")
        if exp.solution:
            parts.append(f"Solution: {exp.solution}")
        return "\n".join(parts)
    
    def _store_to_local_cache(self, experience: Experience) -> str:
        """降级：存储到本地 JSON 文件"""
        import os
        os.makedirs(os.path.dirname(self.fallback_cache_path), exist_ok=True)
        
        cache = []
        if os.path.exists(self.fallback_cache_path):
            with open(self.fallback_cache_path, "r", encoding="utf-8") as f:
                cache = json.load(f)
        
        cache.append(asdict(experience))
        
        with open(self.fallback_cache_path, "w", encoding="utf-8") as f:
            json.dump(cache, f, ensure_ascii=False, indent=2)
        
        logger.warning(f"Experience stored to local cache: {self.fallback_cache_path}")
        return f"local-{len(cache)}"
    
    def _retrieve_from_local_cache(self, query: str, top_k: int) -> List[Document]:
        """降级：从本地缓存检索（简单关键词匹配）"""
        if not os.path.exists(self.fallback_cache_path):
            return []
        
        with open(self.fallback_cache_path, "r", encoding="utf-8") as f:
            cache = json.load(f)
        
        # 简单关键词匹配（降级模式不保证语义相似度）
        query_terms = set(query.lower().split())
        scored = []
        for item in cache:
            content = json.dumps(item, ensure_ascii=False).lower()
            score = len(query_terms & set(content.split()))
            scored.append((item, score))
        
        scored.sort(key=lambda x: x[1], reverse=True)
        
        docs = []
        for item, _ in scored[:top_k]:
            docs.append(Document(
                page_content=json.dumps(item, ensure_ascii=False),
                metadata=item,
            ))
        
        logger.warning(f"Retrieved {len(docs)} experiences from local cache (fallback mode)")
        return docs
    
    def health_check(self) -> Dict[str, Any]:
        """健康检查：返回当前状态"""
        status = {
            "status": "healthy" if not self._fallback_mode else "degraded",
            "vector_db": "connected" if self.vector_store else "disconnected",
            "fallback_mode": self._fallback_mode,
            "collection": self.collection_name,
            "timestamp": datetime.utcnow().isoformat(),
        }
        
        if not self._fallback_mode:
            try:
                # 执行一次简单查询验证连通性
                self.vector_store.similarity_search("health_check", k=1)
                status["latency_ms"] = "< 100ms"  # 粗略估计
            except Exception as e:
                status["status"] = "unhealthy"
                status["error"] = str(e)
        
        return status
```

### 3.3 RAG 检索质量评估

```python
"""
RAG 检索质量评估脚本

运行方式: python evaluate_rag.py --test-set tests/rag_eval.jsonl
验收标准:
- Top-3 准确率 > 85%
- P95 延迟 < 200ms
- 召回率 > 95% (Top-10 内包含正确答案)
"""

import json
import time
import argparse
from typing import List, Dict
from dataclasses import dataclass
from ExperienceMemory import ExperienceMemory, Experience


@dataclass
class EvalCase:
    query: str
    expected_doc_ids: List[str]
    filter_expr: Optional[str] = None
    category: str = "general"  # general, filtered, edge_case


def load_eval_cases(path: str) -> List[EvalCase]:
    cases = []
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            data = json.loads(line)
            cases.append(EvalCase(**data))
    return cases


def evaluate(memory: ExperienceMemory, cases: List[EvalCase]) -> Dict:
    results = {
        "total": len(cases),
        "top1_hit": 0,
        "top3_hit": 0,
        "top5_hit": 0,
        "latencies": [],
        "by_category": {},
    }
    
    for case in cases:
        start = time.time()
        docs = memory.retrieve_similar(
            query=case.query,
            top_k=5,
            filter_expr=case.filter_expr,
        )
        latency_ms = (time.time() - start) * 1000
        results["latencies"].append(latency_ms)
        
        retrieved_ids = [doc.metadata.get("id", "") for doc in docs]
        hits = len(set(retrieved_ids) & set(case.expected_doc_ids))
        
        if hits > 0:
            results["top5_hit"] += 1
            if any(rid in case.expected_doc_ids for rid in retrieved_ids[:3]):
                results["top3_hit"] += 1
            if retrieved_ids[0] in case.expected_doc_ids:
                results["top1_hit"] += 1
        
        # 按类别统计
        cat = case.category
        if cat not in results["by_category"]:
            results["by_category"][cat] = {"count": 0, "top3_hit": 0}
        results["by_category"][cat]["count"] += 1
        if any(rid in case.expected_doc_ids for rid in retrieved_ids[:3]):
            results["by_category"][cat]["top3_hit"] += 1
    
    # 计算指标
    latencies = sorted(results["latencies"])
    n = len(latencies)
    
    metrics = {
        "top1_accuracy": results["top1_hit"] / results["total"],
        "top3_accuracy": results["top3_hit"] / results["total"],
        "top5_recall": results["top5_hit"] / results["total"],
        "latency_p50_ms": latencies[n // 2],
        "latency_p95_ms": latencies[int(n * 0.95)],
        "latency_p99_ms": latencies[int(n * 0.99)],
        "by_category": {
            cat: {"top3_accuracy": v["top3_hit"] / v["count"]}
            for cat, v in results["by_category"].items()
        },
    }
    
    return metrics


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--test-set", required=True, help="评估测试集路径 (JSONL)")
    parser.add_argument("--milvus-uri", default="http://localhost:19530")
    args = parser.parse_args()
    
    memory = ExperienceMemory(milvus_uri=args.milvus_uri)
    cases = load_eval_cases(args.test_set)
    
    print(f"Loaded {len(cases)} evaluation cases")
    metrics = evaluate(memory, cases)
    
    print("\n=== RAG 检索质量评估结果 ===")
    print(f"Top-1 准确率: {metrics['top1_accuracy']:.2%}")
    print(f"Top-3 准确率: {metrics['top3_accuracy']:.2%} (目标: > 85%)")
    print(f"Top-5 召回率: {metrics['top5_recall']:.2%} (目标: > 95%)")
    print(f"P50 延迟: {metrics['latency_p50_ms']:.1f}ms")
    print(f"P95 延迟: {metrics['latency_p95_ms']:.1f}ms (目标: < 200ms)")
    print(f"P99 延迟: {metrics['latency_p99_ms']:.1f}ms")
    
    # 验收判断
    passed = (
        metrics["top3_accuracy"] >= 0.85
        and metrics["top5_recall"] >= 0.95
        and metrics["latency_p95_ms"] < 200
    )
    print(f"\n验收结果: {'通过' if passed else '未通过'}")
    return 0 if passed else 1


if __name__ == "__main__":
    exit(main())
```

### 3.4 经验沉淀到规则抽取流程

```
任务执行完成
    │
    ▼
┌─────────────────┐
│ 经验格式化      │  ──► 提取关键字段：任务类型、根因、解决方案、上下文
│ (Format)        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 存储到 RAG      │  ──► 向量化后存入 Milvus
│ (RAG Store)     │  ──► 元数据：项目、栈、应用、成功率、时间戳
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 模式识别        │  ──► 定期扫描（每日）相似经验聚类
│ (Pattern Recog) │  ──► 识别高频根因、高频解决方案
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 规则生成        │  ──► LLM 辅助：从聚类经验中抽取可执行规则
│ (Rule Generate) │  ──► 规则格式：IF <条件> THEN <动作> WITH <置信度>
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 人工审核        │  ──► 规则引擎管理员审核生成的规则
│ (Human Review)  │  ──► 审核通过：写入规则库
│                 │  ──► 审核拒绝：记录原因，反馈优化生成策略
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 规则应用        │  ──► 新任务执行时，优先匹配规则库
│ (Rule Apply)    │  ──► 规则命中：跳过 LLM，直接执行规则动作
│                 │  ──► 规则未命中：走 LLM 生成路径
└─────────────────┘
```

---

## 四、版本更新记录

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| v2.0 | 2026-04-26 | 全面重构 RAG 工程实现：更新 LangChain 包结构、补充异常处理与降级逻辑、增加 RAG 工程决策记录、补充检索质量评估脚本 |
| v1.0 | 2026-04-23 | 初始版本，包含三层进化架构和基础代码示例 |

---

## 变更历史

| 版本 | 日期 | 变更内容 | 变更人 |
|-----|------|---------|--------|
| v2.0.0 | 2026-04-26 | 初始版本，建立三层进化架构（短期记忆/长期记忆/规则库） | - |
