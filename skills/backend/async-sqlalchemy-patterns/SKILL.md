---
name: async-sqlalchemy-patterns
description: Async SQLAlchemy 2.0+ database patterns for FastAPI including session management, connection pooling, Alembic migrations, relationship loading strategies, and query optimization. Use when implementing database models, configuring async sessions, setting up migrations, optimizing queries, managing relationships, or when user mentions SQLAlchemy, async database, ORM, Alembic, database performance, or connection pooling.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
---

# Async SQLAlchemy Patterns

**Purpose:** Implement production-ready async SQLAlchemy 2.0+ patterns in FastAPI with proper session management, migrations, and performance optimization.

**Activation Triggers:**
- Database model implementation
- Async session configuration
- Alembic migration setup
- Query performance optimization
- Relationship loading issues
- Connection pool configuration
- Transaction management
- Database schema migrations

**Key Resources:**
- `scripts/setup-alembic.sh` - Initialize Alembic with async support
- `scripts/generate-migration.sh` - Create migrations from model changes
- `templates/base_model.py` - Base model with common patterns
- `templates/session_manager.py` - Async session factory and dependency
- `templates/alembic.ini` - Alembic configuration for async
- `examples/user_model.py` - Complete model with relationships
- `examples/async_context_examples.py` - Session usage patterns

## Core Patterns

### 1. Async Engine and Session Setup

**Database Configuration:**

```python
# app/core/database.py
from sqlalchemy.ext.asyncio import (
    AsyncSession,
    create_async_engine,
    async_sessionmaker
)
from sqlalchemy.orm import declarative_base
from app.core.config import settings

# Create async engine
engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_pre_ping=True,  # Verify connections before using
    pool_size=5,         # Base number of connections
    max_overflow=10,     # Additional connections when pool is full
    pool_recycle=3600,   # Recycle connections after 1 hour
    pool_timeout=30,     # Wait 30s for available connection
)

# Create async session factory
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,  # Keep objects usable after commit
    autoflush=False,         # Manual flush control
    autocommit=False,        # Explicit commits only
)

Base = declarative_base()
```

**Dependency Injection Pattern:**

```python
# app/core/deps.py
from typing import AsyncGenerator
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import AsyncSessionLocal

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """
    FastAPI dependency for database sessions.
    Automatically handles cleanup.
    """
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

### 2. Base Model Pattern

Use template from `templates/base_model.py` with:
- UUID primary keys by default
- Automatic created_at/updated_at timestamps
- Soft delete support
- Common query methods

**Essential Mixins:**

```python
from datetime import datetime
from sqlalchemy import DateTime, Boolean, func
from sqlalchemy.orm import Mapped, mapped_column

class TimestampMixin:
    """Auto-managed timestamps"""
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False
    )

class SoftDeleteMixin:
    """Soft delete support"""
    is_deleted: Mapped[bool] = mapped_column(
        Boolean,
        default=False,
        nullable=False
    )
    deleted_at: Mapped[datetime | None] = mapped_column(
        DateTime(timezone=True),
        nullable=True
    )
```

### 3. Relationship Loading Strategies

**Lazy Loading (Default - N+1 Problem Risk):**
```python
# Bad: Causes N+1 queries
users = await session.execute(select(User))
for user in users.scalars():
    print(user.posts)  # Separate query per user!
```

**Eager Loading (Recommended):**

```python
from sqlalchemy.orm import selectinload, joinedload

# selectinload: Separate query, good for one-to-many
stmt = select(User).options(selectinload(User.posts))
result = await session.execute(stmt)
users = result.scalars().all()

# joinedload: SQL JOIN, good for many-to-one
stmt = select(Post).options(joinedload(Post.author))
result = await session.execute(stmt)
posts = result.scalars().unique().all()  # unique() required with joins!

# subqueryload: Subquery loading
stmt = select(User).options(subqueryload(User.posts))
```

**Relationship Configuration:**

```python
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = "users"

    # One-to-many with cascade
    posts: Mapped[list["Post"]] = relationship(
        back_populates="author",
        cascade="all, delete-orphan",
        lazy="selectin"  # Default eager loading
    )

    # Many-to-many
    roles: Mapped[list["Role"]] = relationship(
        secondary="user_roles",
        back_populates="users",
        lazy="selectin"
    )
