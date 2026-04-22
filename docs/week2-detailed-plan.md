# 第 2 周详细学习计划 - 测试代码生成智能体

**版本：** 2026.04 (Quality Agent Edition)  
**周期：** 第 2 周（6 天，共 30 小时）  
**主题：** 代码理解、测试框架自动生成、测试用例实现  
**质量智能体能力：** Level 1: 测试生成  
**代表项目：** Test Generator Agent

---

## 📅 本周学习目标

### 知识目标
- [ ] 理解测试代码生成智能体的核心能力（代码理解、框架生成、用例实现）
- [ ] 掌握 AST 解析与代码依赖分析原理
- [ ] 理解测试框架选型与适配策略
- [ ] 掌握测试用例设计方法（边界值、异常场景、集成测试）
- [ ] 理解 RAG 在测试生成中的应用
- [ ] 了解 Test-Agent（蚂蚁 CodeFuse）架构与集成方式
- [ ] 了解 Superpowers Framework 的 TDD 方法论参考价值

### 技能目标
- [ ] 能使用 AST 解析 Python/TypeScript 代码
- [ ] 能为代码自动生成测试框架（pytest/Jest 等）
- [ ] 能生成边界值测试、异常场景测试用例
- [ ] 能实现简单的 Test Generator Agent 原型
- [ ] 能为生成的测试代码编写质量评估报告
- [ ] 能评估 Test-Agent 集成可行性并完成基础集成

### 产出目标
- [ ] Test Generator Agent 原型系统
- [ ] 测试框架生成代码（支持 pytest/Jest）
- [ ] 生成的测试用例覆盖率 80%+
- [ ] 测试代码生成智能体学习笔记（6 篇）

---

## ⏰ 每日详细计划

---

## 第 7 天（周一）- 数据流水线的测试点

**学习目标：** 理解数据管线测试的核心挑战，设计测试策略

### 📚 上午：理论学习（2.5 小时）

#### 09:00-10:00 数据管线架构与测试挑战（1 小时）

**学习内容：**

```markdown
# 典型数据管线架构

原始数据 → 数据清洗 → 特征工程 → 模型训练 → 评估 → 部署
    ↓          ↓           ↓           ↓         ↓       ↓
 验证点 1    验证点 2    验证点 3    验证点 4   验证点 5  验证点 6

# 数据管线测试挑战

## 1. 数据依赖性
- 测试需要真实数据，但真实数据可能敏感
- 合成数据可能无法覆盖真实场景

## 2. 非确定性
- 随机初始化导致结果不同
- 并行处理顺序不确定

## 3. 资源密集
- 大数据集加载耗时
- 完整的端到端测试成本高

## 4. 版本管理
- 数据版本与代码版本需要同步
- 特征存储的管理
```

**阅读材料：**
```
必读文章：
1. "Testing Data Pipelines" (Towards Data Science)
   - https://towardsdatascience.com/data-pipeline-testing

2. "Data Engineering Testing Best Practices"
   - https://multithreaded.stitchfix.com/blog/2020/data-engineering-testing/

学习笔记要点：
- 列出数据管线测试的 5 个关键验证点
- 理解每个验证点的测试目标
- 记录常见陷阱和解决方案
```

**完成标志：**
- [ ] 完成数据管线测试架构图
- [ ] 能向他人解释测试挑战

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 输入/输出校验策略（1 小时）

**学习内容：**

```python
# 数据管线输入校验

def validate_input(df: pd.DataFrame) -> dict:
    """验证输入数据"""
    issues = []
    
    # 1. 结构校验
    expected_columns = ["user_id", "age", "gender", "income"]
    missing_cols = set(expected_columns) - set(df.columns)
    if missing_cols:
        issues.append(f"缺少列：{missing_cols}")
    
    # 2. 类型校验
    if not pd.api.types.is_numeric_dtype(df["age"]):
        issues.append("age 列应为数值类型")
    
    # 3. 范围校验
    if df["age"].min() < 0 or df["age"].max() > 120:
        issues.append("age 超出合理范围 [0, 120]")
    
    # 4. 完整性校验
    missing_rate = df.isnull().sum() / len(df)
    if (missing_rate > 0.5).any():
        issues.append(f"缺失率超过 50%: {missing_rate[missing_rate > 0.5].index.tolist()}")
    
    return {
        "is_valid": len(issues) == 0,
        "issues": issues
    }

# 数据管线输出校验

def validate_output(output_df: pd.DataFrame) -> dict:
    """验证输出数据"""
    issues = []
    
    # 1. 输出格式校验
    if not isinstance(output_df, pd.DataFrame):
        issues.append("输出应为 DataFrame 格式")
    
    # 2. 数据量校验
    if len(output_df) == 0:
        issues.append("输出数据为空")
    
    # 3. 预测结果校验
    if "prediction" in output_df.columns:
        unique_values = output_df["prediction"].nunique()
        if unique_values == 1:
            issues.append("预测结果单一，可能模型异常")
    
    return {
        "is_valid": len(issues) == 0,
        "issues": issues
    }
```

**练习：**
```python
# 为你的数据管线设计输入/输出校验
# 1. 定义期望的列名和类型
# 2. 设计边界值校验规则
# 3. 实现缺失值检测逻辑
# 4. 编写对应的测试用例
```

**完成标志：**
- [ ] 理解输入/输出校验的关键点
- [ ] 能设计完整的校验规则

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 实现数据管线测试（2 小时）

**任务：** 为完整数据管线编写测试

