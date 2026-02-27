---
name: python-patterns
description: Python patterns, type hints, async/await, and best practices for production code
license: MIT
compatibility: opencode
---

# Python Patterns Skill

## Overview

This skill provides guidelines for writing production-ready Python code following modern Python best practices, type hints, async patterns, and framework-specific conventions for FastAPI, Django, and data processing.

## Core Principles

### 1. Type Hints

Always use type hints for function signatures and complex data structures.

```python
# DO: Use type hints for all function parameters and return types
from typing import Optional, List, Dict, Union
from dataclasses import dataclass

def process_user(user_id: int, name: str, active: bool = True) -> Dict[str, Union[str, int, bool]]:
    """Process a user and return their data."""
    return {"id": user_id, "name": name, "active": active}

# DO: Use dataclasses for data containers
@dataclass
class User:
    id: int
    name: str
    email: str
    active: bool = True

# DO: Use Optional for nullable types
def find_user(user_id: int) -> Optional[User]:
    """Find a user by ID. Returns None if not found."""
    # Implementation
    return None

# DO: Use Union for multiple possible types
def parse_value(value: Union[str, int, float]) -> float:
    """Parse a value to float."""
    return float(value)
```

### 2. Error Handling

Use exceptions with proper context and custom exception classes.

```python
# DO: Create custom exceptions for domain errors
class ValidationError(Exception):
    """Raised when input validation fails."""
    pass

class NotFoundError(Exception):
    """Raised when a resource is not found."""
    pass

# DO: Provide context in error messages
def get_user(user_id: int) -> User:
    """Get a user by ID."""
    try:
        user = database.get(user_id)
    except DatabaseError as e:
        raise NotFoundError(f"User with ID {user_id} not found") from e
    
    if user is None:
        raise NotFoundError(f"User with ID {user_id} not found")
    
    return user

# DO: Use try-except-finally for resource cleanup
def process_file(filename: str) -> str:
    """Process a file and return its contents."""
    file = None
    try:
        file = open(filename, 'r')
        return file.read()
    except FileNotFoundError:
        raise ValidationError(f"File {filename} not found")
    finally:
        if file:
            file.close()

# BETTER: Use context managers
from pathlib import Path

def process_file(filename: str) -> str:
    """Process a file and return its contents."""
    try:
        return Path(filename).read_text()
    except FileNotFoundError:
        raise ValidationError(f"File {filename} not found")
```

### 3. Async/Await Patterns

Use asyncio for concurrent operations.

```python
import asyncio
from typing import List
import aiohttp

# DO: Use async/await for I/O operations
async def fetch_data(url: str) -> dict:
    """Fetch data from a URL asynchronously."""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()

# DO: Use asyncio.gather for concurrent operations
async def fetch_multiple(urls: List[str]) -> List[dict]:
    """Fetch data from multiple URLs concurrently."""
    tasks = [fetch_data(url) for url in urls]
    return await asyncio.gather(*tasks)

# DO: Use async context managers
class DatabaseConnection:
    async def __aenter__(self):
        self.conn = await create_connection()
        return self.conn
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def query_database():
    async with DatabaseConnection() as conn:
        result = await conn.execute("SELECT * FROM users")
        return result

# DO: Handle async iteration
async def process_items(items: List[dict]):
    """Process items asynchronously."""
    for item in items:
        await process_single_item(item)
```

## FastAPI Patterns

### 1. Project Structure

```
myapp/
├── app/
│   ├── __init__.py
│   ├── main.py          # Application entry point
│   ├── config.py        # Configuration settings
│   ├── models/          # Database models
│   ├── schemas/         # Pydantic schemas
│   ├── routers/         # API route handlers
│   ├── services/        # Business logic
│   ├── dependencies.py  # FastAPI dependencies
│   └── utils/           # Utility functions
├── tests/
├── requirements.txt
└── Dockerfile
```

### 2. Dependency Injection

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import Generator

app = FastAPI()

