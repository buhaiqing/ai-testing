# Python 自动化测试开发规范与最佳实践指南

**版本**：v1.0  
**最后更新**：2026-04-22  
**适用项目**：AI Agent 认证与自动化测试（ai-testing）  
**目标人群**：开发/AI 背景转型测试工程师

---

## 📋 目录

1. [项目结构与入口文件规范](#1-项目结构与入口文件规范)
2. [编码最佳实践](#2-编码最佳实践)
3. [常见错误与反模式](#3-常见错误与反模式)
4. [AI/ML 测试专项规范](#4-aiml-测试专项规范)
5. [CI/CD 集成规范](#5-cicd-集成规范)
6. [附录](#6-附录)

---

## 1️⃣ 项目结构与入口文件规范

### 1.1 标准项目结构

```
project-root/
├── src/                          # 源代码
│   ├── app/                      # 主应用代码
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI/Flask 入口
│   │   └── config.py            # 配置管理
│   │
│   ├── api/                      # API 路由
│   │   ├── __init__.py
│   │   ├── endpoints/
│   │   │   ├── __init__.py
│   │   │   └── users.py
│   │   │   └── products.py
│   │   └── dependencies.py
│   │
│   ├── models/                   # 数据模型 (SQLModel/Pydantic)
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── product.py
│   │
│   └── utils/                    # 工具函数
│       ├── __init__.py
│       ├── helpers.py
│       └── logger.py
│
├── tests/                        # 测试代码
│   ├── __init__.py
│   ├── conftest.py              # pytest 全局配置（fixture, hooks）
│   ├── requirements.txt         # 测试依赖
│   │
│   ├── unit/                    # 单元测试
│   │   ├── __init__.py
│   │   ├── test_models.py
│   │   └── test_utils.py
│   │
│   ├── integration/             # 集成测试
│   │   ├── __init__.py
│   │   ├── api/                 # API 集成测试
│   │   │   ├── __init__.py
│   │   │   ├── test_users.py
│   │   │   └── test_products.py
│   │   ├── database/            # 数据库集成测试
│   │   │   ├── __init__.py
│   │   │   └── test_db_operations.py
│   │   └── services/            # 第三方服务集成测试
│   │       ├── __init__.py
│   │       └── test_email.py
│   │
│   ├── e2e/                     # 端到端测试
│   │   ├── __init__.py
│   │   └── test_user_workflow.py
│   │
│   ├── performance/             # 性能测试
│   │   ├── __init__.py
│   │   └── test_api_load.py
│   │
│   └── ai/                      # AI/ML 专项测试
│       ├── __init__.py
│       ├── test_model_validation.py
│       ├── test_data_quality.py
│       └── test_agent_workflow.py
│
├── docs/                         # 文档
│   ├── api-reference.md
│   └── testing-guide.md
│
├── data/                         # 测试数据
│   ├── fixtures/                # 固定测试数据
│   │   ├── users.json
│   │   └── products.json
│   └── sample/                  # 样本数据
│       └── training_data.parquet
│
├── scripts/                      # 辅助脚本
│   ├── setup-test-db.sh
│   └── generate-fixtures.py
│
├── alembic/                      # 数据库迁移 (如果使用 SQLAlchemy)
│   ├── versions/
│   └── env.py
│
├── Dockerfile                    # Docker 配置
├── docker-compose.yml            # Docker Compose 配置
├── pytest.ini                    # pytest 配置文件
├── pyproject.toml                # 项目配置（ Poetry/Pyproject）
├── requirements.txt              # 运行时依赖
├── requirements-dev.txt          # 开发依赖
└── README.md
```

---

### 1.2 入口文件规范

#### 1.2.1 `src/app/main.py` - 应用入口

```python
"""
应用入口文件
遵循 FastAPI 最佳实践
"""

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.config import settings
from app.api.endpoints import users, products

def create_app() -> FastAPI:
    """Factory pattern 创建应用"""
    app = FastAPI(
        title=settings.APP_NAME,
        version=settings.VERSION,
        description=settings.DESCRIPTION,
    )
    
    #中间件配置
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.ALLOWED_ORIGINS,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )
    
    # 路由注册
    app.include_router(users.router, prefix="/api/v1/users", tags=["users"])
    app.include_router(products.router, prefix="/api/v1/products", tags=["products"])
    
    return app

# 全局应用实例
app = create_app()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)
```

#### 1.2.2 `tests/conftest.py` - pytest 全局配置

```python
"""
pytest 全局配置文件
定义 fixture、hook 函数、测试配置等
"""

import asyncio
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.config import settings


# ========== 数据库配置 ==========

@pytest.fixture(scope="session")
def event_loop():
    """为 pytest-asyncio 创建事件循环"""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="session")
async def async_engine():
    """异步数据库引擎（测试专用）"""
    engine = create_async_engine(
        settings.TEST_DATABASE_URL,
        echo=False,
        pool_pre_ping=True,
    )
    yield engine
    await engine.dispose()


@pytest.fixture(scope="function")
async def db_session(async_engine):
    """每个测试的数据库 session（事务隔离）"""
    TestingSessionLocal = sessionmaker(
        bind=async_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )
    
    async with TestingSessionLocal() as session:
        yield session
        await session.rollback()  # 自动回滚


# ========== 应用测试客户端 ==========

@pytest.fixture
def async_client() -> AsyncClient:
    """异步 HTTP 客户端（用于 API 测试）"""
    return AsyncClient(app=app, base_url="http://test")


# ========== 测试数据 fixtures ==========

@pytest.fixture
def sample_user_data():
    """示例用户数据"""
    return {
        "username": "testuser",
        "email": "test@example.com",
        "password": "password123"
    }


@pytest.fixture
def sample_product_data():
    """示例产品数据"""
    return {
        "name": "Test Product",
        "price": 99.99,
        "description": "Test description"
    }


# ========== Hook 函数 ==========

def pytest_configure(config):
    """pytest 配置钩子"""
    config.addinivalue_line(
        "markers", "ai: AI/ML 测试标记"
    )
    config.addinivalue_line(
        "markers", "integration: 集成测试标记"
    )
    config.addinivalue_line(
        "markers", "performance: 性能测试标记"
    )


def pytest_addoption(parser):
    """添加命令行选项"""
    parser.addoption(
        "--run-ai",
        action="store_true",
        default=False,
        help="运行 AI/ML 测试"
    )
    parser.addoption(
        "--run-performance",
        action="store_true",
        default=False,
        help="运行性能测试"
    )


# ========== pytest_RUNTEST_LOGSTART Hook ==========

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """测试报告钩子：记录失败用例的详细信息"""
    outcome = yield
    rep = outcome.get_result()
    
    if rep.failed:
        # 记录失败信息到日志
        print(f"\n❌ 测试失败: {item.nodeid}")
        if call.excinfo:
            print(f"   异常类型: {call.excinfo.type.__name__}")
            print(f"   异常信息: {call.excinfo.value}")
```

#### 1.2.3 `pytest.ini` - pytest 配置

```ini
# pytest 配置文件
[pytest]
# 测试文件模式
python_files = test_*.py *_test.py
python_classes = Test*
python_functions = test_*

# 标记
markers =
    ai: AI/ML 测试
    integration: 集成测试
    performance: 性能测试
    slow: 慢速测试

# 测试路径
testpaths = 
    tests

# 添加 import 路径
pythonpath = 
    .

# 日志配置
log_cli = True
log_cli_level = INFO
log_cli_date_format = %Y-%m-%d %H:%M:%S

# 覆盖率配置
addopts = 
    --cov=app
    --cov-report=term-missing
    --cov-report=html:htmlcov
    --cov-fail-under=80
    -v
    --tb=short

# 异步测试支持
asyncio_mode = auto
asyncio_default_fixture_loop_scope = function

# 忽略的警告
filterwarnings =
    ignore::DeprecationWarning
    ignore::PendingDeprecationWarning
```

---

## 2️⃣ 编码最佳实践

### 2.1 pytest 高级特性

#### 2.1.1 Fixture 作用域管理

```python
"""
Fixture 作用域最佳实践
"""

import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# ========== 默认作用域（function）==========
@pytest.fixture
def db_session():
    """每个测试一个 session（推荐）"""
    engine = create_engine("sqlite:///./test.db")
    Session = sessionmaker(bind=engine)
    session = Session()
    try:
        yield session
    finally:
        sessionrollback()
        session.close()


# ========== 跨测试共享（module）==========
@pytest.fixture(scope="module")
def shared_db_connection():
    """整个模块共享数据库连接（性能优化）"""
    engine = create_engine("postgresql://localhost/testdb")
    conn = engine.connect()
    yield conn
    conn.close()


# ========== 测试套件级（session）==========
@pytest.fixture(scope="session")
def test_data_loader():
    """整个测试会话加载一次大型数据集"""
    print("/loading large test dataset...")
    data = load_large_dataset()  # 可能很慢
    yield data
    print("Cleaning up test data...")


# ========== 依赖注入链 ==========

@pytest.fixture
def user_repository(db_session):
    """用户仓库"""
    return UserRepository(db_session)


@pytest.fixture
def user_service(user_repository):
    """用户服务（依赖用户仓库）"""
    return UserService(user_repository)


@pytest.fixture
def auth_client(user_service):
    """认证客户端（依赖用户服务）"""
    return AuthClient(user_service)
```

#### 2.1.2 Parametrize 参数化测试

```python
"""
参数化测试最佳实践
"""

import pytest
from app.models import User
from app.services import UserService


class TestUserCreation:
    """用户创建测试"""
    
    @pytest.mark.parametrize(
        "username, email, password, expected_errors",
        [
            # 有效用例
            ("alice", "alice@example.com", "pass123", []),
            ("bob_smith", "bob@example.com", "secure456", []),
            
            # 无效用例
            ("", "test@example.com", "pass123", ["username is required"]),
            ("alice", "invalid-email", "pass123", ["invalid email format"]),
            ("alice", "alice@example.com", "123", ["password too short"]),
            
            # 边界条件
            ("a" * 50, "valid@example.com", "pass123", []),  # 长用户名
            ("valid", "a" * 100 + "@example.com", "pass123", ["email too long"]),
        ]
    )
    def test_user_validation(
        self,
        username: str,
        email: str,
        password: str,
        expected_errors: list
    ):
        """参数化测试：用户数据验证"""
        user = User(
            username=username,
            email=email,
            password=password
        )
        
        errors = user.validate()
        
        assert errors == expected_errors, f"Expected {expected_errors}, got {errors}"
    
    @pytest.mark.parametrize(
        "role, expected_permission",
        [
            ("admin", ["read", "write", "delete"]),
            ("editor", ["read", "write"]),
            ("viewer", ["read"]),
        ],
        ids=["admin-permissions", "editor-permissions", "viewer-permissions"]
    )
    def test_role_permissions(
        self,
        role: str,
        expected_permission: list,
        user_service: UserService
    ):
        """参数化测试：角色权限验证"""
        user = user_service.create_user(
            username=f"test_{role}",
            email=f"{role}@example.com",
            password="pass123",
            role=role
        )
        
        permissions = user_service.get_permissions(user)
        
        assert set(permissions) == set(expected_permission)
```

#### 2.1.3 Markers 自定义标记

```python
"""
标记使用最佳实践
"""

import pytest

# ========== 自定义标记 ==========

@pytest.mark.ai
@pytest.mark.asyncio
async def test_llm_inference_performance(async_client: AsyncClient):
    """AI 模型推理性能测试（需要 --run-ai 运行）"""
    response = await async_client.post(
        "/api/v1/llm/inference",
        json={"prompt": "Hello, world!", "max_tokens": 100}
    )
    
    assert response.status_code == 200
    assert "response_time" in response.json()
    assert response.json()["response_time"] < 2.0


@pytest.mark.integration
def test_database_connection(db_session):
    """数据库连接集成测试"""
    result = db_session.execute("SELECT 1")
    assert result.scalar() == 1


@pytest.mark.slow
def test_large_dataset_processing():
    """大数据集处理测试（运行较慢）"""
    # 模拟大数据处理
    data = load_large_dataset(size=1_000_000)
    result = process_data(data)
    assert result is not None


# ========== conftest.py 中的标记过滤 ==========

def pytest_collection_modifyitems(config, items):
    """根据命令行选项筛选测试"""
    if not config.getoption("--run-ai"):
        # 过滤掉 AI 测试
        skip_ai = pytest.mark.skip(reason="need --run-ai to run")
        for item in items:
            if "ai" in item.keywords:
                item.add_marker(skip_ai)
    
    if not config.getoption("--run-performance"):
        # 过滤掉性能测试
        skip_perf = pytest.mark.skip(reason="need --run-performance to run")
        for item in items:
            if "performance" in item.keywords:
                item.add_marker(skip_perf)


# ========== pytest.ini 中注册标记 ==========

# [pytest]
# markers =
#     ai: AI/ML 测试
#     integration: 集成测试
#     performance: 性能测试
#     slow: 慢速测试
```

#### 2.1.4 Fixture 依赖注入

```python
"""
Fixture 依赖链最佳实践
"""

import pytest
from httpx import AsyncClient


@pytest.fixture
def database_url():
    """数据库 URL 配置"""
    return "postgresql://localhost/testdb"


@pytest.fixture
async def async_engine(database_url):
    """创建异步引擎"""
    engine = create_async_engine(database_url)
    yield engine
    await engine.dispose()


@pytest.fixture
async def test_db(async_engine):
    """创建测试表"""
    async with async_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with async_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)


@pytest.fixture
async def async_session(async_engine, test_db):
    """提供异步 session"""
    AsyncSessionLocal = sessionmaker(
        bind=async_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )
    async with AsyncSessionLocal() as session:
        yield session


@pytest.fixture
async def auth_client(async_session):
    """提供认证客户端"""
    app.dependency_overrides[get_session] = lambda: async_session
    return AsyncClient(app=app, base_url="http://test")


# ========== 使用 chain fixture ==========

@pytest.mark.asyncio
async def test_create_user_with_profile(auth_client: AsyncClient):
    """测试：创建用户及个人资料（依赖链：auth_client -> async_session -> test_db）"""
    # 测试逻辑
    response = await auth_client.post(
        "/api/v1/users",
        json={
            "username": "newuser",
            "email": "user@example.com",
            "profile": {"age": 25, "location": "NYC"}
        }
    )
    
    assert response.status_code == 201
```

---

### 2.2 异步测试处理

#### 2.2.1 async/await 测试模式

```python
"""
异步测试最佳实践
"""

import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession


@pytest.mark.asyncio
async def test_async_api_endpoint(async_client: AsyncClient):
    """异步 API 端点测试"""
    response = await async_client.get("/api/v1/users/1")
    
    assert response.status_code == 200
    user = response.json()
    assert "username" in user


@pytest.mark.asyncio
async def test_async_database_operation(db_session: AsyncSession):
    """异步数据库操作测试"""
    from app.models import User
    
    # 创建用户
    user = User(username="async_user", email="async@example.com")
    db_session.add(user)
    await db_session.commit()
    await db_session.refresh(user)
    
    # 查询用户
    result = await db_session.get(User, user.id)
    assert result.username == "async_user"
    
    # 清理
    await db_session.delete(user)
    await db_session.commit()


@pytest.mark.asyncio
async def test_async_concurrent_requests(async_client: AsyncClient):
    """并发请求测试（使用 asyncio.gather）"""
    import asyncio
    
    async def make_request(user_id: int):
        return await async_client.get(f"/api/v1/users/{user_id}")
    
    # 并发发送 10 个请求
    tasks = [make_request(i) for i in range(1, 11)]
    responses = await asyncio.gather(*tasks)
    
    # 验证所有响应
    for response in responses:
        assert response.status_code == 200
```

#### 2.2.2 异步 fixture 管理

```python
"""
异步 fixture 最佳实践
"""

import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker


@pytest.fixture(scope="session")
def event_loop():
    """创建事件循环（session 级别）"""
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()


@pytest.fixture(scope="session")
async def async_engine():
    """异步引擎（session 级别）"""
    engine = create_async_engine(
        "postgresql://localhost/testdb",
        pool_pre_ping=True,
    )
    yield engine
    await engine.dispose()


@pytest.fixture(scope="function")
async def async_session(async_engine):
    """每个测试一个 session（function 级别）"""
    TestingSessionLocal = sessionmaker(
        bind=async_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )
    
    async with TestingSessionLocal() as session:
        yield session
        await session.rollback()
```

---

### 2.3 日志记录策略

#### 2.3.1 测试日志配置

```python
"""
测试日志最佳实践
"""

import logging
import pytest


@pytest.fixture(scope="function")
def caplog_level(caplog: pytest.LogCaptureFixture):
    """自定义日志级别 fixture"""
    caplog.set_level(logging.INFO)
    return caplog


def test_application_start(caplog):
    """应用启动日志测试"""
    from app.main import create_app
    
    app = create_app()
    
    # 检查日志输出
    assert any("Application started" in record.message for record in caplog.records)


@pytest.mark.asyncio
async def test_async_operation_logs(async_client, caplog):
    """异步操作日志测试"""
    response = await async_client.post("/api/v1/data", json={"data": "test"})
    
    assert response.status_code == 200
    
    # 检查特定日志消息
    log_messages = [record.message for record in caplog.records]
    assert any("Processing data" in msg for msg in log_messages)
    assert any("Data processed successfully" in msg for msg in log_messages)
```

#### 2.3.2 结构化日志测试

```python
"""
结构化日志测试（JSON 格式）
"""

import json
import logging


def test_structured_logging_output(caplog):
    """测试结构化日志输出"""
    import structlog
    
    logger = structlog.get_logger()
    
    with caplog.at_level(logging.INFO):
        logger.info("user_login", user_id=123, ip="192.168.1.1")
        
        # 验证 JSON 格式
        log_entry = json.loads(caplog.text.strip())
        assert log_entry["event"] == "user_login"
        assert log_entry["user_id"] == 123
        assert log_entry["ip"] == "192.168.1.1"
```

---

### 2.4 断言策略

#### 2.4.1 断言层次

```python
"""
断言最佳实践：从简单到复杂
"""

import pytest


class TestUserAPI:
    """API 测试断言层次"""
    
    def test_response_status(self, async_client):
        """Level 1: 基本断言 - 状态码"""
        response = await async_client.get("/api/v1/users/1")
        
        assert response.status_code == 200  # ✓ 最基本
        assert response.status_code != 404  # ✗ 避免否定断言
    
    def test_response_structure(self, async_client):
        """Level 2: 结构断言 - JSON Schema"""
        response = await async_client.get("/api/v1/users/1")
        data = response.json()
        
        # ✓ 检查关键字段
        assert "id" in data
        assert "username" in data
        assert "email" in data
        assert "created_at" in data
        
        # ✓ 类型检查
        assert isinstance(data["id"], int)
        assert isinstance(data["username"], str)
    
    def test_business_logic(self, async_client):
        """Level 3: 业务逻辑断言"""
        # 创建用户
        create_response = await async_client.post(
            "/api/v1/users",
            json={"username": "testuser", "email": "test@example.com"}
        )
        
        # 查询用户
        get_response = await async_client.get(
            f"/api/v1/users/{create_response.json()['id']}"
        )
        
        # ✓ 业务逻辑验证
        created_user = create_response.json()
        retrieved_user = get_response.json()
        
        assert created_user["id"] == retrieved_user["id"]
        assert created_user["username"] == retrieved_user["username"]
    
    def test_data_consistency(self, db_session):
        """Level 4: 数据一致性断言"""
        from app.models import User
        from app.services import UserService
        
        service = UserService(db_session)
        
        # 创建用户
        user = service.create_user(
            username="consistency_test",
            email="consistency@example.com",
            password="pass123"
        )
        
        # ✓ 数据库一致性验证
        db_user = db_session.get(User, user.id)
        assert db_user.username == user.username
        assert db_user.email == user.email
        
        # ✓ 重新查询一致性
        reloaded_user = service.get_user_by_id(user.id)
        assert reloaded_user.username == user.username
```

#### 2.4.2 Pytest-Assert 优势

```python
"""
使用 pytest 的 assert 而非 unittest
"""

import pytest


class Test_assertvs_unittest:
    
    # ❌ 不推荐： unittest 风格
    def test_old_style(self):
        import unittest
        self = unittest.TestCase()
        self.assertEqual(1 + 1, 2)  # 错误信息不清晰
    
    # ✅ 推荐：pytest 风格
    def test_pytest_style(self):
        assert 1 + 1 == 2  # ✓ 清晰的错误信息
    
    # ✅ 推荐：带描述的断言
    def test_with_description(self):
        result = calculate_sum(1, 1)
        assert result == 2, f"Expected 2, but got {result}"
    
    # ✅ 推荐：链式断言
    def test_chained_assertions(self):
        user = create_user(username="test")
        
        assert user.id > 0
        assert user.username == "test"
        assert user.email is not None
        assert user.created_at is not None
    
    # ✅ 推荐：异常断言
    def test_exception_assertion(self):
        with pytest.raises(ValueError, match="Invalid username"):
            validate_username("")  # 空用户名应该抛出异常
        
        with pytest.raises(ValidationError, match="email.*invalid"):
            validate_email("not-an-email")
```

---

## 3️⃣ 常见错误与反模式

### 3.1 数据与配置反模式

#### ❌ 反模式 1：硬编码测试数据

```python
"""
❌ 反模式：硬编码测试数据
"""

def test_user_creation():
    # ❌ 硬编码数据，难以维护
    response = client.post(
        "/api/v1/users",
        json={
            "username": "testuser123",
            "email": "test123@example.com",
            "password": "password123"
        }
    )
    
    # ❌ 硬编码 ID
    assert response.json()["id"] == 123


# ✅ 修正：使用 fixture 和动态数据
@pytest.fixture
def sample_user():
    """动态生成测试数据"""
    import uuid
    return {
        "username": f"user_{uuid.uuid4().hex[:8]}",
        "email": f"user_{uuid.uuid4().hex[:8]}@example.com",
        "password": "password123"
    }


def test_user_creation(sample_user):
    response = client.post("/api/v1/users", json=sample_user)
    
    # ✅ 验证返回的 ID（而不是硬编码）
    user_id = response.json()["id"]
    assert user_id > 0
    
    # ✅ 验证数据一致性
    assert response.json()["username"] == sample_user["username"]
```

---

#### ❌ 反模式 2：全局状态依赖

```python
"""
❌ 反模式：依赖全局变量/状态
"""

# 全局状态
created_user_id = None

def test_create_user():
    global created_user_id
    response = client.post("/api/v1/users", json={"username": "test"})
    created_user_id = response.json()["id"]  # ❌ 设置全局状态

def test_get_user():
    # ❌ 依赖上一个测试设置的全局状态
    response = client.get(f"/api/v1/users/{created_user_id}")
    assert response.status_code == 200


# ✅ 修正：使用 fixture 和参数传递
@pytest.fixture
def created_user(client):
    """创建用户并返回"""
    response = client.post("/api/v1/users", json={"username": "test"})
    return response.json()

def test_get_user(client, created_user):
    # ✅ 通过 fixture 传递数据
    response = client.get(f"/api/v1/users/{created_user['id']}")
    assert response.status_code == 200
    assert response.json()["username"] == created_user["username"]


# ✅ 进阶：使用 pytest-order 控制测试顺序（不推荐）
import pytest

@pytest.mark.order(1)
def test_create_user(client):
    response = client.post("/api/v1/users", json={"username": "test1"})
    return response.json()

@pytest.mark.order(2)
def test_get_user(client, test_create_user):
    response = client.get(f"/api/v1/users/{test_create_user['id']}")
    assert response.status_code == 200
```

---

#### ❌ 反模式 3：测试间耦合

```python
"""
❌ 反模式：测试间共享状态
"""

def test_step_1_create_user():
    # 创建用户
    response = client.post("/api/v1/users", json={"username": "user1"})
    assert response.status_code == 201

def test_step_2_update_user():
    # ❌ 假设用户已存在（依赖 test_step_1）
    response = client.put("/api/v1/users/1", json={"username": "updated"})
    assert response.status_code == 200

def test_step_3_delete_user():
    # ❌ 假设用户已存在（依赖 test_step_2）
    response = client.delete("/api/v1/users/1")
    assert response.status_code == 204


# ✅ 修正：每个测试独立，使用 fixture 管理依赖
@pytest.fixture
def user_factory(client):
    """用户工厂 fixture"""
    def create_user(username="testuser"):
        response = client.post(
            "/api/v1/users",
            json={"username": username, "email": f"{username}@example.com"}
        )
        return response.json()
    return create_user

def test_create_user(user_factory):
    # ✅ 独立测试：创建用户
    user = user_factory("test1")
    assert user["username"] == "test1"

def test_update_user(user_factory):
    # ✅ 独立测试：更新用户（ Thanks fixture）
    user = user_factory("test2")
    response = client.put(
        f"/api/v1/users/{user['id']}",
        json={"username": "updated2"}
    )
    assert response.status_code == 200
    assert response.json()["username"] == "updated2"

def test_delete_user(user_factory):
    # ✅ 独立测试：删除用户（Thanks fixture）
    user = user_factory("test3")
    response = client.delete(f"/api/v1/users/{user['id']}")
    assert response.status_code == 204
```

---

### 3.2 Mock 与依赖注入反模式

#### ❌ 反模式 4：过度 Mock

```python
"""
❌ 反模式：过度 Mock（单元测试变成单元"无"测试）
"""

from unittest.mock import Mock, patch

def test_user_service_overmocked():
    # ❌ Mock 了所有依赖，测试没有实际意义
    db_session = Mock()
    user_repo = Mock()
    user_service = UserService(db_session, user_repo)
    
    # ❌ Mock 返回固定值
    user_repo.get_user.return_value = {"id": 1, "username": "mocked"}
    
    result = user_service.get_user(1)
    
    # ❌ 只验证了 Mock 行为，没有验证真实逻辑
    assert result["username"] == "mocked"


# ✅ 修正：选择合适的测试策略
@pytest.fixture
def real_db_session():
    """真实数据库 session（集成测试）"""
    # 使用测试数据库
    ...

def test_user_service_with_real_db(real_db_session):
    # ✅ 集成测试：验证真实逻辑
    service = UserService(real_db_session)
    
    user = service.create_user("real_user", "real@example.com", "pass123")
    retrieved = service.get_user(user["id"])
    
    # ✅ 验证真实数据库操作
    assert retrieved["username"] == "real_user"
    assert retrieved["id"] == user["id"]


# ✅ 修正：适度 Mock（单元测试）
def test_user_service_unit():
    # ✅ 只 Mock 外部依赖（数据库、API）
    from unittest.mock import MagicMock
    
    db_backend = MagicMock()
    db_backend.execute.return_value = {"id": 1, "username": "test_user"}
    
    service = UserService(db_backend)
    result = service.create_user("test", "test@example.com", "pass123")
    
    # ✅ 验证与 Mock 的交互
    db_backend.execute.assert_called_once()
    assert result["username"] == "test_user"
```

---

#### ❌ 反模式 5：Mock 静态方法/类

```python
"""
❌ 反模式：Mock 内置/静态方法
"""

from unittest.mock import patch

def test_password_hashing_overmocked():
    # ❌ Mock 了密码哈希函数
    with patch("app.services.hash_password") as mock_hash:
        mock_hash.return_value = "hashed_password"
        
        result = hash_password("my_password")
        
        # ❌ 只验证了 Mock，没有测试真实逻辑
        assert result == "hashed_password"


# ✅ 修正：测试真实逻辑（或只 Mock 不可控的外部依赖）
def test_password_hashing_real():
    # ✅ 测试真实的密码哈希
    password = "my_password"
    hashed = hash_password(password)
    
    # ✅ 验证哈希结果格式
    assert hashed.startswith("$2b$")  # bcrypt 前缀
    assert len(hashed) > len(password)  # 哈希比原文长
    
    # ✅ 验证一致性
    assert hash_password(password) == hashed


def test_user_creation_with_external_service():
    # ✅ 只 Mock 外部服务（如 SMTP、Stripe）
    with patch("app.services.send_email") as mock_send:
        mock_send.return_value = True
        
        create_user("test", "test@example.com", "pass123")
        
        mock_send.assert_called_once()
```

---

#### ❌ 反模式 6：忽略异常处理

```python
"""
❌ 反模式：忽略异常处理
"""

def test_api_without_exception_handling():
    # ❌ 没有测试异常路径
    response = client.get("/api/v1/users/99999")
    assert response.status_code == 200  # ❌ 假设用户总是存在


# ✅ 修正：全面的异常测试
def test_user_not_found():
    # ✅ 测试资源不存在
    response = client.get("/api/v1/users/99999")
    assert response.status_code == 404
    assert response.json()["detail"] == "User not found"


def test_invalid_input():
    # ✅ 测试无效输入
    response = client.post(
        "/api/v1/users",
        json={"username": "", "email": "invalid-email"}
    )
    assert response.status_code == 422
    errors = response.json()["detail"]
    assert len(errors) > 0


def test_database_connection_error():
    # ✅ 测试数据库连接失败
    with patch("app.db.get_session") as mock_session:
        mock_session.side_effect = ConnectionError("DB connection failed")
        
        response = client.get("/api/v1/users")
        assert response.status_code == 503
        assert "service unavailable" in response.json()["detail"].lower()
```

---

### 3.3 技术实践反模式

#### ❌ 反模式 7：测试命名不规范

```python
"""
❌ 反模式：模糊的测试命名
"""

def test_user():
    # ❌ 测试名太模糊
    ...

def test_1():
    # ❌ 无意义的数字命名
    ...

def test_function():
    # ❌ 泛泛而谈
    ...


# ✅ 修正：清晰的测试命名
class TestUserAPI:
    """用户 API 测试"""
    
    def test_create_user_with_valid_data_returns_201(self):
        # ✅ 描述：What + Expected Result
        ...
    
    def test_create_user_with_duplicate_username_returns_409(self):
        # ✅ 描述：Scenario + Error Code
        ...
    
    def test_get_user_by_id_returns_user_with_profile(self):
        # ✅ 描述：Action + Expected Result
        ...
    
    def test_delete_user_removes_from_database(self):
        # ✅ 描述：Action + Side Effect
        ...
```

---

#### ❌ 反模式 8：测试代码重复

```python
"""
❌ 反模式：测试代码重复（DRY 违反）
"""

def test_create_user_returns_201():
    response = client.post("/api/v1/users", json={...})
    assert response.status_code == 201

def test_update_user_returns_200():
    response = client.put("/api/v1/users/1", json={...})
    assert response.status_code == 200

def test_delete_user_returns_204():
    response = client.delete("/api/v1/users/1")
    assert response.status_code == 204


# ✅ 修正：参数化测试 + 辅助函数
import pytest

@pytest.mark.parametrize(
    "endpoint, method, expected_status",
    [
        ("/api/v1/users", "post", 201),   # 创建
        ("/api/v1/users/1", "put", 200),  # 更新
        ("/api/v1/users/1", "delete", 204), # 删除
    ],
    ids=["create_user", "update_user", "delete_user"]
)
def test_user_crud_operations(endpoint, method, expected_status):
    # ✅ 通用 CRUD 测试
    response = client.request(method, endpoint, json={"username": "test"})
    assert response.status_code == expected_status


# ✅ 进阶： Factory Pattern
class UserFactory:
    """用户数据工厂"""
    @staticmethod
    def create(**overrides):
        data = {
            "username": f"user_{uuid.uuid4().hex[:8]}",
            "email": f"user_{uuid.uuid4().hex[:8]}@example.com",
            "password": "password123"
        }
        data.update(overrides)
        return data

def test_create_user(user_factory):
    user_data = UserFactory.create(username="custom_user")
    response = client.post("/api/v1/users", json=user_data)
    assert response.status_code == 201
```

---

#### ❌ 反模式 9：忽略异步资源清理

```python
"""
❌ 反模式：异步测试资源泄漏
"""

@pytest.mark.asyncio
async def test_async_operation():
    # ❌ 没有清理异步资源
    async with AsyncClient(app=app) as client:
        response = await client.get("/api/v1/data")
        assert response.status_code == 200
    # ❌ session 没有关闭


# ✅ 修正：使用 fixture 管理异步资源
@pytest.fixture
async def async_client():
    """异步客户端 fixture（自动清理）"""
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client
    # ✅ 自动关闭 session


@pytest.mark.asyncio
async def test_async_operation(async_client):
    # ✅ 使用 fixture，无需手动清理
    response = await async_client.get("/api/v1/data")
    assert response.status_code == 200
```

---

#### ❌ 反模式 10：忽略测试性能

```python
"""
❌ 反模式：慢速测试阻塞 CI/CD
"""

# ❌ 每个测试都启动完整应用
def test_api_1():
    app = create_app()
    client = TestClient(app)
    response = client.get("/api/v1/data")
    ...

def test_api_2():
    app = create_app()  # ❌ 重复启动
    client = TestClient(app)
    response = client.get("/api/v1/data")
    ...

def test_api_3():
    app = create_app()  # ❌ 重复启动
    client = TestClient(app)
    response = client.get("/api/v1/data")
    ...


# ✅ 修正：使用 session 级别 fixture
@pytest.fixture(scope="session")
def test_app():
    """会话级应用实例（只创建一次）"""
    app = create_app()
    return app

@pytest.fixture
def client(test_app):
    """客户端 fixture（基于 session app）"""
    return TestClient(test_app)

def test_api_1(client):
    response = client.get("/api/v1/data")
    assert response.status_code == 200

def test_api_2(client):
    response = client.get("/api/v1/data")
    assert response.status_code == 200

def test_api_3(client):
    response = client.get("/api/v1/data")
    assert response.status_code == 200
```

---

## 4️⃣ AI/ML 测试专项规范

### 4.1 模型验证测试

```python
"""
AI/ML 模型验证测试规范
"""

import pytest
import pandas as pd
from sklearn.metrics import (
    accuracy_score, 
    precision_score, 
    recall_score, 
    f1_score,
    roc_auc_score
)


class TestModelValidation:
    """模型验证测试"""
    
    @pytest.fixture
    def model(self):
        """加载训练好的模型"""
        import joblib
        return joblib.load("models/trained_model.pkl")
    
    def test_model__inference(self, model):
        """测试模型推理功能"""
        # 准备测试数据
        X_test = [[5.1, 3.5, 1.4, 0.2]]  # Iris 示例
        
        # 执行推理
        prediction = model.predict(X_test)
       _proba = model.predict_proba(X_test)
        
        # 验证输出格式
        assert isinstance(prediction, (list, numpy.ndarray))
        assert len(prediction) == 1
        assert _proba.shape == (1, 3)  # 3 个类别
    
    def test_model_accuracy_baseline(self, model):
        """测试模型准确率基线"""
        # 加载测试数据
        X_test, y_test = load_test_data("data/test.parquet")
        
        # 执行预测
        y_pred = model.predict(X_test)
        
        # 计算指标
        metrics = {
            "accuracy": accuracy_score(y_test, y_pred),
            "precision": precision_score(y_test, y_pred, average="weighted"),
            "recall": recall_score(y_test, y_pred, average="weighted"),
            "f1": f1_score(y_test, y_pred, average="weighted")
        }
        
        # 验证基线（模型必须达到的最低标准）
        assert metrics["accuracy"] >= 0.95, f"Accuracy too low: {metrics['accuracy']}"
        assert metrics["precision"] >= 0.90
        assert metrics["recall"] >= 0.85
        assert metrics["f1"] >= 0.88
    
    def test_model_response_time(self, model):
        """测试模型响应时间（性能）"""
        import time
        
        X_test = [[5.1, 3.5, 1.4, 0.2] for _ in range(100)]
        
        start_time = time.time()
        for X in X_test:
            model.predict([X])
        elapsed = time.time() - start_time
        
        # 验证 P95 响应时间
        avg_time = elapsed / 100
        assert avg_time < 0.05, f"Average inference time {avg_time}s > 50ms"
    
    def test_model_input_validation(self, model):
        """测试模型输入验证"""
        from pydantic import ValidationError
        
        # 有效输入
        valid_input = [[5.1, 3.5, 1.4, 0.2]]
        result = model.predict(valid_input)
        assert len(result) == 1
        
        # 无效输入：维度错误
        with pytest.raises((ValueError, ValidationError)):
            model.predict([[5.1, 3.5]])  # 缺少特征
    
    def test_model_stability(self, model):
        """测试模型稳定性（相同输入 => 相同输出）"""
        X_test = [[5.1, 3.5, 1.4, 0.2]]
        
        # 多次推理
        results = [model.predict(X_test) for _ in range(10)]
        
        # 验证所有结果一致
        assert all(r[0] == results[0][0] for r in results), "Model is not deterministic"
```

---

### 4.2 数据质量测试

```python
"""
数据质量测试规范
"""

import pytest
import pandas as pd
from great_expectations import expectations as ge


class TestGameDataQuality:
    """游戏数据质量测试"""
    
    @pytest.fixture
    def training_data(self):
        """加载训练数据"""
        return pd.read_parquet("data/training_data.parquet")
    
    @pytest.fixture
    def test_data(self):
        """加载测试数据"""
        return pd.read_parquet("data/test_data.parquet")
    
    def test_data_completeness(self, training_data):
        """测试数据完整性（无缺失值）"""
        # 检查关键字段
        required_columns = ["user_id", "score", "timestamp"]
        
        for col in required_columns:
            assert col in training_data.columns
            assert not training_data[col].isnull().any(), f"Missing values in {col}"
    
    def test_data_validity(self, training_data):
        """测试数据有效性"""
        # 数值范围验证
        assert (training_data["score"] >= 0).all(), "Negative scores found"
        assert (training_data["score"] <= 10000).all(), "Unrealistic scores"
        
        # 时间范围验证
        assert training_data["timestamp"].dtype == "datetime64[ns]"
        assert (training_data["timestamp"] >= "2024-01-01").all()
    
    def test_data_distribution(self, training_data, test_data):
        """测试训练/测试数据分布一致性"""
        from scipy import stats
        
        # Kolmogorov-Smirnov 检验
        for col in ["score", "play_time"]:
            stat, p_value = stats.ks_2samp(
                training_data[col],
                test_data[col]
            )
            
            # p-value > 0.05 表示分布无显著差异
            assert p_value > 0.05, f"Distribution mismatch in {col}"
    
    def test_data_duplicates(self, training_data):
        """测试数据重复值"""
        # 检查主键重复
        assert not training_data["user_id"].duplicated().any(), "Duplicate user_ids found"
    
    def test_data_outliers(self, training_data):
        """测试数据异常值（IQR 方法）"""
        Q1 = training_data["score"].quantile(0.25)
        Q3 = training_data["score"].quantile(0.75)
        IQR = Q3 - Q1
        
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR
        
        outliers = training_data[
            (training_data["score"] < lower_bound) | 
            (training_data["score"] > upper_bound)
        ]
        
        # 异常值比例不应超过 5%
        assert len(outliers) / len(training_data) < 0.05


# ========== Great Expectations 集成 ==========

class TestGEDataValidation:
    """数据验证框架测试"""
    
    def test_game_data_expectations(self):
        """使用 Great Expectations 验证数据"""
        import great_expectations as gx
        
        context = gx.get_context()
        datasource = context.sources.add_pandas(name="pandas_datasource")
        data_asset = datasource.add_dataframe_dataframe_asset(
            name="game_data_asset"
        )
        
        batch_request = data_asset.build_batch_request()
        
        # 创建 expectation suite
        expectation_suite_name = "game_data_validation"
        context.suites.add(gx.ExpectationSuite(name=expectation_suite_name))
        
        # 添加验证规则
       .validator = context.get_validator(
            batch_request=batch_request,
            expectation_suite_name=expectation_suite_name
        )
        
        # 定义期望
        validator.expect_column_values_to_be_unique(column="user_id")
        validator.expect_column_values_to_not_be_null(column="score")
        validator.expect_column_min_to_be_between(
            column="score", 
            min_value=0, 
            max_value=10000
        )
        
        # 保存验证结果
        validator.save_expectation_suite(discard_failed_expectations=False)
        
        # 运行检查
        results = validator.validate()
        assert results.success, "Data validation failed"
```

---

### 4.3 AI Agent 测试

```python
"""
AI Agent 工作流测试规范
"""

import pytest
from langchain.agents import AgentType, initialize_agent
from langchain_openai import ChatOpenAI
from langchain.memory import ConversationBufferMemory


class TestAIAgentWorkflow:
    """AI Agent 工作流测试"""
    
    @pytest.fixture
    def agent_executor(self):
        """初始化 Agent 执行器"""
        llm = ChatOpenAI(temperature=0)
        
        tools = [
            # 搜索工具
            # 计算工具
            # 数据库查询工具
        ]
        
        memory = ConversationMemory(verbose=True)
        
        return initialize_agent(
            tools=tools,
            llm=llm,
            agent=AgentType.CHAT_CONVERSATIONAL_REACT_DESCRIPTION,
            memory=memory,
            verbose=True
        )
    
    def test_agent_can_answer_basic_questions(self, agent_executor):
        """测试 Agent 回答基本问题"""
        response = agent_executor.run("今天星期几？")
        
        assert isinstance(response, str)
        assert len(response) > 0
        assert len(response) < 500  # 防止 token 溢出
    
    def test_agent_can_use_tools(self, agent_executor):
        """测试 Agent 使用工具的能力"""
        response = agent_executor.run("请搜索'AI 测试' relevant information")
        
        # 验证工具被调用
        assert "tool" in agent_executor.memory.chat_memory.messages[-2].content.lower()
        
        # 验证响应包含搜索结果
        assert len(response) > 100  # 应该有详细回答
    
    def test_agent_handles_conversation_history(self, agent_executor):
        """测试 Agent 处理对话历史"""
        # 第一轮对话
        response1 = agent_executor.run("我的名字是 Alice")
        assert "Alice" in response1 or "hello" in response1.lower()
        
        # 第二轮对话（应记住前文）
        response2 = agent_executor.run("我叫什么名字？")
        assert "Alice" in response2
    
    def test_agent_error_handling(self, agent_executor):
        """测试 Agent 错误处理"""
        # 无效查询
        response = agent_executor.run("skdfjsdkfjlsdkfjsdlkfj")
        
        # 应该优雅降级
        assert isinstance(response, str)
        assert len(response) > 0
    
    def test_agent_performance(self, agent_executor):
        """测试 Agent 性能（响应时间）"""
        import time
        
        prompt = "请详细解释什么是机器学习（500 字以上）"
        
        start_time = time.time()
        response = agent_executor.run(prompt)
        elapsed = time.time() - start_time
        
        # 验证响应时间 < 10 秒
        assert elapsed < 10.0, f"Agent response took {elapsed}s > 10s"
        
        # 验证响应长度
        assert len(response) > 500, "Response too short"


# ========== 多 Agent 协作测试 ==========

class TestMultiAgentCollaboration:
    """多 Agent 协作测试"""
    
    @pytest.fixture
    def task_compiler(self):
        """任务编排 Agent"""
        from langchain.agents import AgentType, initialize_agent
        llm = ChatOpenAI(temperature=0)
        
        tools = [compile_task_tool]
        return initialize_agent(
            tools=tools,
            llm=llm,
            agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
            verbose=True
        )
    
    @pytest.fixture
    def data_analyst(self):
        """数据分析 Agent"""
        from langchain.agents import AgentType, initialize_agent
        llm = ChatOpenAI(temperature=0)
        
        tools = [query_database_tool, analyze_data_tool]
        return initialize_agent(
            tools=tools,
            llm=llm,
            agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
            verbose=True
        )
    
    def test_task_decomposition(self, task_compiler):
        """测试任务分解能力"""
        task = "分析过去 30 天的销售数据并生成报告"
        
       分解 = task_compiler.run(f"将任务分解为子任务: {task}")
        
        # 验证子任务
        assert "query" in decomposition.lower()
        assert "analyze" in decomposition.lower()
        assert "report" in decomposition.lower()
    
    def test_multi_agent_workflow(self, task_compiler, data_analyst):
        """测试多 Agent 工作流"""
        # 任务编排
        task = "分析用户行为并生成洞察"
        subtasks = task_compiler.run(f"分解任务: {task}")
        
        # 数据分析
        analysis = data_analyst.run(f"执行分析: {subtasks}")
        
        # 验证结果
        assert "insight" in analysis.lower()
        assert len(analysis) > 200
```

---

## 5️⃣ CI/CD 集成规范

### 5.1 GitHub Actions 配置

```yaml
# .github/workflows/test.yml
name: Python Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  schedule:
    - cron: '0 2 * * *'  # 每天 2AM 运行性能测试

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements-dev.txt
      
      - name: Run unit tests
        run: |
          pytest tests/unit/ -v --tb=short
      
      - name: Run integration tests
        run: |
          pytest tests/integration/ -v --tb=short
        env:
          TEST_DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
      
      - name: Run AI tests (if flagged)
        if: github.ref == 'refs/heads/main'
        run: |
          pytest tests/ai/ -v --run-ai --tb=short
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      
      - name: Run performance tests (nightly)
        if: github.event_name == 'schedule'
        run: |
          pytest tests/performance/ -v --run-performance --tb=short
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.xml
          flags: unittests
      
      - name: Generate test report
        if: always()
        run: |
          pytest --junitxml=report.xml
        continue-on-error: true
      
      - name: Upload test report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: pytest-results
          path: report.xml
          retention-days: 7
```

---

### 5.2 Docker Compose 测试环境

```yaml
# docker-compose.test.yml
version: '3.8'

services:
  app:
    build: .
    environment:
      - DATABASE_URL=postgresql://test:test@db:5432/testdb
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    command: pytest tests/ -v --cov=app --cov-report=xml

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=test
      - POSTGRES_PASSWORD=test
      - POSTGRES_DB=testdb
    ports:
      - "5432:5432"
    volumes:
      - ./data/init-db.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

---

### 5.3 Pytest 插件开发

```python
"""
自定义 pytest 插件：AI 测试增强
"""

import pytest
from datetime import datetime


def pytest_addoption(parser):
    """添加命令行选项"""
    group = parser.getgroup("ai-testing")
    
    group.addoption(
        "--run-ai",
        action="store_true",
        default=False,
        help="Run AI/ML tests"
    )
    
    group.addoption(
        "--ai-model",
        type=str,
        default="gpt-3.5-turbo",
        help="AI model to use for testing"
    )


def pytest_configure(config):
    """注册自定义标记"""
    config.addinivalue_line(
        "markers", "ai: AI/ML tests"
    )
    config.addinivalue_line(
        "markers", "slow: Slow running tests"
    )


@pytest.fixture
def ai_model(request):
    """AI 模型 fixture"""
    model_name = request.config.getoption("--ai-model")
    
    if model_name == "gpt-3.5-turbo":
        from langchain_openai import ChatOpenAI
        return ChatOpenAI(model="gpt-3.5-turbo")
    elif model_name == "gpt-4":
        from langchain_openai import ChatOpenAI
        return ChatOpenAI(model="gpt-4")
    else:
        raise ValueError(f"Unknown AI model: {model_name}")


class AIMark:
    """AI 测试标记插件"""
    def __init__(self):
        self.results = {}
    
    def pytest_runtest_logstart(self, nodeid, location):
        """记录测试开始"""
        self.results[nodeid] = {
            "start": datetime.now().isoformat(),
            "status": "running"
        }
    
    def pytest_runtest_logreport(self, report):
        """记录测试报告"""
        if report.when == "call":
            self.results[report.nodeid].update({
                "status": "passed" if report.passed else "failed",
                "duration": report.duration,
                "end": datetime.now().isoformat()
            })
```

---

## 6️⃣ 附录

### 6.1 常用测试装饰器

```python
"""
常用测试装饰器速查
"""

import pytest
from functools import wraps


# ========== 重试装饰器 ==========

def retry(max_attempts=3, delay=1):
    """重试装饰器（处理 flaky tests）"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        import time
                        time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator


@pytest.mark.asyncio
@retry(max_attempts=3, delay=0.5)
async def test_flaky_api_call(async_client):
    """重试测试（处理网络波动）"""
    response = await async_client.get("/api/v1/data")
    assert response.status_code == 200


# ========== 超时装饰器 ==========

def timeout(seconds=10):
    """超时装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            import signal
            
            def timeout_handler(signum, frame):
                raise TimeoutError(f"Test timed out after {seconds}s")
            
            signal.signal(signal.SIGALRM, timeout_handler)
            signal.alarm(seconds)
            
            try:
                result = func(*args, **kwargs)
            finally:
                signal.alarm(0)
            
            return result
        return wrapper
    return decorator


@timeout(seconds=5)
def test_slow_function():
    """超时测试"""
    import time
    time.sleep(10)  # 这个会超时


# ========== 标记跳过装饰器 ==========

@pytest.mark.skipif(not os.getenv("CI"), reason="Only run in CI")
def test_ci_only_feature():
    """仅在 CI 运行的测试"""
    ...


@pytest.mark.skip(reason="Feature not implemented yet")
def test_future_feature():
    """跳过测试（功能未实现）"""
    ...


# ========== 参数化求积装饰器 ==========

@pytest.mark.parametrize("a", [1, 2, 3])
@pytest.mark.parametrize("b", [4, 5])
def test_combinatorial(a, b):
    """组合测试（a=3 种, b=2 种 => 6 个测试）"""
    result = a + b
    assert result in [5, 6, 7, 8]
```

---

### 6.2 性能基准测试

```python
"""
性能基准测试规范
"""

import pytest
import time


@pytest.mark.performance
class TestAPIPerformance:
    """API 性能基准测试"""
    
    def test_endpoint_response_time(self, async_client):
        """测试端点响应时间"""
        start = time.time()
        response = await async_client.get("/api/v1/users")
        elapsed = time.time() - start
        
        assert response.status_code == 200
        assert elapsed < 1.0, f"Response time {elapsed}s > 1s"
    
    def test_endpoint_throughput(self, async_client):
        """测试端点吞吐量（每秒请求数）"""
        import asyncio
        
        async def make_request():
            return await async_client.get("/api/v1/users")
        
        start = time.time()
        tasks = [make_request() for _ in range(100)]
        await asyncio.gather(*tasks)
        elapsed = time.time() - start
        
        requests_per_second = 100 / elapsed
        assert requests_per_second > 50, f"Throughput {requests_per_second} req/s < 50"
    
    def test_db_query_performance(self, db_session):
        """测试数据库查询性能"""
        import time
        
        start = time.time()
        users = db_session.query(User).all()
        elapsed = time.time() - start
        
        assert len(users) > 0
        assert elapsed < 0.5, f"Query took {elapsed}s > 500ms"


# ========== pytest-benchmark 插件 ==========
# pip install pytest-benchmark

@pytest.mark.benchmark(
    min_time=0.1,
    max_time=0.5,
    rounds=5
)
def test_benchmark_user_creation(benchmark, db_session):
    """使用 pytest-benchmark 进行精确测量"""
    def setup():
        return {"db": db_session}
    
    def test_user_creation(db):
        from app.models import User
        user = User(username="benchmark_user", email="bench@example.com")
        db.add(user)
        db.commit()
        return user
    
    benchmark.pedantic(test_user_creation, setup=setup, rounds=5)
```

---

### 6.3 测试覆盖率报告

```bash
# pytest.ini 中配置
[pytest]
addopts = 
    --cov=app
    --cov-report=term-missing
    --cov-report=html:htmlcov
    --cov-report=xml:coverage.xml
    --cov-fail-under=80
```

```python
# 覆盖率排除配置
# .coveragerc

[run]
source = app

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    if self.debug:
    raise AssertionError
    raise NotImplementedError
    if 0:
    if __name__ == .__main__.:

omit =
    */tests/*
    */__pycache__/*
    */migrations/*
```

---

## 📚 推荐阅读与资源

### 官方文档

- **pytest**: https://docs.pytest.org/
- **FastAPI Testing**: https://fastapi.tiangolo.com/tutorial/testing/
- **Pydantic Testing**: https://docs.pydantic.dev/latest/concepts/testing/
- **Great Expectations**: https://docs.greatexpectations.io/

### 书籍推荐

1. **"Python Testing with pytest"** -Brian Okken
2. **"Clean Code"** -Robert C. Martin（测试章节）
3. **"The Art of Software Testing"** - Glenford J. Myers

### 社区资源

- **Awesome Testing**: https://github.com/TestingBrowserless/awesome-testing
- **Pytest Plugins**: https://github.com/pytest-dev/pytest-plugins

---

## 📄 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-04-22 | 初始版本，AI Testing 项目定制 |

---

**本规范由 AI Testing 项目维护**  
**最后更新：2026-04-22**
