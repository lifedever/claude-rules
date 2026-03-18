# Python 规范

## 基本原则

- 遵循 PEP 8，用 ruff 做格式化和 lint
- 类型标注：所有公共函数的参数和返回值必须有类型标注
- 用 `pathlib.Path` 替代 `os.path`
- 用 f-string 替代 `format()` 和 `%`

## 命名

- 类：`PascalCase`（`UserService`、`DataProcessor`）
- 函数和变量：`snake_case`（`get_user_by_id`、`is_valid`）
- 常量：`UPPER_SNAKE_CASE`（`MAX_RETRY_COUNT`）
- 私有成员：单下划线前缀 `_internal_method`
- 禁止双下划线 name mangling（`__private`），除非有明确理由

## 类型标注

```python
# 禁止
def process(data, config):
    ...

# 正确
def process(data: list[dict[str, Any]], config: ProcessConfig) -> ProcessResult:
    ...
```

- 用 `X | None` 替代 `Optional[X]`（Python 3.10+）
- 用 `list[str]` 替代 `List[str]`（Python 3.9+）
- 复杂类型用 `TypeAlias` 或 `TypedDict`
- 用 `Protocol` 定义结构化子类型，替代 ABC

## 错误处理

- 捕获具体异常，禁止裸 `except:` 和 `except Exception:`
- 自定义业务异常继承自具体的内置异常类
- 用 `raise ... from e` 保留异常链

```python
# 禁止
try:
    result = call_api()
except:
    pass

# 正确
try:
    result = call_api()
except httpx.TimeoutException as e:
    raise ServiceUnavailableError(f"API timeout: {e.url}") from e
```

## 异步

- 异步代码用 `async/await`，不要混用线程和协程
- 用 `asyncio.TaskGroup` 并发执行（Python 3.11+）
- 用 `contextlib.asynccontextmanager` 管理异步资源

## 数据类

- 简单数据容器用 `dataclass` 或 `pydantic.BaseModel`
- 不可变数据用 `@dataclass(frozen=True)`
- 配置对象用 `pydantic.BaseSettings`

## 项目结构

- 用 `pyproject.toml` 管理项目配置（不要 `setup.py`）
- 测试用 pytest，配置放 `pyproject.toml` 的 `[tool.pytest]`
- 依赖管理用 `uv` 或 `poetry`
