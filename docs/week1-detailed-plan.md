# 第 1 周详细学习计划 - 质量智能体认知与数据感知能力

**版本：** 2026.04 (Quality Agent Edition)  
**周期：** 第 1 周（6 天，共 30 小时）  
**主题：** 理解质量智能体、数据采集与治理  
**质量智能体能力：** Level 1: 数据感知  
**代表项目：** Quality Data Agent

---

## 📅 本周学习目标

### 知识目标
- [ ] 理解质量智能体的诞生背景与核心理念
- [ ] 区分「效率工具」vs「质量智能体」的本质差异
- [ ] 掌握质量智能体五维能力模型（感知、认知、决策、执行、进化）
- [ ] 理解质量数据采集与治理的核心概念

### 技能目标
- [ ] 搭建完整的 Python 开发环境
- [ ] 熟练使用 pytest 编写测试用例
- [ ] 实现一个完整的数据采集与验证流水线
- [ ] 构建 Quality Data Agent 基础能力

### 产出目标
- [ ] GitHub 仓库创建完成
- [ ] 数据采集与验证流水线代码（含测试）
- [ ] 测试覆盖率报告（70%+）
- [ ] 质量智能体认知学习笔记

---

## ⏰ 每日详细计划

---

## 第 1 天（周一）- AI 测试概述与环境搭建

**学习目标：** 理解 AI 测试基础概念，完成开发环境搭建

### 📚 上午：理论学习（2.5 小时）

#### 09:00-09:30 AI 测试概述（30 分钟）

**学习内容：**
- 什么是 AI 测试？为什么需要专门的 AI 测试方法？
- AI 测试与传统软件测试的核心区别

**阅读材料：**
```
必读文章（在线）：
1. "AI Testing: What It Is and How to Do It" 
   - https://www.applause.com/learn/ai-testing-101
   
2. "Testing Machine Learning Systems" (Google Testing Blog)
   - https://testing.googleblog.com/2020/01/testing-machine-learning-systems.html

学习笔记要点：
- 列出 AI 系统 vs 传统系统的 5 个关键区别
- 理解"不确定性"对测试的影响
```

**完成标志：**
- [ ] 完成 500 字学习笔记
- [ ] 能向他人解释 AI 测试的必要性

---

#### 09:30-10:00 AI 测试核心概念（30 分钟）

**学习内容：**
- 数据质量（Data Quality）
- 评估指标（Accuracy、Precision、Recall、F1、AUC）
- 实验可重复性（Reproducibility）

**阅读材料：**
```
必读：
1. "Metrics for Machine Learning" 
   - https://developers.google.com/machine-learning/crash-course/classification/metrics

学习笔记要点：
- 每个评估指标的定义和适用场景
- 为什么需要多个指标而不是单一指标
```

**完成标志：**
- [ ] 完成评估指标对照表
- [ ] 能解释 Precision vs Recall 的权衡

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 Python 环境搭建（1 小时）

**实操任务：**

```bash
# 1. 检查 Python 版本（需要 3.10+）
python3 --version

# 2. 创建项目目录
mkdir ai-testing-journey
cd ai-testing-journey

# 3. 创建虚拟环境
python3 -m venv venv

# 4. 激活虚拟环境
# macOS/Linux:
source venv/bin/activate
# Windows:
# venv\Scripts\activate

# 5. 安装核心依赖
pip install pytest pytest-cov scikit-learn pandas numpy

# 6. 验证安装
pytest --version
python -c "import sklearn; print(sklearn.__version__)"
```

**完成标志：**
- [ ] Python 3.10+ 安装成功
- [ ] 虚拟环境创建并激活
- [ ] 所有依赖包安装成功
- [ ] 运行 `pytest --version` 无报错

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-15:00 Git 与 GitHub 配置（1 小时）

**实操任务：**

```bash
# 1. 配置 Git（如未配置）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 2. 初始化 Git 仓库
git init

# 3. 创建 .gitignore
cat > .gitignore << EOF
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
venv/
env/
ENV/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Test
.pytest_cache/
.coverage
htmlcov/
EOF

# 4. 创建初始提交
echo "# AI Testing Journey" > README.md
git add .
git commit -m "Initial commit: Project setup"

# 5. 创建 GitHub 仓库并推送
# 在 GitHub.com 创建新仓库后：
git remote add origin https://github.com/YOUR_USERNAME/ai-testing-journey.git
git branch -M main
git push -u origin main
```

**完成标志：**
- [ ] Git 仓库初始化
- [ ] .gitignore 配置正确
- [ ] GitHub 仓库创建成功
- [ ] 代码成功推送

---

#### 15:00-16:00 pytest 快速入门（1 小时）

**学习内容：**

```python
# 创建测试文件 tests/test_basics.py

# 示例 1：基础测试
def test_addition():
    assert 1 + 1 == 2

def test_string_contains():
    text = "AI Testing"
    assert "Testing" in text

# 示例 2：测试异常
def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0

# 示例 3：参数化测试
import pytest

@pytest.mark.parametrize("input,expected", [
    (2, 4),
    (3, 9),
    (4, 16),
])
def test_square(input, expected):
    assert input ** 2 == expected
```

