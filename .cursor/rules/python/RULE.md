---
description: Python conventions for data engineering pipelines and applications
globs:
  - "**/*.py"
alwaysApply: false
---

# Python Development Standards

## Project Structure

```
project/
  ├── src/
  │   └── package_name/
  │       ├── __init__.py
  │       ├── main.py
  │       ├── models/
  │       ├── services/
  │       └── utils/
  ├── tests/
  │   ├── conftest.py
  │   ├── unit/
  │   └── integration/
  ├── pyproject.toml
  └── requirements.txt
```

## Type Hints

### Always Use Type Hints
```python
from typing import Optional, List, Dict, Any
from collections.abc import Sequence, Mapping

def process_users(
    users: List[User],
    filter_active: bool = True,
) -> Dict[str, User]:
    """Process and index users by ID."""
    result: Dict[str, User] = {}
    for user in users:
        if filter_active and not user.is_active:
            continue
        result[user.id] = user
    return result
```

### Use Modern Type Syntax (Python 3.10+)
```python
# Good: Modern syntax
def get_user(user_id: str) -> User | None:
    ...

# Instead of
def get_user(user_id: str) -> Optional[User]:
    ...
```

## Data Classes and Pydantic

### Pydantic for Validation
```python
from pydantic import BaseModel, Field, field_validator

class User(BaseModel):
    id: str
    email: str
    age: int = Field(ge=0, le=150)
    
    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if '@' not in v:
            raise ValueError('Invalid email')
        return v.lower()
```

### Dataclasses for Simple Structures
```python
from dataclasses import dataclass, field

@dataclass
class Config:
    host: str
    port: int = 8080
    debug: bool = False
    tags: list[str] = field(default_factory=list)
```

## Error Handling

```python
class UserNotFoundError(Exception):
    """Raised when a user cannot be found."""
    def __init__(self, user_id: str):
        self.user_id = user_id
        super().__init__(f"User not found: {user_id}")

def get_user(user_id: str) -> User:
    user = db.query(User).filter_by(id=user_id).first()
    if user is None:
        raise UserNotFoundError(user_id)
    return user
```

## Testing with Pytest

### Test Structure
```python
import pytest
from unittest.mock import Mock, patch

class TestUserService:
    @pytest.fixture
    def user_service(self) -> UserService:
        return UserService(repository=Mock())
    
    def test_get_user_returns_user_when_found(self, user_service: UserService):
        # Arrange
        expected_user = User(id="123", name="Alice")
        user_service.repository.get.return_value = expected_user
        
        # Act
        result = user_service.get("123")
        
        # Assert
        assert result == expected_user
        user_service.repository.get.assert_called_once_with("123")
    
    def test_get_user_raises_when_not_found(self, user_service: UserService):
        user_service.repository.get.return_value = None
        
        with pytest.raises(UserNotFoundError):
            user_service.get("invalid")
```

### Parametrized Tests
```python
@pytest.mark.parametrize("input_value,expected", [
    ("hello", "HELLO"),
    ("World", "WORLD"),
    ("", ""),
])
def test_uppercase(input_value: str, expected: str):
    assert uppercase(input_value) == expected
```

### Fixtures
```python
# conftest.py
@pytest.fixture
def db_session():
    session = create_test_session()
    yield session
    session.rollback()
    session.close()

@pytest.fixture
def sample_user(db_session) -> User:
    user = User(id="test-123", name="Test User")
    db_session.add(user)
    db_session.commit()
    return user
```

## Data Pipelines

### Pandas Patterns
```python
import pandas as pd

def transform_sales_data(df: pd.DataFrame) -> pd.DataFrame:
    """Transform raw sales data."""
    return (
        df
        .pipe(clean_column_names)
        .pipe(filter_valid_records)
        .assign(
            revenue=lambda x: x['quantity'] * x['price'],
            month=lambda x: pd.to_datetime(x['date']).dt.to_period('M'),
        )
        .groupby('month', as_index=False)
        .agg({'revenue': 'sum', 'quantity': 'sum'})
    )
```

### Polars for Performance
```python
import polars as pl

def transform_large_dataset(df: pl.DataFrame) -> pl.DataFrame:
    return (
        df
        .filter(pl.col("status") == "active")
        .with_columns([
            (pl.col("quantity") * pl.col("price")).alias("revenue"),
            pl.col("date").str.to_datetime().dt.month().alias("month"),
        ])
        .group_by("month")
        .agg([
            pl.col("revenue").sum(),
            pl.col("quantity").sum(),
        ])
    )
```

## Logging

```python
import logging
import structlog

# Configure structured logging
structlog.configure(
    processors=[
        structlog.stdlib.filter_by_level,
        structlog.stdlib.add_logger_name,
        structlog.stdlib.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ],
    wrapper_class=structlog.stdlib.BoundLogger,
    context_class=dict,
    logger_factory=structlog.stdlib.LoggerFactory(),
)

logger = structlog.get_logger(__name__)

def process_order(order_id: str) -> None:
    logger.info("processing_order", order_id=order_id)
    try:
        # Process...
        logger.info("order_processed", order_id=order_id)
    except Exception as e:
        logger.error("order_failed", order_id=order_id, error=str(e))
        raise
```

## Dependencies

Use `pyproject.toml` for modern projects:
```toml
[project]
name = "my-project"
version = "1.0.0"
dependencies = [
    "pydantic>=2.0",
    "pandas>=2.0",
    "structlog>=23.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "mypy>=1.0",
    "ruff>=0.1",
]
```

## Linting and Formatting

Use `ruff` for linting and formatting:
```bash
ruff check .
ruff format .
```

