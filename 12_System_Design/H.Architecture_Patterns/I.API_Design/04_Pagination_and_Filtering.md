# 04_Pagination_and_Filtering

> **Pagination** = returning a large collection in small, manageable chunks ("pages") instead of all at once. **Filtering** (and sorting) = letting the client narrow down and order a collection using query parameters, exactly as `01_REST_API_Principles.md` recommended ("use query parameters for filtering/sorting, not new endpoints").

## 📌 What is it?

`GET /products` against a table with 2 million rows should **never** return all 2 million rows in one response. Pagination breaks that collection into pages; Filtering/Sorting lets the client ask for exactly the slice they want, in the order they want it.

```
GET /products?page=2&pageSize=20&category=electronics&sort=price_asc

              │           │              │                │
          pagination  pagination      filter            sort
```

## 🤔 Why do we need it?

| Problem without pagination/filtering                                  | How it helps                                                       |
| --------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Returning 2 million rows crashes the client or takes forever          | Return only 20-50 rows per request — fast, manageable             |
| A Kendo Grid trying to render 2 million rows would freeze the browser | Server sends exactly what fits on one page of the grid             |
| Client wants only "electronics" products but server sends everything  | Filtering lets the DB/query do the narrowing, not the client       |
| Network bandwidth wasted sending data nobody will look at             | Smaller, targeted responses save bandwidth and speed up load times |

## 🌍 Real-world analogy

A **search engine's results page**. Google doesn't show you all 4 billion matching pages at once — it shows you 10 at a time (pagination), lets you narrow by date/type (filtering), and lets you sort by relevance or date (sorting). You'd never want — or be able to use — all the results in one giant unsorted dump.

## 📊 Pagination Strategies

| Strategy                        | How it works                                                                             | Pros                                                              | Cons                                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Offset-based**          | `?page=2&pageSize=20` → `OFFSET 20 ROWS FETCH NEXT 20`                              | Simple, supports "jump to page N"                                 | Gets slower on large offsets (DB must scan/skip past all prior rows); can show duplicates/gaps if data changes between page loads |
| **Cursor-based (keyset)** | `?after=lastSeenId&pageSize=20` → `WHERE Id > lastSeenId ORDER BY Id FETCH NEXT 20` | Fast even on huge tables, stable under concurrent inserts/deletes | Can't jump directly to an arbitrary page number — only "next"/"previous"                                                         |

```
Offset-based:                              Cursor-based:

Page 1: rows 1-20    (OFFSET 0)            First 20: WHERE Id > 0 ORDER BY Id
Page 2: rows 21-40   (OFFSET 20)           Next 20:  WHERE Id > 20 ORDER BY Id
Page 1000: rows 19981-20000                          (uses the LAST seen Id as
   (OFFSET 19980 — DB must skip                       the starting point — no
    ~20,000 rows first — SLOW)                        expensive OFFSET skipping)
```

## 🖼 Why large OFFSET is slow (the classic pagination performance bug)

```
SELECT * FROM Products ORDER BY Id OFFSET 100000 ROWS FETCH NEXT 20 ROWS ONLY;

The database engine typically still has to:
  1. Sort/scan through the first 100,000 rows
  2. THROW THEM AWAY
  3. Return only the next 20

→ Deep pages get progressively SLOWER as the offset grows — a real, measurable
  production issue on "infinite scroll" or "jump to last page" features.
```

## 💻 Code examples

### Basic — offset-based pagination in a stored procedure

```sql
CREATE PROCEDURE sp_GetProductsPaged
    @PageNumber INT,
    @PageSize INT
AS
BEGIN
    SELECT ProductId, Name, Price
    FROM Products
    ORDER BY ProductId
    OFFSET (@PageNumber - 1) * @PageSize ROWS
    FETCH NEXT @PageSize ROWS ONLY;

    -- Also return the total count, so the client can render page numbers
    SELECT COUNT(*) AS TotalCount FROM Products;
END
```

```csharp
[HttpGet]
public IActionResult GetProducts(int page = 1, int pageSize = 20)
{
    var (products, totalCount) = _dal.GetProductsPaged(page, pageSize);

    return Ok(new
    {
        data = products,
        pagination = new
        {
            page,
            pageSize,
            totalCount,
            totalPages = (int)Math.Ceiling(totalCount / (double)pageSize)
        }
    });
}
```

### Intermediate — cursor-based pagination (better for large/frequently-changing tables)

```sql
CREATE PROCEDURE sp_GetProductsAfter
    @LastSeenId INT,
    @PageSize INT
AS
BEGIN
    SELECT TOP (@PageSize) ProductId, Name, Price
    FROM Products
    WHERE ProductId > @LastSeenId   -- no OFFSET needed — index seek, not a scan-and-skip
    ORDER BY ProductId;
END
```

```csharp
[HttpGet]
public IActionResult GetProductsAfter(int lastSeenId = 0, int pageSize = 20)
{
    var products = _dal.GetProductsAfter(lastSeenId, pageSize);

    return Ok(new
    {
        data = products,
        nextCursor = products.Any() ? products.Last().ProductId : (int?)null
    });
}
```

### Practical — filtering AND sorting together, safely (avoiding SQL injection)