**练习题：**
```python
# 创建你自己的测试文件 tests/test_exercise.py
# 完成以下测试：
# 1. 测试字符串反转函数
# 2. 测试列表去重函数
# 3. 测试字典合并函数

def reverse_string(s):
    """反转字符串"""
    pass

def remove_duplicates(lst):
    """列表去重"""
    pass

def merge_dicts(dict1, dict2):
    """合并两个字典"""
    pass

# 为以上三个函数编写测试用例
```

**完成标志：**
- [ ] 运行 `pytest` 所有测试通过
- [ ] 完成 3 个练习题的测试编写

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 生成首日报告（30 分钟）

**任务：** 创建学习笔记文档

```markdown
# 学习日志 - 第 1 天

## 今日学习内容
1. AI 测试概述
2. 评估指标（Accuracy、Precision、Recall、F1）
3. Python 环境搭建
4. Git 配置

## 关键收获
（写下你最重要的 3 个收获）

## 遇到的问题
（记录遇到的问题和解决方案）

## 明日计划
（写下明天的学习目标）
```

**完成标志：**
- [ ] 学习笔记提交到 GitHub
- [ ] 今日所有任务打勾完成

---

### ✅ 第 1 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| AI 测试概述学习 | 完成 500 字笔记 | ☐ |
| 评估指标学习 | 完成对照表 | ☐ |
| Python 环境搭建 | pytest --version 无报错 | ☐ |
| Git 配置 | GitHub 仓库可用 | ☐ |
| pytest 入门 | 3 个练习题完成 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 2 天（周二）- Python 测试基础进阶

**学习目标：** 掌握 pytest 核心功能，能编写完整的测试用例

### 📚 上午：pytest 深入（2.5 小时）

#### 09:00-10:00 pytest fixture（1 小时）

**学习内容：**

```python
# tests/test_fixtures.py
import pytest

# Fixture 基础
@pytest.fixture
def sample_data():
    """提供测试数据"""
    return {"name": "测试", "value": 100}

def test_with_fixture(sample_data):
    assert sample_data["value"] == 100

# Fixture 的 setup/teardown
@pytest.fixture
def database_connection():
    """模拟数据库连接"""
    print("\n设置数据库连接...")
    conn = {"connected": True}
    yield conn
    print("\n关闭数据库连接...")
    conn["connected"] = False

def test_database(database_connection):
    assert database_connection["connected"] == True

# Fixture 作用域
@pytest.fixture(scope="module")
def shared_resource():
    """模块级共享资源"""
    print("\n初始化共享资源（只执行一次）")
    return {"data": []}

def test_1(shared_resource):
    shared_resource["data"].append(1)
    assert len(shared_resource["data"]) == 1

def test_2(shared_resource):
    shared_resource["data"].append(2)
    # 因为 scope="module"，data 会保留 test_1 的修改
    assert len(shared_resource["data"]) == 2
```

**练习：**
```python
# 为你的 ML 项目创建 fixture
# 1. 创建样本数据集 fixture
# 2. 创建临时目录 fixture
# 3. 创建数据库连接 mock fixture
```

**完成标志：**
- [ ] 理解 fixture 的 5 种作用域（function/class/module/package/session）
- [ ] 完成 3 个练习题

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 pytest 高级特性（1 小时）

**学习内容：**

```python
# 1. 测试跳过和预期失败
import pytest
import sys

@pytest.mark.skip(reason="尚未实现")
def test_future_feature():
    pass

@pytest.mark.skipif(sys.version_info < (3, 10), reason="需要 Python 3.10+")
def test_python310_feature():
    pass

@pytest.mark.xfail(reason="已知问题")
def test_known_issue():
    assert False

# 2. 测试标记和选择运行
@pytest.mark.slow
def test_slow_operation():
    import time
    time.sleep(2)

@pytest.mark.smoke
def test_critical_path():
    assert True

# 运行特定标记的测试：
# pytest -m smoke
# pytest -m "not slow"

# 3. Mock 和 patch
from unittest.mock import Mock, patch

def test_with_mock():
    mock_api = Mock()
    mock_api.get.return_value = {"status": "ok"}
    result = mock_api.get("/users")
    assert result["status"] == "ok"
    mock_api.get.assert_called_once_with("/users")

@patch('requests.get')
def test_with_patch(mock_get):
    mock_get.return_value.json.return_value = {"data": []}
    # 测试代码...
    mock_get.assert_called()
```

**练习：**
```python
# 创建综合练习题
# 1. 为 API 调用编写 mock 测试
# 2. 为耗时操作添加 @pytest.mark.slow 标记
# 3. 为一个已知 bug 添加 xfail 标记
```

**完成标志：**
- [ ] 掌握 skip、xfail、mark 的用法
- [ ] 能用 Mock 测试外部依赖

---

### 💻 下午：综合练习（2.5 小时）

#### 14:00-15:30 数据模块测试（1.5 小时）

**任务：** 为数据加载模块编写完整测试

```python
# src/data_loader.py
import pandas as pd
from typing import Dict, Any

class DataLoader:
    """数据加载器"""
    
    def __init__(self, file_path: str):
        self.file_path = file_path
    
    def load_csv(self) -> pd.DataFrame:
        """加载 CSV 文件"""
        return pd.read_csv(self.file_path)
    
    def validate_data(self, df: pd.DataFrame) -> Dict[str, Any]:
        """验证数据质量"""
        return {
            "row_count": len(df),
            "column_count": len(df.columns),
            "missing_values": df.isnull().sum().to_dict(),
            "duplicate_rows": df.duplicated().sum()
        }
```

