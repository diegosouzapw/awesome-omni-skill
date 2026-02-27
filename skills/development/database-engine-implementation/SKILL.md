---
name: database-engine-implementation
description: Guide for implementing database engines in IterableData. Use when adding support for new SQL or NoSQL databases, implementing DBDriver classes, or extending database capabilities.
---

# Database Engine Implementation Guide

## Overview

Database engines provide read-only access to SQL and NoSQL databases as iterable data sources. They wrap database drivers to provide a unified `BaseIterable` interface.

## Architecture

- **DBDriver** (`iterable/db/base.py`) - Abstract base class for database drivers
- **DatabaseIterable** (`iterable/db/iterable.py`) - Wrapper that makes DBDriver work as BaseIterable
- **Driver Registry** (`iterable/db/__init__.py`) - Registry for mapping engine names to driver classes

## Adding New Database Engines

### Step-by-Step Process

1. **Create driver file**: `iterable/db/<engine>.py`
2. **Implement DBDriver**: Inherit from `DBDriver` in `iterable/db/base.py`
3. **Required methods**: `connect()`, `iterate()`, `close()` (close has default implementation)
4. **Register driver**: Call `register_driver()` in `iterable/db/__init__.py`
5. **Add detection**: Update `iterable/helpers/detect.py` to recognize database URLs
6. **Create tests**: `tests/test_db_engines.py` or add to existing test file
7. **Update dependencies**: Add optional dependency to `pyproject.toml`
8. **Update documentation**: Add engine to docs

### Implementation Pattern

```python
from collections.abc import Iterator
from typing import Any
from ..types import Row
from .base import DBDriver

class NewEngineDriver(DBDriver):
    """Database driver for NewEngine.
    
    Supports streaming queries using batch processing.
    """
    
    def __init__(self, source: str | Any, **kwargs: Any) -> None:
        """Initialize driver.
        
        Args:
            source: Connection string/URL or existing connection object
            **kwargs: Additional parameters:
                - query: Query string or table name
                - batch_size: Rows per batch (default: 10000)
                - on_error: Error handling policy ('raise', 'skip', 'warn')
        """
        super().__init__(source, **kwargs)
        self._cursor: Any = None
    
    def connect(self) -> None:
        """Establish database connection.
        
        Raises:
            ImportError: If required driver library is not installed
            ConnectionError: If connection fails
        """
        try:
            import database_library
        except ImportError:
            raise ImportError(
                "database-library is required. Install with: pip install database-library"
            ) from None
        
        # Handle existing connection object
        if hasattr(self.source, "cursor"):
            self.conn = self.source
            self._connected = True
            return
        
        # Parse connection string
        if not isinstance(self.source, str):
            raise ValueError("Source must be connection string or connection object")
        
        try:
            self.conn = database_library.connect(self.source, **self.kwargs.get("connect_args", {}))
            self._connected = True
        except Exception as e:
            self._connected = False
            raise ConnectionError(f"Failed to connect: {e}") from e
    
    def iterate(self) -> Iterator[Row]:
        """Return iterator of dict rows.
        
        Yields:
            dict: Database row as dictionary
            
        Raises:
            RuntimeError: If not connected
        """
        if not self._connected:
            raise RuntimeError("Not connected. Call connect() first.")
        
        self._start_metrics()
        batch_size = self.kwargs.get("batch_size", 10000)
        
        try:
            query = self._build_query()
            cursor = self.conn.cursor()
            
            # Execute query with batching
            cursor.execute(query)
            
            while True:
                rows = cursor.fetchmany(batch_size)
                if not rows:
                    break
                
                # Convert rows to dicts
                column_names = [desc[0] for desc in cursor.description]
                for row in rows:
                    row_dict = dict(zip(column_names, row))
                    self._update_metrics(rows_read=1)
                    yield row_dict
                    
        except Exception as e:
            self._handle_error(e, "during iteration")
            if self._on_error == "raise":
                raise
```

## Key Implementation Details

### Connection Handling

- Support both connection strings and existing connection objects
- Parse connection strings appropriately (URL format)
- Handle connection errors gracefully with clear messages
- Set `self._connected = True` on success

### Query Building