```csharp
[HttpGet]
public IActionResult GetProducts(
    string? category,
    decimal? minPrice,
    decimal? maxPrice,
    string sort = "name_asc",
    int page = 1,
    int pageSize = 20)
{
    // Pass filters as PARAMETERS to the stored procedure — never concatenate into SQL text
    var result = _dal.GetFilteredProducts(category, minPrice, maxPrice, sort, page, pageSize);
    return Ok(result);
}
```

```sql
CREATE PROCEDURE sp_GetFilteredProducts
    @Category NVARCHAR(100) = NULL,
    @MinPrice DECIMAL(10,2) = NULL,
    @MaxPrice DECIMAL(10,2) = NULL,
    @SortColumn NVARCHAR(50) = 'Name',
    @SortDirection NVARCHAR(4) = 'ASC',
    @PageNumber INT,
    @PageSize INT
AS
BEGIN
    SELECT ProductId, Name, Price, Category
    FROM Products
    WHERE (@Category IS NULL OR Category = @Category)
      AND (@MinPrice IS NULL OR Price >= @MinPrice)
      AND (@MaxPrice IS NULL OR Price <= @MaxPrice)
    ORDER BY
        CASE WHEN @SortColumn = 'Price' AND @SortDirection = 'ASC'  THEN Price END ASC,
        CASE WHEN @SortColumn = 'Price' AND @SortDirection = 'DESC' THEN Price END DESC,
        CASE WHEN @SortColumn = 'Name'  AND @SortDirection = 'ASC'  THEN Name  END ASC
    OFFSET (@PageNumber - 1) * @PageSize ROWS
    FETCH NEXT @PageSize ROWS ONLY;
END
```

> ⚠️ Note: **never** build the `ORDER BY` clause via raw string concatenation with user input (e.g., `"ORDER BY " + sortColumn`) — that's a direct SQL injection risk. The `CASE WHEN` pattern above (or a strict whitelist-checked column name) keeps sorting both flexible and safe.

## ⚡ Performance considerations

- Always **index the columns used for filtering and sorting** (e.g., an index on `Category`, `Price`) — without it, every filtered query becomes a full table scan.
- Prefer **cursor-based pagination** for large or frequently-changing tables, or for "infinite scroll" UIs — offset-based pagination degrades as page depth increases.
- Always return **total count** (or at least "has more pages") so the client (e.g., a Kendo Grid) can render pagination controls correctly — but be aware `COUNT(*)` on a huge table has its own cost; cache it if it's expensive and doesn't need to be perfectly real-time.

## 🚨 Common mistakes

- ❌ Returning an entire table with no pagination "because the table is small right now" — tables grow, and this becomes a production incident later.
- ❌ Building `ORDER BY`/filter clauses via raw string concatenation of user input — a classic SQL injection vector.
- ❌ Using offset-based pagination for a real-time, frequently-updated feed — items can shift between pages, causing duplicates or skipped rows as users page through.
- ❌ Not setting a maximum `pageSize` — a client requesting `pageSize=1000000` can still overload the server if there's no server-side cap.

## 💡 Best practices

- ✅ Always paginate any endpoint that could return more than a screen's worth of data — don't wait until it becomes a performance problem.
- ✅ Cap `pageSize` server-side (e.g., max 100) regardless of what the client requests.
- ✅ Use cursor-based pagination for large, frequently-changing, or infinite-scroll-style collections; offset-based is fine for smaller, stable datasets with "jump to page N" UIs (like a typical Kendo Grid).
- ✅ Index every column used in `WHERE` (filtering) and `ORDER BY` (sorting) clauses.
- ✅ Whitelist sortable columns explicitly (as in the `CASE WHEN` example) — never trust a raw column name from the client.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                  | Answer                                                                                                                                      |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Why is pagination necessary for collection endpoints?                     | Prevents returning huge datasets in one response — protects performance, bandwidth, and client rendering (e.g., a grid)                    |
| What's the main downside of offset-based pagination?                      | It gets progressively slower on deep pages, since the database must scan and discard all preceding rows before returning the requested page |
| What's the advantage of cursor-based pagination?                          | Consistently fast regardless of depth, and stable even as rows are inserted/deleted concurrently — no OFFSET-skipping needed               |
| Where should filtering/sorting parameters go in a RESTful API?            | As query parameters (e.g.,`?category=electronics&sort=price_asc`), per REST principles — not as separate endpoints                       |
| Why is dynamically building an ORDER BY clause from user input dangerous? | Raw string concatenation of user input into SQL is a SQL injection risk — use parameterization or a whitelist/CASE WHEN pattern instead    |

## 📝 30-second Revision Cheat Sheet

- Pagination = return data in small pages, never the whole collection at once.
- Offset-based (`page`, `pageSize`) = simple, supports jump-to-page, but slows down on deep pages.
- Cursor-based (`after=lastId`) = fast at any depth, stable under concurrent changes, no jump-to-page.
- Filtering/sorting go in query parameters, per REST principles — never bake them into new endpoints.
- Index filtered/sorted columns; cap `pageSize` server-side; NEVER string-concatenate user input into SQL/ORDER BY.