```python
# src/data_pipeline.py
import pandas as pd
from sklearn.preprocessing import StandardScaler

class DataPipeline:
    """完整数据管线"""
    
    def __init__(self):
        self.scaler = StandardScaler()
        self._is_fitted = False
    
    def load(self, file_path: str) -> pd.DataFrame:
        """加载数据"""
        return pd.read_csv(file_path)
    
    def clean(self, df: pd.DataFrame) -> pd.DataFrame:
        """数据清洗"""
        df = df.copy()
        
        # 删除缺失值过多的列
        missing_rate = df.isnull().sum() / len(df)
        cols_to_drop = missing_rate[missing_rate > 0.5].index
        df = df.drop(columns=cols_to_drop)
        
        # 填充剩余缺失值
        for col in df.select_dtypes(include=["number"]).columns:
            df[col] = df[col].fillna(df[col].median())
        
        for col in df.select_dtypes(include=["object"]).columns:
            df[col] = df[col].fillna(df[col].mode()[0] if len(df[col].mode()) > 0 else "Unknown")
        
        return df
    
    def transform(self, df: pd.DataFrame, feature_columns: list) -> pd.DataFrame:
        """特征变换"""
        df = df.copy()
        
        # 标准化数值特征
        df[feature_columns] = self.scaler.fit_transform(df[feature_columns])
        self._is_fitted = True
        
        return df
    
    def run(self, file_path: str, feature_columns: list) -> pd.DataFrame:
        """运行完整管线"""
        df = self.load(file_path)
        df = self.clean(df)
        df = self.transform(df, feature_columns)
        return df
```

```python
# tests/test_data_pipeline.py
import pytest
import pandas as pd
import numpy as np
from src.data_pipeline import DataPipeline

@pytest.fixture
def sample_data(tmp_path):
    """创建样本数据文件"""
    csv_content = """user_id,age,gender,income,score
1,25,M,50000,85
2,30,F,60000,90
3,,M,55000,88
4,35,F,,92
5,40,M,70000,
"""
    csv_file = tmp_path / "sample.csv"
    csv_file.write_text(csv_content)
    return str(csv_file)

@pytest.fixture
def pipeline():
    """创建管线的 fixture"""
    return DataPipeline()

def test_load(pipeline, sample_data):
    """测试数据加载"""
    df = pipeline.load(sample_data)
    
    assert isinstance(df, pd.DataFrame)
    assert len(df) == 5
    assert list(df.columns) == ["user_id", "age", "gender", "income", "score"]

def test_clean_handles_missing_values(pipeline, sample_data):
    """测试清洗处理缺失值"""
    df = pipeline.load(sample_data)
    cleaned = pipeline.clean(df)
    
    # 检查缺失值已被填充
    assert cleaned.isnull().sum().sum() == 0
    
    # 检查数据行数不变
    assert len(cleaned) == len(df)

def test_transform_standardizes(pipeline, sample_data):
    """测试标准化变换"""
    df = pipeline.load(sample_data)
    df = pipeline.clean(df)
    transformed = pipeline.transform(df, ["age", "income"])
    
    # 标准化后均值应接近 0
    assert abs(transformed["age"].mean()) < 0.001
    assert abs(transformed["income"].mean()) < 0.001

def test_full_pipeline(pipeline, sample_data):
    """测试完整管线运行"""
    result = pipeline.run(sample_data, ["age", "income"])
    
    assert isinstance(result, pd.DataFrame)
    assert len(result) == 5
    assert "age" in result.columns
    assert "income" in result.columns
```

**完成标志：**
- [ ] 数据管线测试覆盖率达到 80%+
- [ ] 所有边界情况有测试覆盖

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志

```markdown
# 学习日志 - 第 7 天

## 今日学习内容
1. 数据管线架构与测试挑战
2. 输入/输出校验策略
3. 数据管线测试实现

## 关键收获
1. ________________
2. ________________
3. ________________

## 遇到的问题
- 问题：________________
- 解决方案：________________

## 代码提交
- 提交哈希：________________
- 提交信息：________________
```

**完成标志：**
- [ ] 学习日志提交
- [ ] 代码推送到 GitHub

---

### ✅ 第 7 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| 数据管线理论 | 完成架构图 | ☐ |
| 输入/输出校验 | 能设计校验规则 | ☐ |
| 数据管线测试 | 覆盖率 80%+ | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 8 天（周二）- 单元测试数据生成

**学习目标：** 掌握测试数据生成方法，能创建高质量 Mock 数据

### 📚 上午：测试数据生成（2.5 小时）

#### 09:00-10:00 Faker 库入门（1 小时）

**学习内容：**

```python
# 安装 Faker
# pip install faker

from faker import Faker
import pandas as pd

fake = Faker("zh_CN")  # 中文本地化

# 生成单个字段
print(f"姓名：{fake.name()}")
print(f"地址：{fake.address()}")
print(f"邮箱：{fake.email()}")
print(f"电话号码：{fake.phone_number()}")
print(f"公司：{fake.company()}")

# 生成数据集
def generate_user_data(n: int = 100) -> pd.DataFrame:
    """生成用户数据"""
    data = []
    for _ in range(n):
        user = {
            "user_id": fake.uuid4(),
            "name": fake.name(),
            "email": fake.email(),
            "phone": fake.phone_number(),
            "address": fake.address(),
            "age": fake.random_int(min=18, max=80),
            "income": fake.random_int(min=3000, max=50000),
            "registration_date": fake.date_between(start_date="-2y", end_date="today").isoformat()
        }
        data.append(user)
    return pd.DataFrame(data)

# 生成并保存
df = generate_user_data(100)
df.to_csv("data/generated_users.csv", index=False)
print(f"生成 {len(df)} 条用户数据")
```

**练习：**
```python
# 为你的项目生成测试数据
# 1. 定义数据结构（列名、类型）
# 2. 创建数据生成函数
# 3. 生成 1000 条测试数据
# 4. 保存到 CSV 文件
```

**完成标志：**
- [ ] 理解 Faker 核心 API
- [ ] 能生成符合业务逻辑的测试数据

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 pytest + Faker 集成（1 小时）

**学习内容：**