```python
# tests/test_data_loader.py
import pytest
import pandas as pd
import os
from src.data_loader import DataLoader

@pytest.fixture
def sample_csv(tmp_path):
    """创建临时 CSV 文件"""
    csv_content = """name,age,city
Alice,25,Beijing
Bob,30,Shanghai
Charlie,,Guangzhou
"""
    csv_file = tmp_path / "sample.csv"
    csv_file.write_text(csv_content)
    return str(csv_file)

def test_load_csv(sample_csv):
    loader = DataLoader(sample_csv)
    df = loader.load_csv()
    assert len(df) == 3
    assert list(df.columns) == ["name", "age", "city"]

def test_validate_data(sample_csv):
    loader = DataLoader(sample_csv)
    df = loader.load_csv()
    validation = loader.validate_data(df)
    
    assert validation["row_count"] == 3
    assert validation["column_count"] == 3
    assert "age" in validation["missing_values"]
    assert validation["duplicate_rows"] == 0
```

**完成标志：**
- [ ] 数据加载测试覆盖率达到 80%+
- [ ] 所有边界情况有测试覆盖

---

#### 15:30-16:00 ☕ 休息（30 分钟）

---

#### 16:00-17:00 测试覆盖率报告（1 小时）

**任务：** 生成并分析测试覆盖率报告

```bash
# 1. 运行带覆盖率的测试
pytest --cov=src --cov-report=html --cov-report=term

# 2. 查看终端报告
# 在终端输出中查看覆盖率

# 3. 打开 HTML 报告
open htmlcov/index.html  # macOS
# 或
xdg-open htmlcov/index.html  # Linux
# 或
start htmlcov\\index.html  # Windows
```

**目标：**
- [ ] 整体覆盖率 >= 70%
- [ ] 核心模块覆盖率 >= 80%
- [ ] 无完全未覆盖的文件

**分析任务：**
```markdown
# 覆盖率分析报告

## 覆盖率总览
- 行覆盖率：__%
- 分支覆盖率：__%
- 函数覆盖率：__%

## 未覆盖的关键代码
（列出需要补充测试的代码）

## 改进计划
（写下如何提高覆盖率）
```

**完成标志：**
- [ ] HTML 报告生成成功
- [ ] 覆盖率达到目标
- [ ] 提交覆盖率报告到 GitHub

---

### ✅ 第 2 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| pytest fixture | 3 个练习题完成 | ☐ |
| pytest 高级特性 | mock 测试完成 | ☐ |
| 数据模块测试 | 覆盖率 80%+ | ☐ |
| 覆盖率报告 | HTML 报告生成 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 3 天（周三）- 数据加载与清洗测试

**学习目标：** 掌握数据质量测试方法，实现数据验证

### 📚 上午：数据质量理论（2.5 小时）

#### 09:00-10:00 数据质量问题类型（1 小时）

**学习内容：**

```markdown
# 常见数据质量问题

## 1. 缺失值（Missing Values）
- 完全随机缺失（MCAR）
- 随机缺失（MAR）
- 非随机缺失（MNAR）

## 2. 异常值（Outliers）
- 统计异常（3σ原则）
- 业务异常（超出合理范围）

## 3. 不一致性（Inconsistency）
- 格式不一致（日期、电话）
- 命名不一致（同义词）

## 4. 重复数据（Duplicates）
- 完全重复
- 部分重复

## 5. 数据漂移（Data Drift）
- 分布变化
- 概念漂移
```

**阅读材料：**
```
必读：
1. "Data Quality for Machine Learning" 
   - https://www.mountaingoatsoftware.com/blog/data-quality-for-ml

2. Google 的 "Data Quality Essentials"
   - https://cloud.google.com/dataflow/docs/concepts/data-quality
```

**完成标志：**
- [ ] 完成数据质量问题分类笔记
- [ ] 能识别 5 种数据质量问题

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 数据验证框架（1 小时）

**学习内容：**

```python
# src/data_validation.py
from typing import List, Dict, Any
import pandas as pd

class DataValidator:
    """数据验证器"""
    
    def __init__(self, df: pd.DataFrame):
        self.df = df
        self.issues = []
    
    def check_missing(self, columns: List[str], threshold: float = 0.5):
        """检查缺失值"""
        for col in columns:
            if col not in self.df.columns:
                self.issues.append({
                    "type": "missing_column",
                    "column": col,
                    "severity": "error"
                })
                continue
            
            missing_ratio = self.df[col].isnull().sum() / len(self.df)
            if missing_ratio > threshold:
                self.issues.append({
                    "type": "high_missing_ratio",
                    "column": col,
                    "ratio": missing_ratio,
                    "severity": "warning"
                })
        return self
    
    def check_duplicates(self):
        """检查重复数据"""
        dup_count = self.df.duplicated().sum()
        if dup_count > 0:
            self.issues.append({
                "type": "duplicates",
                "count": dup_count,
                "severity": "warning"
            })
        return self
    
    def check_value_range(self, column: str, min_val: Any, max_val: Any):
        """检查值范围"""
        if column not in self.df.columns:
            return self
        
        invalid = self.df[
            (self.df[column] < min_val) | 
            (self.df[column] > max_val)
        ]
        
        if len(invalid) > 0:
            self.issues.append({
                "type": "out_of_range",
                "column": column,
                "count": len(invalid),
                "severity": "warning"
            })
        return self
    
    def get_report(self) -> Dict[str, Any]:
        """生成验证报告"""
        error_count = sum(1 for i in self.issues if i["severity"] == "error")
        warning_count = sum(1 for i in self.issues if i["severity"] == "warning")
        
        return {
            "total_issues": len(self.issues),
            "errors": error_count,
            "warnings": warning_count,
            "issues": self.issues,
            "passed": error_count == 0
        }
```

