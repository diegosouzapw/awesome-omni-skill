---
name: affinity-python-sdk
description: Use when writing Python code with the Affinity SDK, or when user asks about "affinity-sdk", "affinity package", typed IDs, async Affinity client, pagination, or Python scripts for Affinity CRM.
---

# Affinity Python SDK

Use this skill when writing Python scripts to interact with Affinity CRM.

## IMPORTANT: Write Operations Require Explicit User Request

**Always use read-only mode by default.** Only allow writes when the user explicitly requests data modification.

```python
from affinity.policies import Policies, WritePolicy

# DEFAULT: Read-only mode (prevents accidental data modification)
with Affinity.from_env(policies=Policies(write=WritePolicy.DENY)) as client:
    ...  # Write operations will raise WriteNotAllowedError

# ONLY when user explicitly approves writes:
with Affinity.from_env() as client:
    ...
```

## Installation

```bash
# SDK only (Python API wrapper)
pip install affinity-sdk

# SDK with .env file support
pip install "affinity-sdk[dotenv]"
```

## Client Initialization

```python
from affinity import Affinity, AsyncAffinity
from affinity.policies import Policies, WritePolicy

# RECOMMENDED: Read-only with .env file
with Affinity.from_env(load_dotenv=True, policies=Policies(write=WritePolicy.DENY)) as client:
    me = client.whoami()
    companies = client.companies.all()

# Async client
async with AsyncAffinity.from_env(policies=Policies(write=WritePolicy.DENY)) as client:
    companies = await client.companies.all()
```

## Typed IDs (ALWAYS USE)

Prevent mixing up entity types by using typed IDs:

```python
from affinity.types import (
    PersonId, CompanyId, ListId, ListEntryId,
    OpportunityId, FieldId, NoteId, UserId
)

# CORRECT:
person = client.persons.get(PersonId(123))
company = client.companies.get(CompanyId(456))
entries = client.lists.entries(ListId(789))

# WRONG - will cause type errors:
person = client.persons.get(123)  # Don't do this!
```

## Pagination Patterns

```python
# Single page (default 100 items)
page = client.companies.list(limit=50)
for company in page.data:
    process(company)
# For next page, use pages() iterator instead

# All items as list (default max 100,000)
all_companies = client.companies.all()

# Adjust limit
companies = client.companies.all(limit=1000)

# Disable limit (use with caution!)
companies = client.companies.all(limit=None)

# Memory-efficient iterator (large datasets)
for person in client.persons.iter():
    process(person)

# Page-by-page iteration
for page in client.companies.pages():
    for company in page.data:
        process(company)

# Progress callback
from affinity import PaginationProgress

def log_progress(p: PaginationProgress) -> None:
    print(f"Page {p.page_number}: {p.items_so_far} items")

for company in client.companies.all(on_progress=log_progress):
    ...
```

## Filtering (Custom Fields Only)

**Note:** Filtering on `persons`, `companies`, `opportunities` is server-side (efficient). Filtering on **list entries** is client-side (fetches all data first) - use saved views for large lists.

```python
from affinity import F

# Simple comparisons
client.persons.list(filter=F.field("Department").equals("Sales"))
client.companies.list(filter=F.field("Industry").contains("Tech"))
client.persons.list(filter=F.field("Title").starts_with("VP"))
client.opportunities.list(filter=F.field("Amount").greater_than(100000))

# Null checks
client.persons.list(filter=F.field("Manager").is_null())
client.persons.list(filter=F.field("Email").is_not_null())

# Boolean logic: AND (&), OR (|), NOT (~)
active_sales = client.persons.list(
    filter=F.field("Department").equals("Sales") & F.field("Status").equals("Active")
)

tech_or_finance = client.companies.list(
    filter=F.field("Industry").equals("Tech") | F.field("Industry").equals("Finance")
)

non_archived = client.persons.list(
    filter=~F.field("Archived").equals(True)
)

# In list
multi_region = client.companies.list(
    filter=F.field("Region").in_list(["US", "Canada", "Mexico"])
)
```

**Cannot filter on built-in fields**: `type`, `firstName`, `lastName`, `primaryEmail`, `name`, `domain` - fetch all, filter client-side.