```

### 4. Query Patterns

**Basic CRUD:**

```python
from sqlalchemy import select, update, delete
from sqlalchemy.ext.asyncio import AsyncSession

# Create
async def create_user(session: AsyncSession, user_data: dict):
    user = User(**user_data)
    session.add(user)
    await session.commit()
    await session.refresh(user)
    return user

# Read with filters
async def get_users(session: AsyncSession, skip: int = 0, limit: int = 100):
    stmt = (
        select(User)
        .where(User.is_deleted == False)
        .offset(skip)
        .limit(limit)
        .order_by(User.created_at.desc())
    )
    result = await session.execute(stmt)
    return result.scalars().all()

# Update
async def update_user(session: AsyncSession, user_id: int, updates: dict):
    stmt = (
        update(User)
        .where(User.id == user_id)
        .values(**updates)
        .returning(User)
    )
    result = await session.execute(stmt)
    await session.commit()
    return result.scalar_one()

# Delete
async def delete_user(session: AsyncSession, user_id: int):
    stmt = delete(User).where(User.id == user_id)
    await session.execute(stmt)
    await session.commit()
```

**Complex Queries:**

```python
from sqlalchemy import func, and_, or_

# Aggregations
stmt = (
    select(User.id, func.count(Post.id))
    .join(Post)
    .group_by(User.id)
    .having(func.count(Post.id) > 5)
)

# Subqueries
subq = (
    select(func.avg(Post.views))
    .where(Post.user_id == User.id)
    .scalar_subquery()
)
stmt = select(User).where(User.id.in_(
    select(Post.user_id).where(Post.views > subq)
))

# Window functions
from sqlalchemy import over
stmt = select(
    Post,
    func.row_number().over(
        partition_by=Post.user_id,
        order_by=Post.created_at.desc()
    ).label('row_num')
).where(over.row_num <= 10)
```

### 5. Transaction Management

**Nested Transactions:**

```python
async def transfer_funds(
    session: AsyncSession,
    from_account: int,
    to_account: int,
    amount: float
):
    async with session.begin_nested():  # Savepoint
        # Debit
        stmt = (
            update(Account)
            .where(Account.id == from_account)
            .where(Account.balance >= amount)
            .values(balance=Account.balance - amount)
        )
        result = await session.execute(stmt)
        if result.rowcount == 0:
            raise ValueError("Insufficient funds")

        # Credit
        stmt = (
            update(Account)
            .where(Account.id == to_account)
            .values(balance=Account.balance + amount)
        )
        await session.execute(stmt)

    await session.commit()
```

**Manual Transaction Control:**

```python
async def batch_operation(session: AsyncSession, items: list):
    try:
        for item in items:
            session.add(item)
            if len(session.new) >= 100:
                await session.flush()  # Flush but don't commit

        await session.commit()
    except Exception as e:
        await session.rollback()
        raise
```

### 6. Alembic Migration Setup

```bash
# Initialize Alembic with async template
./scripts/setup-alembic.sh

# Creates:
# - alembic/ directory
# - alembic.ini configured for async
# - env.py with async support
```

**Generate Migrations:**

```bash
# Auto-generate from model changes
./scripts/generate-migration.sh "add user table"

# Review generated migration in alembic/versions/
# Always review auto-generated migrations!
```

**Migration Best Practices:**

1. **Always review auto-generated migrations**
2. **Use batch operations for large tables**
3. **Add indexes in separate migrations**
4. **Include both upgrade and downgrade**
5. **Test migrations on staging first**

**Manual Migration Template:**

```python
# alembic/versions/xxx_add_index.py
from alembic import op
import sqlalchemy as sa

def upgrade() -> None:
    # Use batch for SQLite compatibility
    with op.batch_alter_table('users') as batch_op:
        batch_op.create_index(
            'ix_users_email',
            ['email'],
            unique=True
        )

def downgrade() -> None:
    with op.batch_alter_table('users') as batch_op:
        batch_op.drop_index('ix_users_email')