**完成标志：**
- [ ] 理解数据验证核心方法
- [ ] 能解释每种验证的用途

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 实现数据验证测试（2 小时）

**任务：** 为 DataValidator 编写完整测试

```python
# tests/test_data_validation.py
import pytest
import pandas as pd
from src.data_validation import DataValidator

@pytest.fixture
def clean_df():
    """干净的数据集"""
    return pd.DataFrame({
        "name": ["Alice", "Bob", "Charlie"],
        "age": [25, 30, 35],
        "city": ["Beijing", "Shanghai", "Guangzhou"]
    })

@pytest.fixture
def dirty_df():
    """有问题的数据集"""
    return pd.DataFrame({
        "name": ["Alice", "Bob", "Bob", None],
        "age": [25, -5, 30, 200],  # 负年龄和异常值
        "city": ["Beijing", "Shanghai", "Shanghai", "Beijing"]
    })

def test_check_missing_with_valid_data(clean_df):
    validator = DataValidator(clean_df)
    validator.check_missing(["name", "age"])
    report = validator.get_report()
    
    assert report["errors"] == 0
    assert report["passed"] == True

def test_check_missing_with_invalid_column(clean_df):
    validator = DataValidator(clean_df)
    validator.check_missing(["nonexistent_column"])
    report = validator.get_report()
    
    assert report["errors"] == 1
    assert report["issues"][0]["type"] == "missing_column"

def test_check_duplicates(dirty_df):
    validator = DataValidator(dirty_df)
    validator.check_duplicates()
    report = validator.get_report()
    
    assert report["warnings"] >= 1
    assert any(i["type"] == "duplicates" for i in report["issues"])

def test_check_value_range(dirty_df):
    validator = DataValidator(dirty_df)
    validator.check_value_range("age", 0, 120)
    report = validator.get_report()
    
    assert report["warnings"] >= 1
    assert any(i["type"] == "out_of_range" for i in report["issues"])

def test_full_validation_pipeline(dirty_df):
    """完整验证流程测试"""
    validator = DataValidator(dirty_df)
    validator.check_missing(["name"]).check_duplicates().check_value_range("age", 0, 120)
    report = validator.get_report()
    
    assert report["total_issues"] > 0
    # 注意：有错误时 passed 应为 False
    assert report["passed"] == (report["errors"] == 0)
```

**完成标志：**
- [ ] 所有测试通过
- [ ] 测试覆盖所有验证方法
- [ ] 边界情况有测试覆盖

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 学习日志（30 分钟）

**任务：** 更新学习日志，记录今日收获

```markdown
# 学习日志 - 第 3 天

## 今日学习内容
1. 数据质量问题类型（5 种）
2. 数据验证框架实现
3. 数据验证测试

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

### ✅ 第 3 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| 数据质量理论 | 完成分类笔记 | ☐ |
| 数据验证框架 | DataValidator 实现 | ☐ |
| 数据验证测试 | 所有测试通过 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 4 天（周四）- 特征工程与测试

**学习目标：** 掌握特征工程测试方法，实现特征处理模块

### 📚 上午：特征工程理论（2.5 小时）

#### 09:00-10:00 特征工程基础（1 小时）

**学习内容：**

```markdown
# 特征工程核心方法

## 1. 数值特征处理
### 标准化（Standardization）
- 公式：z = (x - μ) / σ
- 适用：符合正态分布的数据
- 工具：StandardScaler

### 归一化（Normalization）
- 公式：x' = (x - min) / (max - min)
- 适用：有明确边界的数据
- 工具：MinMaxScaler

## 2. 类别特征处理
### One-Hot 编码
- 适用：无序类别
- 注意：维度灾难

### Label 编码
- 适用：有序类别
- 注意：虚假顺序关系

## 3. 缺失值处理
- 删除（数据充足时）
- 填充（均值/中位数/众数）
- 预测填充（复杂场景）
```

**实践：**
```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, OneHotEncoder
import pandas as pd

# 示例代码
df = pd.DataFrame({
    "age": [25, 30, 35, 40],
    "income": [50000, 60000, 70000, 80000],
    "city": ["Beijing", "Shanghai", "Guangzhou", "Shenzhen"]
})

# 标准化
scaler = StandardScaler()
df[["age_std", "income_std"]] = scaler.fit_transform(df[["age", "income"]])

# One-Hot 编码
encoder = OneHotEncoder(sparse=False)
city_encoded = encoder.fit_transform(df[["city"]])
```

**完成标志：**
- [ ] 理解标准化 vs 归一化的区别
- [ ] 能解释 One-Hot 编码的优缺点

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 特征工程测试策略（1 小时）

**学习内容：**

```python
# 特征工程测试要点

## 1. 变换一致性测试
- 同一样本多次变换结果一致
- 训练/测试集使用相同的变换参数

## 2. 边界值测试
- 极小值、极大值的处理
- 缺失值的处理

