# Python Application Template

## Purpose
This template provides comprehensive guidance for building Python applications with modern best practices, type safety, and proper project structure.

## Use Case
Use when starting a new Python project for web applications, APIs, data processing, or automation tools.

## Template

```
Create a Python application with the following specifications:

## Application Overview
[Describe the application's purpose and main functionality]

## Technical Stack
- Python Version: 3.11+ (or 3.10+ minimum)
- Framework: [FastAPI / Flask / Django / None]
- Dependency Management: [Poetry / Pipenv / pip-tools]
- Type Checking: mypy with strict mode
- Code Quality: ruff (linter + formatter)
- Testing: pytest with coverage
- Documentation: Sphinx or MkDocs

## Architecture Principles
1. Follow PEP 8 style guide
2. Use type hints throughout
3. Implement proper error handling
4. Follow SOLID principles
5. Separate concerns (MVC or layered architecture)
6. Use dataclasses or Pydantic models
7. Apply dependency injection

## Project Structure

### FastAPI/Flask Application
```
project_name/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── main.py              # Application entry point
│       ├── api/                 # API routes/endpoints
│       │   ├── __init__.py
│       │   ├── v1/
│       │   │   ├── __init__.py
│       │   │   ├── users.py
│       │   │   └── auth.py
│       │   └── dependencies.py  # Shared dependencies
│       ├── core/                # Core configuration
│       │   ├── __init__.py
│       │   ├── config.py        # Settings and configuration
│       │   ├── security.py      # Auth and security
│       │   └── logging.py       # Logging configuration
│       ├── models/              # Database models
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── base.py
│       ├── schemas/             # Pydantic schemas (DTOs)
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── common.py
│       ├── services/            # Business logic layer
│       │   ├── __init__.py
│       │   ├── user_service.py
│       │   └── auth_service.py
│       ├── repositories/        # Data access layer
│       │   ├── __init__.py
│       │   └── user_repository.py
│       ├── utils/               # Utility functions
│       │   ├── __init__.py
│       │   └── helpers.py
│       └── exceptions.py        # Custom exceptions
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # Pytest fixtures
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/                     # Utility scripts
├── docs/                        # Documentation
├── .env.example                 # Environment variables template
├── pyproject.toml              # Project configuration
├── Dockerfile
└── README.md
```

### Library/Package
```
project_name/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── exceptions.py
├── tests/
├── docs/
├── examples/
├── pyproject.toml
└── README.md
```

## Code Style & Best Practices

### Type Hints
```python
from typing import Optional, List, Dict, Any, Union
from collections.abc import Sequence

# Always use type hints for function signatures
def process_users(
    users: Sequence[User],
    max_count: int = 100,
    include_inactive: bool = False
) -> List[ProcessedUser]:
    """Process a sequence of users."""
    processed: List[ProcessedUser] = []
    for user in users:
        if not include_inactive and not user.is_active:
            continue
        processed.append(ProcessedUser.from_user(user))
    return processed[:max_count]

# Use generic types for containers
def get_user_by_id(user_id: int) -> Optional[User]:
    """Retrieve user by ID."""
    user = db.query(User).filter(User.id == user_id).first()
    return user

# Use Protocol for structural typing
from typing import Protocol

class Authenticator(Protocol):
    def authenticate(self, credentials: Credentials) -> bool: ...

# Use TypedDict for dictionaries with known structure
from typing import TypedDict

class UserDict(TypedDict):
    id: int
    name: str
    email: str
```

### Error Handling
```python
# Custom exception hierarchy
class ApplicationError(Exception):
    """Base exception for application errors."""
    def __init__(self, message: str, code: str) -> None:
        self.message = message
        self.code = code
        super().__init__(message)

class ValidationError(ApplicationError):
    """Raised when validation fails."""
    def __init__(self, message: str, field: Optional[str] = None) -> None:
        super().__init__(message, "VALIDATION_ERROR")
        self.field = field