```python
# tests/conftest.py
import pytest
from faker import Faker
import pandas as pd

fake = Faker("zh_CN")

@pytest.fixture
def user_data_factory():
    """用户数据生成器 fixture"""
    def generate(n: int = 100, **kwargs):
        """生成用户数据
        
        Args:
            n: 数据条数
            **kwargs: 可以覆盖的字段
            
        Returns:
            pd.DataFrame
        """
        data = []
        for _ in range(n):
            user = {
                "user_id": fake.uuid4(),
                "name": fake.name(),
                "email": fake.email(),
                "age": fake.random_int(min=18, max=80),
                "income": fake.random_int(min=3000, max=50000)
            }
            user.update(kwargs)
            data.append(user)
        return pd.DataFrame(data)
    return generate

@pytest.fixture
def sample_users(user_data_factory):
    """样本用户数据"""
    return user_data_factory(100)

@pytest.fixture
def large_users(user_data_factory):
    """大数据集（用于性能测试）"""
    return user_data_factory(10000)

# tests/test_with_faker.py
def test_data_processing(sample_users):
    """使用 Faker 数据测试"""
    # sample_users 包含 100 条随机生成的用户数据
    
    # 验证数据结构
    assert len(sample_users) == 100
    assert list(sample_users.columns) == ["user_id", "name", "email", "age", "income"]
    
    # 测试数据处理逻辑
    # ...
```

**完成标志：**
- [ ] 理解 pytest fixture 与 Faker 集成
- [ ] 能创建可复用的数据生成 fixture

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 创建测试数据工厂（2 小时）

**任务：** 实现灵活的数据生成工厂

```python
# tests/data_factory.py
from faker import Faker
import pandas as pd
import numpy as np
from typing import Optional, Dict, Any

fake = Faker("zh_CN")

class TestDataFactory:
    """测试数据工厂"""
    
    @staticmethod
    def create_user_df(
        n: int = 100,
        missing_rate: float = 0.0,
        outlier_rate: float = 0.0,
        seed: Optional[int] = None
    ) -> pd.DataFrame:
        """
        创建用户测试数据
        
        Args:
            n: 数据条数
            missing_rate: 缺失率 (0-1)
            outlier_rate: 异常值率 (0-1)
            seed: 随机种子（确保可重复性）
            
        Returns:
            pd.DataFrame
        """
        if seed is not None:
            Faker.seed(seed)
            np.random.seed(seed)
        
        data = []
        for i in range(n):
            user = {
                "user_id": i + 1,
                "name": fake.name(),
                "age": fake.random_int(min=18, max=80),
                "income": fake.random_int(min=3000, max=50000),
                "score": round(np.random.normal(75, 15), 2)
            }
            
            # 注入缺失值
            if np.random.random() < missing_rate:
                field = np.random.choice(["age", "income", "score"])
                user[field] = np.nan
            
            # 注入异常值
            if np.random.random() < outlier_rate:
                field = np.random.choice(["age", "income"])
                if field == "age":
                    user["age"] = np.random.choice([-5, 200, 300])
                elif field == "income":
                    user["income"] = np.random.choice([-1000, 1000000])
            
            data.append(user)
        
        return pd.DataFrame(data)
    
    @staticmethod
    def create_transaction_df(
        n: int = 1000,
        fraud_rate: float = 0.05
    ) -> pd.DataFrame:
        """
        创建交易测试数据（含欺诈标签）
        
        Args:
            n: 交易笔数
            fraud_rate: 欺诈比例
            
        Returns:
            pd.DataFrame
        """
        data = []
        for i in range(n):
            transaction = {
                "transaction_id": f"TXN{i+1:06d}",
                "user_id": fake.random_int(min=1, max=1000),
                "amount": round(np.random.exponential(500), 2),
                "merchant": fake.company(),
                "timestamp": fake.date_time_this_year().isoformat(),
                "is_fraud": 1 if np.random.random() < fraud_rate else 0
            }
            data.append(transaction)
        
        return pd.DataFrame(data)

# tests/test_data_factory.py
import pytest
from tests.data_factory import TestDataFactory

def test_user_df_with_missing():
    """测试生成含缺失值的数据"""
    df = TestDataFactory.create_user_df(n=100, missing_rate=0.1)
    
    assert len(df) == 100
    assert df.isnull().sum().sum() > 0
    assert df.isnull().sum().sum() <= 100 * 0.15  # 允许一定波动

def test_user_df_with_outliers():
    """测试生成含异常值的数据"""
    df = TestDataFactory.create_user_df(n=1000, outlier_rate=0.1)
    
    # 检查异常值存在
    assert (df["age"] < 0).any() or (df["age"] > 120).any()

def test_transaction_df():
    """测试生成交易数据"""
    df = TestDataFactory.create_transaction_df(n=1000, fraud_rate=0.05)
    
    assert len(df) == 1000
    assert "is_fraud" in df.columns
    fraud_rate = df["is_fraud"].sum() / len(df)
    assert 0.03 <= fraud_rate <= 0.07  # 允许波动

def test_reproducibility():
    """测试可重复性"""
    df1 = TestDataFactory.create_user_df(n=100, seed=42)
    df2 = TestDataFactory.create_user_df(n=100, seed=42)
    
    pd.testing.assert_frame_equal(df1, df2)
```

**完成标志：**
- [ ] TestDataFactory 实现完成
- [ ] 支持缺失值、异常值注入
- [ ] 测试验证可重复性

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志

```markdown
# 学习日志 - 第 8 天

## 今日学习内容
1. Faker 库基础
2. pytest + Faker 集成
3. 测试数据工厂实现

## 关键收获
1. ________________
2. ________________
3. ________________

## 遇到的问题
- 问题：________________
- 解决方案：________________

## 代码提交
- 提交哈希：________________
- 提交信息：________________
```

**完成标志：**
- [ ] 学习日志提交
- [ ] 代码推送到 GitHub

---