## 3. 数据泄露防护
- 测试集的统计量不能用于训练集变换
- 变换参数只能从训练集学习

## 4. 可逆变换测试（如适用）
- 逆变换后能恢复原始数据
```

**示例测试：**

```python
# tests/test_feature_engineering.py
import pytest
import numpy as np
from sklearn.preprocessing import StandardScaler

def test_scaler_consistency():
    """测试变换一致性"""
    scaler = StandardScaler()
    X_train = np.array([[1], [2], [3], [4], [5]])
    
    # 拟合并变换
    scaler.fit(X_train)
    X_train_transformed = scaler.transform(X_train)
    
    # 再次变换应该得到相同结果
    X_train_transformed_2 = scaler.transform(X_train)
    np.testing.assert_array_equal(X_train_transformed, X_train_transformed_2)

def test_no_data_leakage():
    """测试无数据泄露"""
    scaler = StandardScaler()
    
    X_train = np.array([[1], [2], [3]])
    X_test = np.array([[10], [20]])  # 测试集有更大值
    
    # 正确做法：只用训练集拟合
    scaler.fit(X_train)
    X_train_scaled = scaler.transform(X_train)
    X_test_scaled = scaler.transform(X_test)
    
    # 测试集缩放后可能超出 [-1, 1] 范围（这是正常的）
    # 错误做法是用整个数据集拟合
    assert np.mean(X_train_scaled) == pytest.approx(0)
```

**完成标志：**
- [ ] 理解特征工程测试的核心要点
- [ ] 能识别数据泄露风险

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-16:00 实现特征处理模块（2 小时）

**任务：** 创建可测试的特征处理模块

```python
# src/feature_engineering.py
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.base import BaseEstimator, TransformerMixin
import pandas as pd
import numpy as np

class FeatureProcessor(BaseEstimator, TransformerMixin):
    """特征处理器"""
    
    def __init__(self, numeric_columns: list, categorical_columns: list):
        self.numeric_columns = numeric_columns
        self.categorical_columns = categorical_columns
        self.numeric_scaler = StandardScaler()
        self.categorical_encoder = OneHotEncoder(handle_unknown="ignore", sparse=False)
        self._is_fitted = False
    
    def fit(self, X: pd.DataFrame, y=None):
        """拟合 Transform"""
        # 处理数值特征
        if self.numeric_columns:
            self.numeric_scaler.fit(X[self.numeric_columns])
        
        # 处理类别特征
        if self.categorical_columns:
            self.categorical_encoder.fit(X[self.categorical_columns])
        
        self._is_fitted = True
        return self
    
    def transform(self, X: pd.DataFrame) -> np.ndarray:
        """变换数据"""
        if not self._is_fitted:
            raise RuntimeError("处理器尚未拟合，请先调用 fit()")
        
        features = []
        
        # 变换数值特征
        if self.numeric_columns:
            numeric_transformed = self.numeric_scaler.transform(X[self.numeric_columns])
            features.append(numeric_transformed)
        
        # 变换类别特征
        if self.categorical_columns:
            categorical_transformed = self.categorical_encoder.transform(X[self.categorical_columns])
            features.append(categorical_transformed)
        
        return np.hstack(features) if features else None
    
    def fit_transform(self, X: pd.DataFrame, y=None) -> np.ndarray:
        """拟合并变换"""
        return self.fit(X, y).transform(X)
```

**完成标志：**
- [ ] FeatureProcessor 实现完成
- [ ] 代码符合 sklearn Transformer API 规范

---

#### 16:00-16:30 ☕ 休息（30 分钟）

---

#### 16:30-17:00 编写特征处理测试（30 分钟）

```python
# tests/test_feature_engineering.py
import pytest
import pandas as pd
import numpy as np
from src.feature_engineering import FeatureProcessor

@pytest.fixture
def sample_df():
    """样本数据"""
    return pd.DataFrame({
        "age": [25, 30, 35, 40, 45],
        "income": [50000, 60000, 70000, 80000, 90000],
        "city": ["Beijing", "Shanghai", "Beijing", "Guangzhou", "Shenzhen"],
        "target": [0, 1, 0, 1, 1]
    })

def test_fit_transform(sample_df):
    """测试拟合和变换"""
    processor = FeatureProcessor(
        numeric_columns=["age", "income"],
        categorical_columns=["city"]
    )
    
    result = processor.fit_transform(sample_df)
    
    # 检查结果形状
    assert result is not None
    assert result.shape[0] == len(sample_df)
    # 数值特征 2 列 + 类别特征（4 座城市）4 列 = 6 列
    assert result.shape[1] == 6

def test_transform_without_fit(sample_df):
    """测试未拟合时变换应报错"""
    processor = FeatureProcessor(
        numeric_columns=["age", "income"],
        categorical_columns=["city"]
    )
    
    with pytest.raises(RuntimeError):
        processor.transform(sample_df)

def test_consistency(sample_df):
    """测试变换一致性"""
    processor = FeatureProcessor(
        numeric_columns=["age", "income"],
        categorical_columns=["city"]
    )
    
    # 两次变换结果应相同
    result1 = processor.fit_transform(sample_df)
    result2 = processor.transform(sample_df)
    
    np.testing.assert_array_equal(result1, result2)

