---
name: adding-api-sources
description: Use when implementing a new data source adapter for metapyle, before writing any source code
---

# Adding API Sources to Metapyle

## Overview

Add new financial data source adapters following TDD and established patterns. Each source provides `fetch()` and `get_metadata()` methods with lazy imports for optional dependencies.

**Core principle:** Use `brainstorming` skill first for design decisions, then implement following established patterns.

## Workflow

1. **Design** - Use `brainstorming` skill to decide data model mapping
2. **Plan** - Use `writing-plans` skill for implementation plan
3. **Implement** - Follow TDD with subagents (see Quick Reference)

## Design Questions (Brainstorming Phase)

Before coding, answer these questions using the `brainstorming` skill:

| Question | Why It Matters |
|----------|----------------|
| What maps to `symbol`? | Primary identifier (ticker, bbid, series ID) |
| What maps to `field`? | Secondary identifier if needed (PX_LAST, dataset::column) |
| Need `params` field? | Extra filters (tenor, location, deltaStrike) |
| Authentication model? | External (user calls auth) or internal (credentials passed) |
| Batch strategy? | Single call for all symbols, or group by some key? |
| Column naming? | Symbol only, or symbol::field for uniqueness? |
| Metadata available? | What can `get_metadata()` return? |

## Quick Reference

| Step | Files | Key Actions |
|------|-------|-------------|
| 1. Branch | — | `git checkout -b feature/<source>-source` |
| 2. Skeleton | `sources/<source>.py` | Lazy import + class with `NotImplementedError` |
| 3. Export | `sources/__init__.py` | Add import + `__all__` |
| 4. Tests | `tests/unit/test_sources_<source>.py` | Mock-based tests (RED) |
| 5. Implement | `sources/<source>.py` | `fetch()` then `get_metadata()` (GREEN) |
| 6. Config | `pyproject.toml` | Optional dep + mypy ignore |
| 7. Verify | — | pytest, mypy, ruff |

## Batch Fetch API

Sources receive batched requests via `Sequence[FetchRequest]`:

```python
from collections.abc import Sequence
from metapyle.sources.base import BaseSource, FetchRequest, make_column_name, register_source

@register_source("<source>")
class <Source>Source(BaseSource):
    def fetch(
        self,
        requests: Sequence[FetchRequest],
        start: str,
        end: str,
    ) -> pd.DataFrame:
        """
        Parameters
        ----------
        requests : Sequence[FetchRequest]
            Each has: symbol, field (optional), path (optional), params (optional)
        start, end : str
            ISO dates (YYYY-MM-DD)
            
        Returns
        -------
        pd.DataFrame
            DatetimeIndex, columns named via make_column_name(symbol, field)
        """
        if not requests:
            return pd.DataFrame()
        # ... implementation
```

## FetchRequest Fields

```python
@dataclass(frozen=True, slots=True, kw_only=True)
class FetchRequest:
    symbol: str                          # Required - primary identifier
    field: str | None = None             # Optional - e.g., "PX_LAST", "dataset::col"
    path: str | None = None              # Optional - for localfile source
    params: dict[str, Any] | None = None # Optional - extra filters
```

## Column Naming

Always use `make_column_name()` for output columns:

```python
from metapyle.sources.base import make_column_name

# In fetch(), rename columns:
for req in requests:
    col_name = make_column_name(req.symbol, req.field)  # "AAPL::PX_LAST" or "AAPL"
    result[col_name] = data[req.symbol]
```

## Batch Grouping Pattern

When API requires grouping (e.g., by dataset):

```python
def fetch(self, requests: Sequence[FetchRequest], start: str, end: str) -> pd.DataFrame:
    # Group by some key (dataset_id, field type, etc.)
    groups: dict[str, list[FetchRequest]] = {}
    for req in requests:
        key = extract_key(req.field)  # Your grouping logic
        groups.setdefault(key, []).append(req)
    
    # Fetch each group (potentially in parallel)
    result_dfs: list[pd.DataFrame] = []
    for key, group_requests in groups.items():
        symbols = [req.symbol for req in group_requests]
        df = api.batch_fetch(key, symbols, start, end)
        result_dfs.append(df)
    
    # Merge results
    result = result_dfs[0]
    for df in result_dfs[1:]:
        result = result.join(df, how="outer")
    return result
```

## Lazy Import Pattern

```python
_LIB_AVAILABLE: bool | None = None
_lib_modules: dict[str, Any] = {}

def _get_lib() -> dict[str, Any]:
    """Lazy import of library modules."""
    global _LIB_AVAILABLE, _lib_modules
    if _LIB_AVAILABLE is None:
        try:
            from library import Module1, Module2
            _lib_modules = {"Module1": Module1, "Module2": Module2}
            _LIB_AVAILABLE = True
        except (ImportError, Exception):
            _lib_modules = {}
            _LIB_AVAILABLE = False
    return _lib_modules
```

## Exception Handling

```python
try:
    data = api.fetch(symbols, start, end)
except (FetchError, NoDataError):
    raise  # Re-raise our exceptions as-is
except Exception as e:
    logger.error("fetch_failed: symbols=%s, error=%s", symbols, str(e))
    raise FetchError(f"API error: {e}") from e

if data.empty:
    raise NoDataError(f"No data returned for {symbols}")
```

## Test Pattern

```python
class TestSourceFetch:
    def test_single_request(self) -> None:
        with patch("metapyle.sources.<source>._get_lib") as mock_get:
            mock_lib = {"API": MagicMock()}
            mock_lib["API"].fetch.return_value = mock_data
            mock_get.return_value = mock_lib

            source = <Source>Source()
            requests = [FetchRequest(symbol="SYM", field="FIELD")]
            df = source.fetch(requests, "2024-01-01", "2024-12-31")

            assert "SYM::FIELD" in df.columns
            assert isinstance(df.index, pd.DatetimeIndex)
```

## pyproject.toml

```toml
[project.optional-dependencies]
<source> = ["<library>"]

[[tool.mypy.overrides]]
module = ["<library>", "<library>.*"]
ignore_missing_imports = true
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong `fetch()` signature | Must be `fetch(requests: Sequence[FetchRequest], start, end)` |
| Import at module level | Use lazy import pattern with `_get_lib()` |
| Manual column naming | Use `make_column_name(symbol, field)` |
| f-strings in logging | Use `logger.debug("msg: %s", var)` |
| Missing empty request check | Return `pd.DataFrame()` if `not requests` |
| Catching exceptions silently | Re-raise `FetchError`/`NoDataError`, wrap others |

## TDD Order

1. **RED:** Write test for `_get_lib()` (library not installed)
2. **GREEN:** Implement lazy import
3. **RED:** Write test for single request fetch
4. **GREEN:** Implement basic fetch
5. **RED:** Write test for batch fetch
6. **GREEN:** Implement batch handling
7. **RED:** Write error handling tests
8. **GREEN:** Implement error handling
9. **VERIFY:** Run full test suite, ruff, mypy
