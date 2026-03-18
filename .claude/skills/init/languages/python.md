# Python Guidelines

## Core Principles

- Follow PEP 8; use ruff for formatting and linting
- Type annotations: all public function parameters and return values must have type annotations
- Use `pathlib.Path` instead of `os.path`
- Use f-strings instead of `format()` and `%`

## Naming

- Classes: `PascalCase` (`UserService`, `DataProcessor`)
- Functions and variables: `snake_case` (`get_user_by_id`, `is_valid`)
- Constants: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`)
- Private members: single underscore prefix `_internal_method`
- No double-underscore name mangling (`__private`) unless there is a clear reason

## Type Annotations

```python
# 禁止
def process(data, config):
    ...

# 正确
def process(data: list[dict[str, Any]], config: ProcessConfig) -> ProcessResult:
    ...
```

- Use `X | None` instead of `Optional[X]` (Python 3.10+)
- Use `list[str]` instead of `List[str]` (Python 3.9+)
- Use `TypeAlias` or `TypedDict` for complex types
- Use `Protocol` to define structural subtypes instead of ABCs

## Error Handling

- Catch specific exceptions; no bare `except:` or `except Exception:`
- Custom business exceptions should inherit from a specific built-in exception class
- Use `raise ... from e` to preserve the exception chain

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

## Async

- Use `async/await` for asynchronous code; do not mix threads and coroutines
- Use `asyncio.TaskGroup` for concurrent execution (Python 3.11+)
- Use `contextlib.asynccontextmanager` to manage async resources

## Data Classes

- Use `dataclass` or `pydantic.BaseModel` for simple data containers
- Use `@dataclass(frozen=True)` for immutable data
- Use `pydantic.BaseSettings` for configuration objects

## Project Structure

- Use `pyproject.toml` for project configuration (not `setup.py`)
- Use pytest for testing; place configuration in `[tool.pytest]` within `pyproject.toml`
- Use `uv` or `poetry` for dependency management