def test_standardization(sample_df):
    """验证标准化效果"""
    processor = FeatureProcessor(
        numeric_columns=["age"],
        categorical_columns=[]
    )
    
    result = processor.fit_transform(sample_df)
    
    # 标准化后均值应接近 0
    assert np.mean(result) == pytest.approx(0, abs=0.001)
```

**完成标志：**
- [ ] 所有测试通过
- [ ] 测试覆盖边界情况

---

### ✅ 第 4 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| 特征工程理论 | 完成学习笔记 | ☐ |
| FeatureProcessor | 实现完成 | ☐ |
| 特征处理测试 | 所有测试通过 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 5 天（周五）- 模型训练与评估

**学习目标：** 实现 ML 模型训练模块，掌握评估指标计算

### 📚 上午：ML 模型基础（2.5 小时）

#### 09:00-10:00 Scikit-learn 入门（1 小时）

**学习内容：**

```python
# Scikit-learn 统一 API

from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# 1. 准备数据
X = ...  # 特征
y = ...  # 标签

# 2. 划分训练/测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 3. 创建模型
model = LogisticRegression()

# 4. 训练
model.fit(X_train, y_train)

# 5. 预测
y_pred = model.predict(X_test)

# 6. 评估
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.4f}")
print(classification_report(y_test, y_pred))
```

**练习：**
```python
# 用不同模型训练并比较
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier

models = {
    "Logistic Regression": LogisticRegression(),
    "Decision Tree": DecisionTreeClassifier(),
    "Random Forest": RandomForestClassifier()
}

for name, model in models.items():
    model.fit(X_train, y_train)
    score = model.score(X_test, y_test)
    print(f"{name}: {score:.4f}")
```

**完成标志：**
- [ ] 理解 sklearn 统一 API
- [ ] 能用 3 种模型训练

---

#### 10:00-10:15 ☕ 休息（15 分钟）

---

#### 10:15-11:15 评估指标深入（1 小时）

**学习内容：**

```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, confusion_matrix, classification_report
)

# 多指标评估
y_true = [0, 1, 1, 0, 1, 0, 1, 0]
y_pred = [0, 1, 0, 0, 1, 1, 1, 0]

print(f"Accuracy: {accuracy_score(y_true, y_pred):.4f}")
print(f"Precision: {precision_score(y_true, y_pred):.4f}")
print(f"Recall: {recall_score(y_true, y_pred):.4f}")
print(f"F1 Score: {f1_score(y_true, y_pred):.4f}")

# 需要预测概率的指标
from sklearn.linear_model import LogisticRegression
model = LogisticRegression()
model.fit(X_train, y_train)
y_proba = model.predict_proba(X_test)[:, 1]
print(f"ROC AUC: {roc_auc_score(y_test, y_proba):.4f}")

# 混淆矩阵
import seaborn as sns
import matplotlib.pyplot as plt

cm = confusion_matrix(y_true, y_pred)
sns.heatmap(cm, annot=True, fmt="d")
plt.xlabel("Predicted")
plt.ylabel("Actual")
plt.show()
```

**完成标志：**
- [ ] 理解每个指标的含义和适用场景
- [ ] 能计算并解释所有指标

---

### 💻 下午：实战练习（2.5 小时）

#### 14:00-15:30 实现模型训练模块（1.5 小时）

**任务：** 创建可测试的模型训练模块

```python
# src/model_training.py
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import accuracy_score, classification_report
import pandas as pd
import numpy as np

class ModelTrainer:
    """模型训练器"""
    
    def __init__(self, model=None):
        self.model = model if model else LogisticRegression()
        self.training_history = []
    
    def train(self, X: np.ndarray, y: np.ndarray, 
              test_size: float = 0.2, random_state: int = 42):
        """训练模型"""
        # 划分数据
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=test_size, random_state=random_state
        )
        
        # 训练
        self.model.fit(X_train, y_train)
        
        # 评估
        y_pred = self.model.predict(X_test)
        y_proba = self.model.predict_proba(X_test)[:, 1] if hasattr(self.model, "predict_proba") else None
        
        metrics = {
            "accuracy": accuracy_score(y_test, y_pred),
            "y_pred": y_pred,
            "y_proba": y_proba,
            "y_test": y_test
        }
        
        self.training_history.append(metrics)
        
        return metrics
    
    def cross_validate(self, X: np.ndarray, y: np.ndarray, cv: int = 5):
        """交叉验证"""
        scores = cross_val_score(self.model, X, y, cv=cv, scoring="accuracy")
        return {
            "mean_accuracy": scores.mean(),
            "std_accuracy": scores.std(),
            "scores": scores
        }
    
    def get_report(self, y_true: np.ndarray, y_pred: np.ndarray) -> str:
        """生成分类报告"""
        return classification_report(y_true, y_pred)
```

**完成标志：**
- [ ] ModelTrainer 实现完成
- [ ] 支持交叉验证

---

#### 15:30-16:00 ☕ 休息（30 分钟）

---

#### 16:00-17:00 编写模型训练测试（1 小时）

```python
# tests/test_model_training.py
import pytest
import numpy as np
from sklearn.datasets import make_classification
from src.model_training import ModelTrainer

@pytest.fixture
def binary_classification_data():
    """二分类数据"""
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    return X, y

def test_train(binary_classification_data):
    """测试训练功能"""
    X, y = binary_classification_data
    trainer = ModelTrainer()
    
    metrics = trainer.train(X, y)
    
    assert "accuracy" in metrics
    assert 0 <= metrics["accuracy"] <= 1
    assert len(metrics["y_pred"]) == int(len(X) * 0.2)