## Services Reference

```python
with Affinity.from_env() as client:
    # Core entities
    client.persons.list() / .get() / .all() / .search()
    client.companies.list() / .get() / .all() / .search()
    client.opportunities.list() / .get() / .all()

    # Lists
    client.lists.list() / .get() / .all()
    client.lists.resolve(name="Pipeline Name")
    client.lists.get_fields(ListId(123))

    # List entries
    entries_service = client.lists.entries(ListId(123))
    entries_service.list() / .get() / .all()
    entries_service.add_person() / .add_company() / .add_opportunity()
    entries_service.update_field_value() / .batch_update_fields()

    # Notes, reminders, interactions
    client.notes.list() / .create()
    client.reminders.list() / .create()
    client.interactions.list()

    # Rate limits
    snapshot = client.rate_limits.snapshot()

    # Identity
    me = client.whoami()
```

## Error Handling

```python
from affinity.exceptions import (
    AffinityError,           # Base class
    AuthenticationError,     # 401 - invalid/missing API key
    AuthorizationError,      # 403 - insufficient permissions
    NotFoundError,           # 404 - entity not found
    ValidationError,         # 400/422 - invalid parameters
    RateLimitError,          # 429 - rate limited
    ServerError,             # 500/503 - server errors
    WriteNotAllowedError,    # Write attempted in read-only mode
    TooManyResultsError,     # .all() exceeded limit
)

try:
    person = client.persons.get(PersonId(123))
except NotFoundError:
    print("Person not found")
except RateLimitError as e:
    print(f"Rate limited. Retry after: {e.retry_after}")
except AffinityError as e:
    print(f"Error: {e}")
    if e.diagnostics:
        print(f"Request ID: {e.diagnostics.request_id}")
```

## Creating Records (requires explicit user approval)

```python
from affinity.models import NoteCreate, ReminderCreate
from affinity.types import NoteType, ReminderType
from datetime import datetime, timedelta

# Add entity to list
entries_service = client.lists.entries(ListId(123))
entry = entries_service.add_person(PersonId(456))
entry = entries_service.add_company(CompanyId(789))

# Create note
note = client.notes.create(NoteCreate(
    content="<p>Meeting notes</p>",
    type=NoteType.HTML,
    person_ids=[PersonId(123)],
))

# Create reminder
reminder = client.reminders.create(ReminderCreate(
    owner_id=UserId(me.user.id),
    type=ReminderType.ONE_TIME,
    content="Follow up",
    due_date=datetime.now() + timedelta(days=7),
    person_id=PersonId(123),
))

# Update field value on list entry
entries_service.update_field_value(
    ListEntryId(456),
    FieldId(789),
    "New Value"
)

# Batch update multiple fields
entries_service.batch_update_fields(
    ListEntryId(456),
    {FieldId(789): "Value1", FieldId(790): "Value2"}
)
```

## Field Selection

```python
from affinity.types import FieldType

# Request specific field types
client.companies.list(field_types=[FieldType.ENRICHED])
client.persons.get(PersonId(123), field_types=[FieldType.GLOBAL, FieldType.RELATIONSHIP_INTELLIGENCE])

# Check if fields were requested and access data
if company.fields.requested:
    for field_name, value in company.fields.data.items():
        print(f"{field_name}: {value}")

# Available: GLOBAL, LIST, ENRICHED, RELATIONSHIP_INTELLIGENCE
```

## Rate Limits

```python
# Check current status (from cached headers)
snapshot = client.rate_limits.snapshot()
print(f"Per-minute: {snapshot.api_key_per_minute.remaining}/{snapshot.api_key_per_minute.limit}")
print(f"Monthly: {snapshot.org_monthly.remaining}/{snapshot.org_monthly.limit}")

# Refresh from API
refreshed = client.rate_limits.refresh()
```

## Retry Behavior

- **GET/HEAD**: Automatic retries (3 by default) for rate limits and transient errors
- **POST/PUT/PATCH/DELETE**: No automatic retries (to avoid duplicates)

```python
# Configure retries
client = Affinity(api_key="key", max_retries=5)
```

## Documentation

- Full SDK docs: https://yaniv-golan.github.io/affinity-sdk/latest/