### ✅ 第 8 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| Faker 入门 | 能生成测试数据 | ☐ |
| pytest 集成 | 创建可复用 fixture | ☐ |
| 测试数据工厂 | 支持缺失值/异常值注入 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 9 天（周三）- Deterministic Tests（确定性测试）

**学习目标：** 理解并实现可重复的确定性测试

### 📚 上午：确定性测试理论（2.5 小时）

#### 09:00-10:00 测试不确定性的来源（1 小时）

**学习内容：**

```markdown
# 测试不确定性的 5 大来源

## 1. 随机数生成
- ML 模型随机初始化
- 随机采样（train/test split）
- Dropout 等随机正则化

## 2. 并行处理
- 多线程执行顺序不确定
- 分布式计算结果顺序

## 3. 浮点数精度
- 不同硬件/平台的浮点运算差异
- 累积误差

## 4. 外部依赖
- 时间戳（datetime.now()）
- 文件系统状态
- 网络请求结果

## 5. 硬件差异
- GPU 并行计算的非确定性
- 不同 CPU 指令集

# 解决方案

## 1. 设置随机种子
```python
import random
import numpy as np
import torch

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
```

## 2. 固定训练/测试划分
```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42  # 固定 random_state
)
```

## 3. 使用确定性算法
```python
# PyTorch 确定性模式
torch.use_deterministic_algorithms(True)
torch.backends.cudnn.deterministic = True

# TensorFlow 确定性
tf.config.experimental.enable_op_determinism()
```

## 4. Mock 外部依赖
```python
from unittest.mock import patch
from datetime import datetime

with patch('datetime.datetime') as mock_datetime:
    mock_datetime.now.return_value = datetime(2024, 1, 1, 12, 0, 0)
```
```

**完成标志：**
- [ ] 理解测试不确定性的 5 大来源
- [ ] 能解释每种解决方案的适用场景

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 实现确定性测试（1 小时）

**学习内容：**

```python
# tests/conftest.py
import pytest
import random
import numpy as np

# 全局随机种子
TEST_SEED = 42

@pytest.fixture(autouse=True)
def set_global_seed():
    """自动设置随机种子的 fixture"""
    random.seed(TEST_SEED)
    np.random.seed(TEST_SEED)
    yield

# tests/test_deterministic.py
import pytest
import numpy as np
from sklearn.model_selection import train_test_split

def test_reproducible_split():
    """测试数据划分的可重复性"""
    X = np.random.rand(100, 10)
    y = np.random.randint(0, 2, 100)
    
    # 第一次划分
    X_train1, X_test1, y_train1, y_test1 = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # 第二次划分
    X_train2, X_test2, y_train2, y_test2 = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # 验证完全相同
    np.testing.assert_array_equal(X_train1, X_train2)
    np.testing.assert_array_equal(X_test1, X_test2)
    np.testing.assert_array_equal(y_train1, y_train2)
    np.testing.assert_array_equal(y_test1, y_test2)

def test_reproducible_model_training():
    """测试模型训练的可重复性"""
    from sklearn.linear_model import LogisticRegression
    from sklearn.datasets import make_classification
    
    # 生成数据
    X, y = make_classification(n_samples=100, n_features=10, random_state=42)
    
    # 第一次训练
    model1 = LogisticRegression(random_state=42, max_iter=1000)
    model1.fit(X, y)
    
    # 第二次训练
    model2 = LogisticRegression(random_state=42, max_iter=1000)
    model2.fit(X, y)
    
    # 验证系数相同
    np.testing.assert_array_almost_equal(model1.coef_, model2.coef_)
    np.testing.assert_almost_equal(model1.intercept_, model2.intercept_)
```

**完成标志：**
- [ ] 理解随机种子设置的重要性
- [ ] 能实现可重复的测试

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 创建确定性测试套件（2 小时）

**任务：** 为 ML 流水线创建完整的确定性测试

```python
# tests/test_deterministic_pipeline.py
import pytest
import numpy as np
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score

SEED = 42

@pytest.fixture(autouse=True)
def setup_seed():
    """每个测试前自动设置种子"""
    np.random.seed(SEED)

@pytest.fixture
def sample_data():
    """固定样本数据"""
    np.random.seed(SEED)
    X = np.random.rand(200, 10)
    y = np.random.randint(0, 2, 200)
    return X, y

def test_deterministic_preprocessing(sample_data):
    """测试预处理的可重复性"""
    X, y = sample_data
    
    # 运行预处理 3 次
    results = []
    for _ in range(3):
        X_processed = (X - X.mean(axis=0)) / X.std(axis=0)
        results.append(X_processed.copy())
    
    # 验证 3 次结果完全相同
    for i in range(1, len(results)):
        np.testing.assert_array_almost_equal(results[0], results[i])

def test_deterministic_training(sample_data):
    """测试模型训练的可重复性"""
    X, y = sample_data
    
    models = []
    for _ in range(3):
        model = LogisticRegression(random_state=SEED, max_iter=1000)
        model.fit(X, y)
        models.append(model)
    
    # 验证 3 次训练的模型完全相同
    for i in range(1, len(models)):
        np.testing.assert_array_almost_equal(models[0].coef_, models[i].coef_)
        np.testing.assert_almost_equal(models[0].intercept_, models[i].intercept_)

def test_deterministic_cross_validation(sample_data):
    """测试交叉验证的可重复性"""
    X, y = sample_data
    
    model = LogisticRegression(random_state=SEED)
    
    # 运行 3 次交叉验证
    scores_list = []
    for _ in range(3):
        scores = cross_val_score(model, X, y, cv=5)
        scores_list.append(scores)
    
    # 验证结果相同
    for i in range(1, len(scores_list)):
        np.testing.assert_array_almost_equal(scores_list[0], scores_list[i])

def test_prediction_determinism(sample_data):
    """测试预测的可重复性"""
    X, y = sample_data
    
    model = LogisticRegression(random_state=SEED)
    model.fit(X, y)
    
    # 多次预测
    predictions_list = []
    for _ in range(3):
        predictions = model.predict(X)
        predictions_list.append(predictions.copy())
    
    # 验证预测结果相同
    for i in range(1, len(predictions_list)):
        np.testing.assert_array_equal(predictions_list[0], predictions_list[i])
```