def test_cross_validate(binary_classification_data):
    """测试交叉验证"""
    X, y = binary_classification_data
    trainer = ModelTrainer()
    
    cv_results = trainer.cross_validate(X, y, cv=5)
    
    assert "mean_accuracy" in cv_results
    assert "std_accuracy" in cv_results
    assert len(cv_results["scores"]) == 5
    assert all(0 <= s <= 1 for s in cv_results["scores"])

def test_training_history(binary_classification_data):
    """测试训练历史记录"""
    X, y = binary_classification_data
    trainer = ModelTrainer()
    
    # 训练多次
    trainer.train(X, y)
    trainer.train(X, y)
    trainer.train(X, y)
    
    assert len(trainer.training_history) == 3

def test_classification_report(binary_classification_data):
    """测试分类报告生成"""
    X, y = binary_classification_data
    trainer = ModelTrainer()
    
    metrics = trainer.train(X, y)
    report = trainer.get_report(metrics["y_test"], metrics["y_pred"])
    
    assert "precision" in report
    assert "recall" in report
    assert "f1-score" in report
```

**完成标志：**
- [ ] 所有测试通过
- [ ] 测试覆盖训练、交叉验证、报告生成

---

### ✅ 第 5 天验收清单

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| Scikit-learn 入门 | 能用 3 种模型训练 | ☐ |
| 评估指标 | 理解并能计算所有指标 | ☐ |
| ModelTrainer | 实现完成 | ☐ |
| 模型训练测试 | 所有测试通过 | ☐ |
| 学习日志 | 提交到 GitHub | ☐ |

---

## 第 6 天（周六）- 周项目整合

**学习目标：** 整合本周所有模块，形成完整可运行的 ML 流水线

### 💻 全天：项目整合（5 小时）

#### 09:00-11:00 整合 ML 流水线（2 小时）

**任务：** 创建完整的 ML 流水线

```python
# src/pipeline.py
from .data_loader import DataLoader
from .data_validation import DataValidator
from .feature_engineering import FeatureProcessor
from .model_training import ModelTrainer
import pandas as pd

class MLPipeline:
    """完整的 ML 流水线"""
    
    def __init__(self, config: dict):
        self.config = config
        self.data_loader = None
        self.validator = None
        self.processor = None
        self.trainer = None
        self.results = {}
    
    def run(self, data_path: str) -> dict:
        """运行完整流水线"""
        # 1. 数据加载
        print("步骤 1: 加载数据...")
        self.data_loader = DataLoader(data_path)
        df = self.data_loader.load_csv()
        
        # 2. 数据验证
        print("步骤 2: 验证数据...")
        self.validator = DataValidator(df)
        validation_report = self.validator.get_report()
        self.results["validation"] = validation_report
        
        if validation_report["errors"] > 0:
            raise ValueError(f"数据验证失败：{validation_report['errors']} 个错误")
        
        # 3. 特征处理
        print("步骤 3: 特征处理...")
        self.processor = FeatureProcessor(
            numeric_columns=self.config["numeric_columns"],
            categorical_columns=self.config["categorical_columns"]
        )
        X = self.processor.fit_transform(df)
        y = df[self.config["target_column"]].values
        
        # 4. 模型训练
        print("步骤 4: 训练模型...")
        self.trainer = ModelTrainer()
        train_metrics = self.trainer.train(X, y)
        self.results["training"] = train_metrics
        
        # 5. 交叉验证
        print("步骤 5: 交叉验证...")
        cv_metrics = self.trainer.cross_validate(X, y)
        self.results["cross_validation"] = cv_metrics
        
        # 6. 生成报告
        print("步骤 6: 生成报告...")
        report = self.generate_report()
        self.results["report"] = report
        
        return self.results
    
    def generate_report(self) -> str:
        """生成完整报告"""
        report = []
        report.append("=" * 50)
        report.append("ML Pipeline 执行报告")
        report.append("=" * 50)
        
        # 数据验证报告
        report.append("\n【数据验证】")
        validation = self.results.get("validation", {})
        report.append(f"总问题数：{validation.get('total_issues', 'N/A')}")
        report.append(f"错误：{validation.get('errors', 'N/A')}")
        report.append(f"警告：{validation.get('warnings', 'N/A')}")
        
        # 训练报告
        report.append("\n【模型训练】")
        training = self.results.get("training", {})
        report.append(f"准确率：{training.get('accuracy', 'N/A'):.4f}")
        
        # 交叉验证报告
        report.append("\n【交叉验证】")
        cv = self.results.get("cross_validation", {})
        report.append(f"平均准确率：{cv.get('mean_accuracy', 'N/A'):.4f} (+/- {cv.get('std_accuracy', 'N/A'):.4f})")
        
        report.append("\n" + "=" * 50)
        
        return "\n".join(report)
```

**完成标志：**
- [ ] 流水线可完整运行
- [ ] 各模块正常协作

---

#### 11:00-11:15 ☕ 休息（15 分钟）

---

#### 11:15-12:00 测试覆盖率检查（45 分钟）

**任务：** 运行完整测试套件，生成覆盖率报告

```bash
# 运行所有测试
pytest tests/ -v --cov=src --cov-report=html --cov-report=term

