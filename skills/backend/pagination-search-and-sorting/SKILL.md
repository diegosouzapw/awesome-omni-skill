---
name: Pagination, Search, and Sorting
description: Implement server-side pagination, search filtering, and column sorting across API endpoints and frontend list pages.
globs:
  - "RestReactAspire.Server/Stores/**"
  - "RestReactAspire.Server/Endpoints/**"
  - "RestReactAspire.Server/Models/Link.cs"
  - "frontend/src/pages/*ListPage.tsx"
---

# Pagination, Search, and Sorting

## Backend

### Query Parameters
All list endpoints accept:
- `page` (int, default: 1) — current page number
- `pageSize` (int, default: 10) — items per page
- `search` (string?, optional) — text to filter results
- `sortBy` (string, default varies) — field name to sort by
- `sortDirection` (string, default: "asc") — `asc` or `desc`

### Default Sort Orders
- **Patients**: by `lastName` then `firstName`
- **Doctors**: by `specialty` then `lastName`
- **Exams**: by `scheduledDate`

### Store Implementation
```csharp
public (IReadOnlyList<T> Items, int TotalCount) GetPaged(int page, int pageSize, string sortBy, string sortDirection)
{
    var totalCount = _collection.Count();
    var items = ApplySort(_collection.FindAll(), sortBy, sortDirection)
        .Skip((page - 1) * pageSize).Take(pageSize).ToList();
    return (items, totalCount);
}

public (IReadOnlyList<T> Items, int TotalCount) SearchPaged(string search, int page, int pageSize, string sortBy, string sortDirection)
{
    var filtered = _collection.FindAll()
        .Where(item => /* case-insensitive string matching on relevant fields */)
        .ToList();
    var totalCount = filtered.Count;
    var items = ApplySort(filtered, sortBy, sortDirection)
        .Skip((page - 1) * pageSize).Take(pageSize).ToList();
    return (items, totalCount);
}
```

### Response Structure
```csharp
public record {Entity}ListResponse(
    IReadOnlyList<{Entity}Response> Items,
    PaginationInfo Pagination,
    SortInfo Sort,
    IReadOnlyList<Link> Links);
```

### Pagination Links
Use `PaginationLinks.Build(basePath, page, pageSize, totalPages, search, sortBy, sortDirection, additionalLinks)`.

## Frontend

### List Page Pattern
- State: `page`, `pageSize`, `search`, `sortBy`, `sortDirection`, `data`.
- Fetch with query params appended to the HATEOAS-discovered base URL.
- MUI `TablePagination` for page navigation.
- MUI `TextField` + `Button` for search input.
- MUI `TableSortLabel` on column headers for sort toggle.
- Clicking a column header cycles: default → asc → desc.