**完成标志：**
- [ ] 所有确定性测试通过
- [ ] 测试涵盖预处理、训练、预测全流程

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志

```markdown
# 学习日志 - 第 9 天

## 今日学习内容
1. 测试不确定性来源
2. 随机种子设置
3. 确定性测试实现

## 关键收获
1. ________________
2. ________________
3. ________________

## 遇到的问题
- 问题：________________
- 解决方案：________________

## 代码提交
- 提交哈希：________________
- 提交信息：________________
```

**完成标志：**
- [ ] 学习日志提交
- [ ] 代码推送到 GitHub

---

### ✅ 第 9 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| 不确定性理论 | 理解 5 大来源 | ☐ |
| 随机种子设置 | 能正确设置 | ☐ |
| 确定性测试 | 全套测试通过 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 10 天（周四）- DVC 数据版本控制

**学习目标：** 掌握 DVC 基础，实现数据版本管理

### 📚 上午：DVC 入门（2.5 小时）

#### 09:00-10:00 DVC 核心概念（1 小时）

**学习内容：**

```markdown
# DVC（Data Version Control）核心概念

## 什么是 DVC？
- 数据版本的 Git
- 管理大型数据文件
- 追踪数据与代码的关系

## 核心概念

### 1. .dvc 文件
- 存储数据文件的元数据
- 包含文件哈希值
- 轻量级，可提交到 Git

### 2. 远程存储（Remote）
- 存储实际数据文件
- 支持本地、S3、GCS、Azure 等

### 3. Pipeline（管线）
- 定义数据处理流程
- 追踪依赖和输出
- 可重现执行

## 基本命令

```bash
# 初始化 DVC
dvc init

# 追踪数据文件
dvc add data/raw_data.csv

# 提交到 Git
git add data/raw_data.dvc .gitignore
git commit -m "Add raw data"

# 配置远程存储
dvc remote add -d myremote s3://mybucket/dvc-storage

# 推送数据到远程
dvc push

# 拉取数据
dvc pull
```
```

**阅读材料：**
```
官方文档：
1. DVC Get Started: https://dvc.org/doc/start
2. DVC Command Reference: https://dvc.org/doc/command-reference

学习笔记要点：
- 理解.dvc 文件的作用
- 掌握基本命令流程
- 了解远程存储配置
```

**完成标志：**
- [ ] 理解 DVC 核心价值
- [ ] 记住基本命令

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 DVC 实战配置（1 小时）

**学习内容：**

```bash
# 1. 安装 DVC
# pip install dvc
# 或（带 S3 支持）
# pip install dvc[s3]

# 2. 初始化 DVC（在项目根目录）
cd /Users/bohaiqing/opensource/git/ai-testing-journey
dvc init

# 查看生成的文件
ls -la .dvc/
cat .dvc/config

# 3. 配置 Git 提交
git add .dvc .dvcignore
git commit -m "Initialize DVC"

# 4. 追踪数据文件
# 创建示例数据
echo "name,age,income" > data/users.csv
echo "Alice,25,50000" >> data/users.csv
echo "Bob,30,60000" >> data/users.csv

# 使用 DVC 追踪
dvc add data/users.csv

# 查看生成的.dvc 文件
cat data/users.csv.dvc

# 5. Git 提交
git add data/users.csv.dvc data/.gitignore
git commit -m "Add users data with DVC"

# 6. 配置本地远程存储（用于演示）
mkdir -p ../dvc-remote
dvc remote add -d local_remote ../dvc-remote

# 7. 推送数据
dvc push
```

**完成标志：**
- [ ] DVC 初始化完成
- [ ] 成功追踪数据文件
- [ ] 配置远程存储

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 实现数据版本管理工作流（2 小时）

**任务：** 在项目中集成 DVC

```bash
# 1. 创建数据版本分支
git checkout -b feat/data-version-v1

# 2. 添加训练数据
# 运行数据生成脚本
python scripts/generate_training_data.py --output data/training_v1.csv

# 使用 DVC 追踪
dvc add data/training_v1.csv

# 3. 提交数据版本
git add data/training_v1.csv.dvc
git commit -m "Add training data v1.0"

# 4. 推送到远程
dvc push

# 5. 创建 pipeline 文件（dvc.yaml）
cat > dvc.yaml << 'EOF'
stages:
  prepare_data:
    cmd: python src/prepare_data.py
    deps:
    - data/training_v1.csv
    - src/prepare_data.py
    outs:
    - data/processed_data.csv
  
  train_model:
    cmd: python src/train.py
    deps:
    - data/processed_data.csv
    - src/train.py
    outs:
    - models/model_v1.pkl
EOF

# 6. 运行 pipeline
dvc repro

# 7. 提交 pipeline
git add dvc.yaml dvc.lock
git commit -m "Add DVC pipeline"

# 8. 推送到 Git
git push origin feat/data-version-v1
```

**完成标志：**
- [ ] DVC 数据追踪完成
- [ ] Pipeline 配置完成
- [ ] 数据推送到远程

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志

```markdown
# 学习日志 - 第 10 天

## 今日学习内容
1. DVC 核心概念
2. DVC 基本命令
3. 数据版本管理工作流

## 关键收获
1. ________________
2. ________________
3. ________________

## 遇到的问题
- 问题：________________
- 解决方案：________________

## 代码提交
- 提交哈希：________________
- 提交信息：________________
```

**完成标志：**
- [ ] 学习日志提交
- [ ] DVC 配置完成
- [ ] 代码推送到 GitHub