# 检查覆盖率是否达标
# 目标：整体 70%+，核心模块 80%+
```

**完成标志：**
- [ ] 所有测试通过
- [ ] 整体覆盖率 >= 70%

---

#### 12:00-13:00 🍽️ 午餐休息（1 小时）

---

#### 13:00-14:30 项目文档编写（1.5 小时）

**任务：** 编写完整的 README 文档

```markdown
# AI Testing Journey - 第 1 周项目

## 项目概述
这是一个完整的 ML 流水线项目，包含数据加载、验证、特征处理和模型训练。

## 项目结构
```
ai-testing-journey/
├── src/
│   ├── data_loader.py
│   ├── data_validation.py
│   ├── feature_engineering.py
│   ├── model_training.py
│   └── pipeline.py
├── tests/
│   ├── test_data_loader.py
│   ├── test_data_validation.py
│   ├── test_feature_engineering.py
│   └── test_model_training.py
├── data/
│   └── sample_data.csv
└── README.md
```

## 快速开始

### 安装依赖
```bash
pip install -r requirements.txt
```

### 运行流水线
```bash
python main.py --data data/sample_data.csv
```

### 运行测试
```bash
pytest tests/ -v --cov=src
```

## 测试结果
- 测试覆盖率：XX%
- 模型准确率：XX%
- 交叉验证得分：XX (+/- XX)
```

**完成标志：**
- [ ] README 文档完整
- [ ] 包含安装、运行、测试说明

---

#### 14:30-15:00 ☕ 休息（30 分钟）

---

#### 15:00-16:00 最终验证（1 小时）

**任务：** 完整运行和验证项目

```bash
# 1. 从干净环境运行
rm -rf venv/
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 2. 运行完整流水线
python main.py --data data/sample_data.csv

# 3. 运行所有测试
pytest tests/ -v

# 4. 生成覆盖率报告
pytest --cov=src --cov-report=html

# 5. 提交到 Git
git add .
git commit -m "Week 1 complete: Full ML pipeline with tests"
git push
```

**完成标志：**
- [ ] 流水线运行成功
- [ ] 所有测试通过
- [ ] 代码提交到 GitHub

---

#### 16:00-17:00 周总结与反思（1 小时）

**任务：** 完成周总结文档

```markdown
# 第 1 周学习总结

## 学习内容回顾
1. AI 测试基础概念
2. Python 测试框架（pytest）
3. 数据加载与验证
4. 特征工程
5. 模型训练与评估
6. 完整 ML 流水线

## 完成的产出
- [ ] GitHub 仓库：https://github.com/...
- [ ] 代码行数：XXX 行
- [ ] 测试覆盖率：XX%
- [ ] 测试用例数：XX 个

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

## 第 2 周计划
- 重点学习：数据管线测试
- 目标完成：___________
- 预计投入：25-30 小时
```

**完成标志：**
- [ ] 周总结完成
- [ ] 第 2 周计划制定

---

### ✅ 第 6 天验收清单（本周最终验收）

| 任务 | 完成标志 | 状态 |
|------|---------|------|
| ML 流水线整合 | 可完整运行 | ☐ |
| 测试覆盖率 | 整体 70%+ | ☐ |
| 项目文档 | README 完整 | ☐ |
| 最终验证 | 所有测试通过 | ☐ |
| 周总结 | 文档提交 | ☐ |
| Git 提交 | 代码推送 | ☐ |

---

## 📊 第 1 周完成度自检表

### 知识掌握

| 知识点 | 理解程度 | 自评 |
|--------|---------|------|
| AI 测试核心概念 | 能向他人解释 | ☐☐☐☐☐ |
| pytest 基础 | 能独立编写测试 | ☐☐☐☐☐ |
| pytest 高级特性 | 能使用 fixture、mock | ☐☐☐☐☐ |
| 数据质量验证 | 能实现验证逻辑 | ☐☐☐☐☐ |
| 特征工程 | 掌握标准化、编码 | ☐☐☐☐☐ |
| 模型训练评估 | 能训练并评估模型 | ☐☐☐☐☐ |

### 技能达成

| 技能 | 目标 | 实际 | 状态 |
|------|------|------|------|
| Python 环境 | 完成搭建 | ☐ | ☐ |
| Git 配置 | 仓库可用 | ☐ | ☐ |
| pytest 使用 | 独立编写测试 | ☐ | ☐ |
| ML 流水线 | 完整实现 | ☐ | ☐ |
| 测试覆盖率 | 70%+ | ☐% | ☐ |

### 产出检查

| 产出物 | 要求 | 完成 |
|--------|------|------|
| GitHub 仓库 | 可访问 | ☐ |
| ML 流水线代码 | 可运行 | ☐ |
| 测试套件 | 全部通过 | ☐ |
| 覆盖率报告 | 70%+ | ☐ |
| 学习日志 | 6 篇完成 | ☐ |
| 周总结 | 提交 | ☐ |

---

## 🎯 第 2 周预习

### 预习内容
- 阅读 Great Expectations 文档
- 了解 DVC 数据版本控制
- 复习确定性测试概念

### 预习时间
- 预计：2-3 小时（周日）

### 预习目标
- 理解数据管线测试核心概念
- 熟悉 DVC 基本命令

---

**第 1 周学习计划完成！** 🎉

准备好迎接第 2 周的挑战了吗？继续加油！💪