```

### 7. Connection Pool Configuration

**Production Settings:**

```python
# For web applications
engine = create_async_engine(
    DATABASE_URL,
    pool_size=20,           # Concurrent requests
    max_overflow=40,        # Burst capacity
    pool_recycle=3600,      # 1 hour
    pool_pre_ping=True,     # Verify connections
    pool_timeout=30,        # Wait time
    echo_pool=False,        # Pool debug logging
)

# For background tasks
engine = create_async_engine(
    DATABASE_URL,
    pool_size=5,            # Fewer connections
    max_overflow=10,
    pool_recycle=7200,      # 2 hours
)
```

**Connection Health Check:**

```python
from sqlalchemy import text

async def check_database_health(session: AsyncSession) -> bool:
    try:
        await session.execute(text("SELECT 1"))
        return True
    except Exception:
        return False
```

### 8. Performance Optimization

**Bulk Operations:**

```python
# Bulk insert
async def bulk_create_users(session: AsyncSession, users: list[dict]):
    stmt = insert(User).values(users)
    await session.execute(stmt)
    await session.commit()

# Bulk update
async def bulk_update_status(session: AsyncSession, user_ids: list[int]):
    stmt = (
        update(User)
        .where(User.id.in_(user_ids))
        .values(status="active")
    )
    await session.execute(stmt)
    await session.commit()
```

**Query Result Streaming:**

```python
async def stream_large_dataset(session: AsyncSession):
    stmt = select(User).execution_options(yield_per=100)
    result = await session.stream(stmt)

    async for partition in result.partitions(100):
        users = partition.scalars().all()
        # Process batch
        yield users
```

**Index Optimization:**

```python
from sqlalchemy import Index

class User(Base):
    __tablename__ = "users"

    email: Mapped[str] = mapped_column(unique=True, index=True)

    __table_args__ = (
        Index('ix_user_status_created', 'status', 'created_at'),
        Index('ix_user_email_active', 'email', postgresql_where=~is_deleted),
    )
```

## Troubleshooting

### Common Issues

**"Object is not bound to session":**
```python
# Bad: Object expires after commit
user = await create_user(session, data)
print(user.email)  # Error if expire_on_commit=True

# Fix: Refresh or configure session
await session.refresh(user)
# Or: AsyncSessionLocal(expire_on_commit=False)
```

**"N+1 Query Problem":**
```python
# Enable SQL logging to detect
engine = create_async_engine(url, echo=True)

# Fix with eager loading
stmt = select(User).options(selectinload(User.posts))
```

**"DetachedInstanceError":**
```python
# Bad: Accessing relationship outside session
async with AsyncSessionLocal() as session:
    user = await session.get(User, user_id)
print(user.posts)  # Error!

# Fix: Load within session or use eager loading
stmt = select(User).options(selectinload(User.posts))
```

**"Connection Pool Exhausted":**
```python
# Increase pool size or add timeout
engine = create_async_engine(
    url,
    pool_size=20,
    max_overflow=40,
    pool_timeout=30
)
```

## Resources

**Scripts:** `scripts/` directory contains:
- `setup-alembic.sh` - Initialize Alembic with async configuration
- `generate-migration.sh` - Create migrations from model changes
- `test-connection.sh` - Test database connectivity
- `optimize-queries.sh` - Analyze and suggest query optimizations

**Templates:** `templates/` directory includes:
- `base_model.py` - Base model with UUID, timestamps, soft delete
- `session_manager.py` - Complete session factory and dependencies
- `alembic.ini` - Production-ready Alembic configuration
- `env.py` - Alembic async environment template

**Examples:** `examples/` directory provides:
- `user_model.py` - Full model with relationships and indexes
- `async_context_examples.py` - Session patterns and best practices
- `query_patterns.py` - Common query optimization examples
- `migration_examples.py` - Manual migration patterns

---

**SQLAlchemy Version:** 2.0+
**Database Support:** PostgreSQL, MySQL, SQLite (async drivers)
**Async Drivers:** asyncpg (PostgreSQL), aiomysql (MySQL), aiosqlite (SQLite)
**Version:** 1.0.0