---

### ✅ 第 10 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| DVC 理论 | 理解核心概念 | ☐ |
| DVC 配置 | 初始化 + 远程配置 | ☐ |
| 数据版本管理 | 成功追踪数据 | ☐ |
| Pipeline | dvc.yaml 配置完成 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 11 天（周五）- Great Expectations 数据验证

**学习目标：** 掌握 Great Expectations 基础，定义数据验证规则

### 📚 上午：Great Expectations 入门（2.5 小时）

#### 09:00-10:00 GE 核心概念（1 小时）

**学习内容：**

```markdown
# Great Expectations（GE）核心概念

## 什么是 Great Expectations？
- 数据质量验证框架
- 定义数据的"期望"（Expectations）
- 自动生成数据文档

## 核心概念

### 1. Expectation（期望）
- 数据应满足的条件
- 例如：列不应为空、值应在范围内

### 2. Expectation Suite（期望套件）
- 一组期望的集合
- 针对特定数据集

### 3. Validation（验证）
- 检查数据是否满足期望
- 生成验证报告

### 4. Data Docs（数据文档）
- 自动生成的文档
- 包含数据质量和期望

## 常用 Expectations

```python
# 列级别期望
expect_column_to_exist("user_id")
expect_column_values_to_not_be_null("age")
expect_column_values_to_be_between("age", min_value=0, max_value=120)

# 分布期望
expect_column_values_to_be_unique("user_id")
expect_column_values_to_be_in_set("gender", ["M", "F"])

# 表格期望
expect_table_row_count_to_be_between(min_value=100, max_value=10000)
```
```

**阅读材料：**
```
官方文档：
1. Getting Started: https://docs.greatexpectations.io/docs/get_started/quickstart
2. Expectation Gallery: https://greatexpectations.io/expectations

学习笔记要点：
- 理解 3 种期望类型（列/分布/表格）
- 记住 10 个常用期望
- 了解验证流程
```

**完成标志：**
- [ ] 理解 GE 核心价值
- [ ] 记住常用期望

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 GE 快速入门（1 小时）

**学习内容：**

```bash
# 1. 安装 Great Expectations
# pip install great_expectations

# 2. 初始化 GE
great_expectations init

# 3. 创建 Expectation Suite
great_expectations suite new

# 或（批处理模式）
great_expectations suite new --suite_name user_data_suite

# 4. 验证数据
great_expectations checkpoint new

# 或使用 Python API
```

```python
# examples/great_expectations_example.py
import great_expectations as gx
import pandas as pd

# 加载数据
df = pd.read_csv("data/users.csv")

# 创建 Great Expectations 上下文
context = gx.get_context()

# 创建或获取 Expectation Suite
suite_name = "user_data_suite"
suite = context.suites.add_or_update(
    gx.ExpectationSuite(name=suite_name)
)

# 添加期望
suite.add_expectation(
    gx.expectations.ExpectColumnToExist(column="user_id")
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToNotBeNull(column="age")
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeBetween(
        column="age", min_value=0, max_value=120
    )
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeUnique(column="user_id")
)

# 保存 Suite
context.suites.add(suite)

# 验证数据
validator = gx.Validator(
    batch_data=df,
    expectation_suite=suite
)

results = validator.validate()
print(f"验证结果：{'通过' if results.success else '失败'}")
print(f"失败数量：{results.statistics['failed_expectations']}")
```

**完成标志：**
- [ ] 理解 GE 初始化流程
- [ ] 能用 Python API 创建期望

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 实现数据验证规则（2 小时）

**任务：** 为项目创建完整的数据验证规则

```python
# src/data_validation_rules.py
import great_expectations as gx
import pandas as pd

def create_user_data_suite():
    """创建用户数据验证套件"""
    suite = gx.ExpectationSuite(name="user_data_suite")
    
    # 列存在性期望
    suite.add_expectation(
        gx.expectations.ExpectColumnToExist(column="user_id")
    )
    suite.add_expectation(
        gx.expectations.ExpectColumnToExist(column="name")
    )
    suite.add_expectation(
        gx.expectations.ExpectColumnToExist(column="age")
    )
    suite.add_expectation(
        gx.expectations.ExpectColumnToExist(column="income")
    )
    
    # 非空期望
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToNotBeNull(column="user_id")
    )
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToNotBeNull(column="age")
    )
    
    # 值范围期望
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToBeBetween(
            column="age", min_value=0, max_value=120
        )
    )
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToBeBetween(
            column="income", min_value=0, max_value=10000000
        )
    )
    
    # 唯一性期望
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToBeUnique(column="user_id")
    )
    
    # 类型期望
    suite.add_expectation(
        gx.expectations.ExpectColumnValuesToBeOfType(
            column="age", type_="int"
        )
    )
    
    return suite

def validate_user_data(df: pd.DataFrame, suite: gx.ExpectationSuite) -> dict:
    """
    验证用户数据
    
    Returns:
        dict: 验证结果
    """
    context = gx.get_context()
    
    validator = gx.Validator(
        batch_data=df,
        expectation_suite=suite,
        context=context
    )
    
    results = validator.validate()
    
    return {
        "success": results.success,
        "failed_expectations": results.statistics['failed_expectations'],
        "total_expectations": results.statistics['evaluated_expectations'],
        "details": [
            {
                "expectation": exp.expectation_type,
                "column": exp.kwargs.get('column'),
                "success": exp.success,
                "exception_info": exp.exception_info
            }
            for exp in results.results
        ]
    }

# tests/test_ge_validation.py
import pytest
import pandas as pd
from src.data_validation_rules import create_user_data_suite, validate_user_data

@pytest.fixture
def valid_user_data():
    """有效的用户数据"""
    return pd.DataFrame({
        "user_id": [1, 2, 3, 4, 5],
        "name": ["Alice", "Bob", "Charlie", "David", "Eve"],
        "age": [25, 30, 35, 40, 45],
        "income": [50000, 60000, 70000, 80000, 90000]
    })

@pytest.fixture
def invalid_user_data():
    """无效的用户数据（含异常值）"""
    return pd.DataFrame({
        "user_id": [1, 2, 3, 4, 5],
        "name": ["Alice", "Bob", "Charlie", "David", "Eve"],
        "age": [25, -5, 200, 40, 45],  # 负年龄和异常值
        "income": [50000, None, 70000, 80000, 90000]  # 缺失值
    })

def test_valid_data_passes(validation, valid_user_data):
    """测试有效数据通过验证"""
    suite = create_user_data_suite()
    results = validate_user_data(valid_user_data, suite)
    
    assert results["success"] == True
    assert results["failed_expectations"] == 0

def test_invalid_data_fails(invalid_user_data):
    """测试无效数据失败"""
    suite = create_user_data_suite()
    results = validate_user_data(invalid_user_data, suite)
    
    assert results["success"] == False
    assert results["failed_expectations"] > 0
    
    # 验证具体哪些期望失败
    failed_expectations = [e for e in results["details"] if not e["success"]]
    assert any("ExpectColumnValuesToBeBetween" in e["expectation"] for e in failed_expectations)
```