- Accept `query` parameter (SQL query or table name)
- If table name provided, build SELECT query
- Support filtering, projection, sorting via kwargs
- Use parameterized queries to prevent SQL injection

### Iteration

- Use batch processing (`batch_size` parameter)
- Convert database rows to dict objects
- Update metrics: `self._update_metrics(rows_read=1)`
- Handle errors according to `on_error` policy
- Start metrics: `self._start_metrics()` before iteration

### Error Handling

Three policies (set via `on_error` kwarg):
- `'raise'` - Raise exceptions (default)
- `'skip'` - Skip problematic rows
- `'warn'` - Warn and continue

Use `self._handle_error(error, context)` for consistent handling.

### Metrics Tracking

Automatic metrics available via `self.metrics`:
- `rows_read` - Number of rows read
- `bytes_read` - Bytes read (may be None)
- `elapsed_seconds` - Time elapsed

Update during iteration: `self._update_metrics(rows_read=count)`

## Registration

Register driver in `iterable/db/__init__.py`:

```python
from .newengine import NewEngineDriver

register_driver("newengine", NewEngineDriver)
```

## Format Detection

Update `iterable/helpers/detect.py` to recognize database URLs:

```python
def detect_file_type(filename, content=None):
    # Database URL detection
    if filename.startswith(("postgresql://", "postgres://")):
        return "database"
    if filename.startswith("newengine://"):
        return "database"
    # ... existing detection
```

## Testing Requirements

### Test Structure

```python
import pytest
from iterable.db.newengine import NewEngineDriver

class TestNewEngineDriver:
    def test_connect(self):
        driver = NewEngineDriver("newengine://localhost/db")
        driver.connect()
        assert driver.is_connected
        driver.close()
    
    def test_iterate(self):
        driver = NewEngineDriver("newengine://localhost/db", query="SELECT * FROM table")
        driver.connect()
        rows = list(driver.iterate())
        assert len(rows) > 0
        assert isinstance(rows[0], dict)
        driver.close()
    
    def test_batch_processing(self):
        driver = NewEngineDriver("newengine://localhost/db", query="SELECT * FROM table", batch_size=100)
        driver.connect()
        # Verify batching works correctly
        driver.close()
    
    def test_error_handling(self):
        driver = NewEngineDriver("newengine://invalid", query="SELECT * FROM table", on_error="skip")
        # Test error handling policies
```

### Test Coverage

- Connection with string and object sources
- Query building (table name vs SQL)
- Batch processing
- Error handling policies
- Metrics tracking
- Context manager support (`with` statement)
- Missing dependencies (should raise ImportError)

## Dependencies

### Optional Dependencies

Add to `pyproject.toml`:

```toml
[project.optional-dependencies]
newengine = ["database-library>=1.0.0"]
```

### Import Handling

Handle missing dependencies:

```python
try:
    import database_library
except ImportError:
    raise ImportError(
        "newengine support requires 'database-library'. "
        "Install with: pip install iterabledata[newengine]"
    ) from None
```

## Examples

Look at existing implementations:
- `iterable/db/postgres.py` - SQL database example (PostgreSQL)
- `iterable/db/mongo.py` - NoSQL database example (MongoDB)
- `iterable/db/clickhouse.py` - Columnar database example
- `iterable/db/elasticsearch.py` - Search engine example

## Common Patterns

### SQL Databases

- Use server-side cursors for memory efficiency
- Support read-only transactions
- Handle connection pooling
- Use parameterized queries

### NoSQL Databases

- Support query filters and projections
- Handle pagination/cursors
- Support aggregation pipelines
- Batch document retrieval

### Connection String Parsing

```python
from urllib.parse import urlparse

def parse_connection_string(conn_str: str) -> dict:
    parsed = urlparse(conn_str)
    return {
        "host": parsed.hostname,
        "port": parsed.port,
        "database": parsed.path.lstrip("/"),
        "username": parsed.username,
        "password": parsed.password,
    }
```

## Common Pitfalls

- **Memory issues**: Always use batch processing, don't load all rows at once
- **Connection leaks**: Always close connections, use context managers
- **Error handling**: Implement all three error policies consistently
- **Metrics**: Update metrics during iteration, not after
- **Reset support**: Most databases don't support reset (set `_reset_supported = False`)
- **Read-only**: Default to read-only access for safety