class NotFoundError(ApplicationError):
    """Raised when resource is not found."""
    def __init__(self, resource: str, identifier: Any) -> None:
        message = f"{resource} with id {identifier} not found"
        super().__init__(message, "NOT_FOUND")

# Use context managers for resource management
from contextlib import contextmanager
from typing import Generator

@contextmanager
def get_db_session() -> Generator[Session, None, None]:
    """Provide a transactional database session."""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()

# Usage
def create_user(user_data: UserCreate) -> User:
    """Create a new user."""
    with get_db_session() as session:
        user = User(**user_data.dict())
        session.add(user)
        return user
```

### Dataclasses and Pydantic Models
```python
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional

# Use dataclasses for internal data structures
@dataclass(frozen=True)  # Immutable
class User:
    id: int
    username: str
    email: str
    created_at: datetime = field(default_factory=datetime.utcnow)
    is_active: bool = True

# Use Pydantic for API validation and serialization
from pydantic import BaseModel, EmailStr, Field, validator

class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=50)
    email: EmailStr
    password: str = Field(..., min_length=8)
    
    @validator('username')
    def username_alphanumeric(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError('Username must be alphanumeric')
        return v

class UserResponse(BaseModel):
    id: int
    username: str
    email: EmailStr
    created_at: datetime
    
    class Config:
        orm_mode = True  # Allow ORM model conversion
```

### Dependency Injection
```python
from typing import Protocol
from abc import ABC, abstractmethod

# Define interfaces
class UserRepository(Protocol):
    def get_by_id(self, user_id: int) -> Optional[User]: ...
    def save(self, user: User) -> User: ...

# Implementation
class SQLUserRepository:
    def __init__(self, session: Session) -> None:
        self.session = session
    
    def get_by_id(self, user_id: int) -> Optional[User]:
        return self.session.query(User).filter(User.id == user_id).first()
    
    def save(self, user: User) -> User:
        self.session.add(user)
        self.session.commit()
        return user

# Service with injected dependency
class UserService:
    def __init__(self, repository: UserRepository) -> None:
        self.repository = repository
    
    def get_user(self, user_id: int) -> Optional[User]:
        return self.repository.get_by_id(user_id)
```

### Testing with pytest
```python
import pytest
from typing import Generator
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

# Fixtures
@pytest.fixture
def db_session() -> Generator[Session, None, None]:
    """Provide a test database session."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    SessionLocal = sessionmaker(bind=engine)
    session = SessionLocal()
    try:
        yield session
    finally:
        session.close()

@pytest.fixture
def user_repository(db_session: Session) -> SQLUserRepository:
    """Provide a user repository with test database."""
    return SQLUserRepository(db_session)

# Test cases
def test_create_user(user_repository: SQLUserRepository) -> None:
    """Test user creation."""
    # Arrange
    user_data = UserCreate(
        username="testuser",
        email="test@example.com",
        password="password123"
    )
    
    # Act
    user = user_repository.create(user_data)
    
    # Assert
    assert user.id is not None
    assert user.username == "testuser"
    assert user.email == "test@example.com"

def test_get_user_not_found(user_repository: SQLUserRepository) -> None:
    """Test retrieving non-existent user."""
    # Act & Assert
    user = user_repository.get_by_id(999)
    assert user is None

@pytest.mark.parametrize("username,expected", [
    ("user1", True),
    ("user-2", False),
    ("user_3", False),
    ("123user", True),
])
def test_username_validation(username: str, expected: bool) -> None:
    """Test username validation."""
    if expected:
        user = UserCreate(
            username=username,
            email="test@example.com",
            password="password123"
        )
        assert user.username == username
    else:
        with pytest.raises(ValueError):
            UserCreate(
                username=username,
                email="test@example.com",
                password="password123"
            )
```

### Configuration Management
```python
from pydantic import BaseSettings, Field
from functools import lru_cache

class Settings(BaseSettings):
    """Application settings."""
    app_name: str = "MyApp"
    debug: bool = False
    database_url: str = Field(..., env="DATABASE_URL")
    secret_key: str = Field(..., env="SECRET_KEY")
    api_key: str = Field(..., env="API_KEY")
    
    # API settings
    api_v1_prefix: str = "/api/v1"
    max_connections: int = 100
    
    # Security
    access_token_expire_minutes: int = 30
    
    class Config:
        env_file = ".env"
        case_sensitive = False

@lru_cache()
def get_settings() -> Settings:
    """Get cached settings instance."""
    return Settings()

# Usage
settings = get_settings()
print(f"Database URL: {settings.database_url}")
```

### Async/Await (for async frameworks)
```python
import asyncio
from typing import List
import httpx

async def fetch_user(user_id: int) -> User:
    """Fetch user asynchronously."""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/users/{user_id}")
        response.raise_for_status()
        return User(**response.json())

async def fetch_multiple_users(user_ids: List[int]) -> List[User]:
    """Fetch multiple users concurrently."""
    tasks = [fetch_user(user_id) for user_id in user_ids]
    return await asyncio.gather(*tasks)

# Database async operations
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine

async def get_user_async(session: AsyncSession, user_id: int) -> Optional[User]:
    """Get user asynchronously from database."""
    result = await session.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()
```

## Tools and Quality Checks

### pyproject.toml (Poetry)
```toml
[tool.poetry]
name = "my-project"
version = "0.1.0"
description = "My Python project"
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
fastapi = "^0.104.0"
pydantic = "^2.0.0"
sqlalchemy = "^2.0.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.4.0"
pytest-cov = "^4.1.0"
mypy = "^1.5.0"
ruff = "^0.1.0"

[tool.ruff]
line-length = 100
target-version = "py311"
select = ["E", "F", "I", "N", "W", "UP", "B", "A", "C4", "SIM"]

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = "test_*.py"
python_functions = "test_*"
addopts = "--cov=src --cov-report=html --cov-report=term"

[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise NotImplementedError",
    "if TYPE_CHECKING:",
]
```

### Makefile
```makefile
.PHONY: install test lint format type-check all clean

install:
	poetry install

test:
	poetry run pytest -v

test-cov:
	poetry run pytest --cov=src --cov-report=html --cov-report=term

lint:
	poetry run ruff check src tests

format:
	poetry run ruff format src tests

type-check:
	poetry run mypy src

all: format lint type-check test

clean:
	rm -rf .pytest_cache .coverage htmlcov .mypy_cache .ruff_cache
	find . -type d -name __pycache__ -exec rm -rf {} +
```

## Best Practices Summary

1. **Always use type hints** - Enable mypy strict mode
2. **Write docstrings** - Use Google or NumPy style
3. **Test thoroughly** - Aim for 80%+ coverage
4. **Use dataclasses/Pydantic** - For structured data
5. **Handle errors properly** - Custom exception hierarchy
6. **Follow PEP 8** - Use ruff for linting and formatting
7. **Dependency injection** - For testability and flexibility
8. **Configuration via environment** - Never hardcode secrets
9. **Logging over print** - Use structured logging
10. **Document your code** - README, docstrings, and type hints
```

## Example Prompt

```
Create a FastAPI application for a task management system with:

Features:
- User authentication (JWT)
- CRUD operations for tasks
- Task filtering and search
- File upload for task attachments
- PostgreSQL database with SQLAlchemy
- Pydantic models for validation
- Comprehensive error handling
- Unit and integration tests (pytest)
- Type hints throughout
- OpenAPI documentation

Use:
- Python 3.11
- FastAPI with async/await
- SQLAlchemy 2.0 (async)
- Poetry for dependency management
- Pydantic v2 for data validation
- pytest with fixtures
- Ruff for linting
- mypy for type checking

Include proper project structure, configuration management, and follow Python best practices.
```

## References
- [PEP 8 - Style Guide](https://peps.python.org/pep-0008/)
- [Type Hints - PEP 484](https://peps.python.org/pep-0484/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [pytest Documentation](https://docs.pytest.org/)
- [Python Packaging Guide](https://packaging.python.org/)