**完成标志：**
- [ ] Great Expectations 套件创建完成
- [ ] 验证函数可用
- [ ] 测试用例通过

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志

```markdown
# 学习日志 - 第 11 天

## 今日学习内容
1. Great Expectations 核心概念
2. Expectation 类型
3. 数据验证规则实现

## 关键收获
1. ________________
2. ________________
3. ________________

## 遇到的问题
- 问题：________________
- 解决方案：________________

## 代码提交
- 提交哈希：________________
- 提交信息：________________
```

**完成标志：**
- [ ] 学习日志提交
- [ ] GE 验证规则可用

---

### ✅ 第 11 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| GE 理论 | 理解核心概念 | ☐ |
| Expectation Suites | 创建验证套件 | ☐ |
| 数据验证 | 验证函数可用 | ☐ |
| 测试用例 | 所有测试通过 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 12 天（周六）- 周项目整合

**学习目标：** 整合本周所有模块，形成完整的数据管线测试方案

### 💻 全天：项目整合（5 小时）

#### 09:00-11:00 整合数据管线测试（2 小时）

**任务：** 创建完整的测试方案

```python
# tests/test_complete_data_pipeline.py
"""
完整数据管线测试套件

整合：
1. 数据加载测试
2. 数据验证测试（GE）
3. 数据清洗测试
4. 特征工程测试
5. 确定性测试
6. DVC 版本追踪
"""

import pytest
import pandas as pd
import numpy as np
from src.data_pipeline import DataPipeline
from src.data_validation_rules import create_user_data_suite, validate_user_data
from tests.data_factory import TestDataFactory

SEED = 42

@pytest.fixture(autouse=True)
def setup_seed():
    """全局随机种子"""
    np.random.seed(SEED)

@pytest.fixture
def test_data():
    """生成测试数据"""
    return TestDataFactory.create_user_df(
        n=100,
        missing_rate=0.05,
        seed=SEED
    )

@pytest.fixture
def ge_suite():
    """Great Expectations 套件"""
    return create_user_data_suite()

def test_pipeline_with_validation(test_data, ge_suite):
    """测试完整数据管线与验证"""
    # 1. GE 预验证
    validation_results = validate_user_data(test_data, ge_suite)
    # 注意：测试数据可能不完全符合生产期望
    
    # 2. 运行数据管线
    pipeline = DataPipeline()
    
    # 加载数据
    df = test_data  # 使用生成的测试数据
    assert isinstance(df, pd.DataFrame)
    
    # 清洗数据
    cleaned = pipeline.clean(df)
    assert cleaned.isnull().sum().sum() == 0
    
    # 特征变换
    feature_cols = ["age", "income"]
    transformed = pipeline.transform(cleaned, feature_cols)
    
    # 验证标准化
    for col in feature_cols:
        assert abs(transformed[col].mean()) < 0.001
    
    assert len(transformed) == len(df)

def test_deterministic_pipeline(test_data):
    """测试管线确定性"""
    results = []
    
    for _ in range(3):
        pipeline = DataPipeline()
        cleaned = pipeline.clean(test_data)
        transformed = pipeline.transform(cleaned, ["age", "income"])
        results.append(transformed.copy())
    
    # 验证 3 次运行结果相同
    for i in range(1, len(results)):
        pd.testing.assert_frame_equal(results[0], results[i])

def test_edge_cases():
    """测试边界情况"""
    # 空数据框
    empty_df = pd.DataFrame()
    pipeline = DataPipeline()
    with pytest.raises(Exception):
        pipeline.clean(empty_df)
    
    # 单行数据
    single_row = pd.DataFrame({
        "user_id": [1],
        "age": [25],
        "income": [50000]
    })
    cleaned = pipeline.clean(single_row)
    assert len(cleaned) == 1

def report_generation(test_data):
    """生成测试报告"""
    report = []
    report.append("=" * 60)
    report.append("数据管线测试报告 - 第 2 周")
    report.append("=" * 60)
    
    report.append(f"\n测试数据量：{len(test_data)} 条")
    report.append(f"\n缺失率：{test_data.isnull().sum().sum() / test_data.size:.2%}")
    
    # 运行测试
    pipeline = DataPipeline()
    cleaned = pipeline.clean(test_data)
    
    report.append(f"\n清洗后缺失率：{cleaned.isnull().sum().sum() / cleaned.size:.2%}")
    report.append(f"\n数据保留率：{len(cleaned) / len(test_data):.2%}")
    
    print("\n".join(report))
```

**完成标志：**
- [ ] 完整测试套件编写完成
- [ ] 所有测试通过

---