# DO: Use dependency injection for database sessions
def get_db() -> Generator[Session, None, None]:
    """Get database session."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# DO: Create reusable dependencies
async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    """Get current authenticated user."""
    user = await verify_token(token)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@app.get("/users/me")
async def read_current_user(current_user: User = Depends(get_current_user)):
    return current_user

# DO: Use dependencies for common parameters
from fastapi import Query

async def pagination_params(
    skip: int = Query(0, ge=0),
    limit: int = Query(100, ge=1, le=1000)
):
    return {"skip": skip, "limit": limit}

@app.get("/items/")
async def read_items(pagination: dict = Depends(pagination_params)):
    return {"skip": pagination["skip"], "limit": pagination["limit"]}
```

### 3. Pydantic Schemas

```python
from pydantic import BaseModel, Field, validator
from typing import Optional
from datetime import datetime

# DO: Use Pydantic for request/response validation
class UserBase(BaseModel):
    email: str = Field(..., regex=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    name: str = Field(..., min_length=1, max_length=100)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain at least one digit')
        return v

class UserResponse(UserBase):
    id: int
    created_at: datetime
    
    class Config:
        orm_mode = True

# DO: Use response models
tags=["users"])
async def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = User(**user.dict())
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

## Django Patterns

### 1. Project Structure

```
myproject/
├── config/                 # Project configuration
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py        # Base settings
│   │   ├── development.py
│   │   └── production.py
│   ├── urls.py
│   └── wsgi.py
├── apps/                   # Django applications
│   ├── users/
│   ├── blog/
│   └── common/
├── templates/
├── static/
├── media/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   └── production.txt
└── manage.py
```

### 2. Models

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
from django.core.validators import EmailValidator

# DO: Use custom User model
class User(AbstractUser):
    """Custom user model."""
    email = models.EmailField(unique=True, validators=[EmailValidator])
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

# DO: Use model managers for complex queries
class ArticleManager(models.Manager):
    def published(self):
        return self.filter(status='published', published_at__isnull=False)
    
    def by_author(self, author):
        return self.filter(author=author)

class Article(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='articles')
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='draft')
    published_at = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    objects = ArticleManager()
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'published_at']),
        ]
    
    def __str__(self):
        return self.title
```

### 3. Views and Serializers (DRF)

```python
from rest_framework import viewsets, serializers, permissions
from rest_framework.decorators import action
from rest_framework.response import Response

# DO: Use ModelSerializer
class ArticleSerializer(serializers.ModelSerializer):
    author_name = serializers.CharField(source='author.username', read_only=True)
    
    class Meta:
        model = Article
        fields = ['id', 'title', 'content', 'author', 'author_name', 'status', 'published_at']
        read_only_fields = ['author']

# DO: Use ViewSets for CRUD operations
class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_queryset(self):
        """Filter articles based on query parameters."""
        queryset = Article.objects.all()
        status = self.request.query_params.get('status')
        if status:
            queryset = queryset.filter(status=status)
        return queryset
    
    def perform_create(self, serializer):
        """Set the author to the current user."""
        serializer.save(author=self.request.user)
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        """Custom action to publish an article."""
        article = self.get_object()
        article.status = 'published'
        article.save()
        return Response({'status': 'article published'})
```

## Testing Patterns

### 1. pytest Best Practices

```python
import pytest
from unittest.mock import Mock, patch
from fastapi.testclient import TestClient

# DO: Use fixtures for setup
@pytest.fixture
def client():
    """Create test client."""
    from app.main import app
    return TestClient(app)

@pytest.fixture
def mock_db():
    """Mock database session."""
    return Mock()

# DO: Use parametrize for multiple test cases
@pytest.mark.parametrize("input_value,expected", [
    ("hello", 5),
    ("world", 5),
    ("", 0),
])
def test_string_length(input_value, expected):
    assert len(input_value) == expected

# DO: Mock external dependencies
@patch('app.services.email_service.send_email')
def test_user_registration(mock_send_email, client):
    """Test user registration with mocked email."""
    response = client.post("/users/", json={
        "email": "test@example.com",
        "password": "SecurePass123"
    })
    assert response.status_code == 201
    mock_send_email.assert_called_once()

# DO: Test async code
@pytest.mark.asyncio
async def test_async_function():
    result = await async_operation()
    assert result is not None
```

### 2. Test Structure

```python
# DO: Group tests in classes
class TestUserService:
    """Test suite for user service."""
    
    def test_create_user(self, mock_db):
        """Test creating a new user."""
        # Arrange
        user_data = {"email": "test@example.com", "password": "pass123"}
        
        # Act
        user = UserService.create_user(mock_db, user_data)
        
        # Assert
        assert user.email == user_data["email"]
        mock_db.add.assert_called_once()
    
    def test_create_user_duplicate_email(self, mock_db):
        """Test creating user with duplicate email raises error."""
        # Arrange
        mock_db.query.return_value.filter.return_value.first.return_value = Mock()
        
        # Act & Assert
        with pytest.raises(ValidationError, match="Email already exists"):
            UserService.create_user(mock_db, {"email": "exists@example.com"})
```

## Data Processing Patterns

### 1. Pandas Best Practices

```python
import pandas as pd
import numpy as np
from typing import Optional

# DO: Use type hints with pandas
def process_sales_data(df: pd.DataFrame) -> pd.DataFrame:
    """Process sales data and return cleaned dataframe."""
    # Remove duplicates
    df = df.drop_duplicates()
    
    # Handle missing values
    df['revenue'] = df['revenue'].fillna(0)
    
    # Type conversion
    df['date'] = pd.to_datetime(df['date'])
    
    return df

# DO: Use vectorized operations (much faster than loops)
def calculate_metrics(df: pd.DataFrame) -> pd.DataFrame:
    """Calculate business metrics."""
    # Vectorized operation
    df['profit_margin'] = (df['revenue'] - df['cost']) / df['revenue']
    
    # DON'T: Use iterrows() for simple calculations
    # for idx, row in df.iterrows():
    #     df.loc[idx, 'profit_margin'] = (row['revenue'] - row['cost']) / row['revenue']
    
    return df

# DO: Use method chaining
def clean_and_transform(df: pd.DataFrame) -> pd.DataFrame:
    """Clean and transform data using method chaining."""
    return (df
        .dropna(subset=['user_id'])
        .assign(revenue=lambda x: x['price'] * x['quantity'])
        .query('revenue > 0')
        .sort_values('revenue', ascending=False)
        .reset_index(drop=True)
    )
```

## Performance Best Practices

### 1. Use Generators for Large Data

```python
from typing import Iterator

# DO: Use generators for memory efficiency
def read_large_file(filepath: str) -> Iterator[str]:
    """Read large file line by line."""
    with open(filepath, 'r') as f:
        for line in f:
            yield line.strip()

# Usage
for line in read_large_file('large_file.txt'):
    process_line(line)

# DO: Use generator expressions
squares = (x**2 for x in range(1000000))  # Generator
# vs
squares_list = [x**2 for x in range(1000000)]  # List (uses more memory)
```

### 2. Caching

```python
from functools import lru_cache
import functools

# DO: Use LRU cache for expensive operations
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    """Calculate fibonacci number with caching."""
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# DO: Use cache for API calls
import requests
from functools import wraps
import time

def cache_with_ttl(seconds: int):
    """Decorator to cache function results with TTL."""
    def decorator(func):
        cache = {}
        
        @wraps(func)
        def wrapper(*args, **kwargs):
            key = str(args) + str(kwargs)
            if key in cache:
                result, timestamp = cache[key]
                if time.time() - timestamp < seconds:
                    return result
            
            result = func(*args, **kwargs)
            cache[key] = (result, time.time())
            return result
        
        return wrapper
    return decorator

@cache_with_ttl(seconds=300)
def fetch_exchange_rate(currency: str) -> float:
    """Fetch exchange rate with 5-minute cache."""
    response = requests.get(f"https://api.example.com/rates/{currency}")
    return response.json()['rate']
```

## When to Use This Skill

Use this skill when:
- Writing or reviewing Python code
- Setting up FastAPI or Django projects
- Implementing async/await patterns
- Processing data with Pandas
- Writing tests with pytest
- Optimizing Python performance

## Related Skills

- `@testing-strategies` - General testing patterns
- `@api-rest-design` - API design principles
- `@postgresql-patterns` - Database patterns
- `@feature-development` - Feature development workflow