#### 11:00-11:15 ☕ 休息（15 分钟）

---

#### 11:15-12:00 测试覆盖率检查（45 分钟）

**任务：** 生成并分析覆盖率报告

```bash
# 运行完整测试套件
pytest tests/ -v --cov=src --cov-report=html --cov-report=term

# 目标：
# - 整体覆盖率 >= 80%
# - 核心模块（data_pipeline.py）覆盖率 >= 90%
```

**完成标志：**
- [ ] HTML 报告生成
- [ ] 覆盖率达到目标

---

#### 12:00-13:00 🍽️ 午餐休息（1 小时）

---

#### 13:00-14:30 项目文档编写（1.5 小时）

**任务：** 更新项目 README 和周文档

```markdown
# AI Testing Journey - 第 2 周项目总结

## 本周学习内容
1. 数据管线测试策略
2. 测试数据生成（Faker）
3. 确定性测试
4. DVC 数据版本控制
5. Great Expectations 数据验证

## 新增模块
- `src/data_pipeline.py` - 完整数据管线
- `src/data_validation_rules.py` - GE 验证规则
- `tests/data_factory.py` - 测试数据工厂

## 技术亮点
- ✅ 数据管线测试覆盖率 80%+
- ✅ DVC 版本追踪配置完成
- ✅ Great Expectations 验证规则
- ✅ 确定性测试确保可重复

## 使用方法

### 运行数据管线
```bash
python src/main.py --data data/sample.csv
```

### 运行测试
```bash
pytest tests/test_data_pipeline.py -v
```

### DVC 操作
```bash
dvc pull          # 拉取数据
dvc push          # 推送数据
dvc repro         # 重现管线
```

### GE 验证
```bash
great_expectations checkpoint run
```

## 测试报告
- 总测试数：XX 个
- 通过率：100%
- 覆盖率：XX%
```

**完成标志：**
- [ ] README 更新
- [ ] 周总结文档完成

---

#### 14:30-15:00 ☕ 休息（30 分钟）

---

#### 15:00-16:00 最终验证（1 小时）

**任务：** 完整运行和验证

```bash
# 1. 运行所有测试
pytest tests/ -v

# 2. 生成覆盖率报告
pytest --cov=src --cov-report=html

# 3. 运行 GE 验证
python src/main.py --validate

# 4. DVC 状态检查
dvc status

# 5. Git 提交
git add .
git commit -m "Week 2 complete: Data pipeline testing full implementation"
git push
```

**完成标志：**
- [ ] 所有测试通过
- [ ] 代码推送

---

#### 16:00-17:00 周总结与反思（1 小时）

**任务：** 完成周总结文档

```markdown
# 第 2 周学习总结

## 学习内容回顾
1. 数据管线测试策略
2. Faker 测试数据生成
3. 确定性测试
4. DVC 数据版本控制
5. Great Expectations 验证

## 完成的产出
- [ ] GitHub 仓库：https://github.com/...
- [ ] 代码行数：XXX 行
- [ ] 测试覆盖率：XX%
- [ ] 测试用例数：XX 个
- [ ] DVC 配置：✅
- [ ] GE 验证规则：✅

## 技术收获
1. ________________
2. ________________
3. ________________

## 遇到的挑战
1. ________________
   解决方案：________________
2. ________________
   解决方案：________________

## 改进空间
1. ________________
2. ________________

## 第 3 周计划
- 重点学习：模型评估与基线对比
- 目标完成：___________
- 预计投入：25-30 小时
```

**完成标志：**
- [ ] 周总结完成
- [ ] 第 3 周计划制定

---

### ✅ 第 12 天验收清单（本周最终验收）

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| 完整测试套件 | 所有测试通过 | ☐ |
| 测试覆盖率 | 80%+ | ☐ |
| 项目文档 | README 更新 | ☐ |
| 最终验证 | 所有检查通过 | ☐ |
| 周总结 | 文档提交 | ☐ |
| Git 提交 | 代码推送 | ☐ |

---

## 📊 第 2 周完成度自检表

### 知识掌握

| 知识点 | 理解程度 | 自评 |
|--------|---------|------|
| 数据管线测试 | 能设计完整测试策略 | ☐☐☐☐☐ |
| Faker 数据生成 | 能生成测试数据 | ☐☐☐☐☐ |
| 确定性测试 | 理解并实现 | ☐☐☐☐☐ |
| DVC | 能配置和使用 | ☐☐☐☐☐ |
| Great Expectations | 能创建验证规则 | ☐☐☐☐☐ |

### 技能达成

| 技能 | 目标 | 实际 | 状态 |
|------|------|------|------|
| 数据管线测试 | 覆盖率 80%+ | ☐% | ☐ |
| Faker 使用 | 生成测试数据 | ☐ | ☐ |
| 确定性测试 | 全部通过 | ☐ | ☐ |
| DVC 配置 | 数据可追踪 | ☐ | ☐ |
| GE 验证 | 规则可用 | ☐ | ☐ |

### 产出检查

| 产出物 | 要求 | 完成 |
|--------|------|------|
| GitHub 仓库 | 可访问 | ☐ |
| 数据管线代码 | 可运行 | ☐ |
| 测试套件 | 全部通过 | ☐ |
| 覆盖率报告 | 80%+ | ☐ |
| 学习日志 | 6 篇完成 | ☐ |
| 周总结 | 提交 | ☐ |

---

## 🎯 第 3 周预习

### 预习内容
- 阅读模型评估指标文档
- 了解交叉验证原理
- 复习统计学基础（均值、方差、置信区间）

### 预习时间
- 预计：2-3 小时（周日）

### 预习目标
- 理解准确率、精确率、召回率、F1 的区别
- 了解 k 折交叉验证流程
- 熟悉假设检验概念

---

**第 2 周学习计划完成！** 🎉

准备好迎接第 3 周的挑战了吗？继续加油！💪
