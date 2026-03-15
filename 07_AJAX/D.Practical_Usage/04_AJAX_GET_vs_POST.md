
# 04 — AJAX GET vs POST

---

## 🎯 One-Line Definition

> **GET is for fetching data — nothing changes on the server. POST is for sending data — something gets created, updated, or deleted on the server.**

---

## 🔑 The Core Difference — One Rule to Remember

```
┌─────────────────────────────────────────────────────────┐
│  GET  →  "Give me something"   Server stays unchanged   │
│  POST →  "Here's something"    Server state changes     │
└─────────────────────────────────────────────────────────┘
```

---

## 🔑 Where Does the Data Go?

This is the biggest practical difference:

```
GET — data travels in the URL (query string)
─────────────────────────────────────────────────────────────
/Employee/Search?name=Alice&dept=IT&page=2&pageSize=10
                 ↑──────────────────────────────────────
                 everyone can see this in the address bar,
                 browser history, server logs

POST — data travels in the request body (hidden)
─────────────────────────────────────────────────────────────
URL:  /Employee/Create
Body: name=Alice&department=IT&salary=75000  ← not in the URL
      (or JSON: {"name":"Alice","department":"IT","salary":75000})
```

---

## 🔑 Side-by-Side Comparison

|                                  | GET                               | POST                       |
| -------------------------------- | --------------------------------- | -------------------------- |
| **Purpose**                | Read / fetch data                 | Send / change data         |
| **Data location**          | URL query string                  | Request body               |
| **Visible in address bar** | ✅ Yes                            | ❌ No                      |
| **Browser history**        | ✅ Saved                          | ❌ Not saved               |
| **Can be bookmarked**      | ✅ Yes                            | ❌ No                      |
| **Data size limit**        | ~2000 characters                  | No practical limit         |
| **Cached by browser**      | ✅ Yes (can be)                   | ❌ No                      |
| **Safe to repeat**         | ✅ Yes — idempotent              | ❌ No — creates duplicate |
| **Sends a body**           | ❌ No                             | ✅ Yes                     |
| **ASP.NET attribute**      | `[HttpGet]`                     | `[HttpPost]`             |
| **jQuery method**          | `$.get()`        | `$.post()` |                            |

---

## 💻 GET — Code and Controller

```javascript
// ── Simple GET — no data ──────────────────────────────────────
$.get("/Employee/GetAll", function(data) {
    renderTable(data);
});

// ── GET with parameters — sent as query string ────────────────
$.get("/Employee/Search", {
    name:       "Alice",
    department: "IT",
    page:       1,
    pageSize:   10
}, function(data) {
    renderTable(data);
});
// URL becomes: /Employee/Search?name=Alice&department=IT&page=1&pageSize=10

// ── GET with $.ajax ───────────────────────────────────────────
$.ajax({
    url:      "/Employee/GetById",
    type:     "GET",
    data:     { id: 5 },           // becomes ?id=5 in URL
    dataType: "json",
    success: function(emp) {
        populateForm(emp);
    }
});
```

```csharp
// Controller for GET — parameters come from query string
[HttpGet]
public JsonResult Search(string name, string department, int page = 1, int pageSize = 10)
// ↑ ASP.NET reads these directly from the URL query string
{
    var query = _db.Employees.AsQueryable();

    if (!string.IsNullOrEmpty(name))
        query = query.Where(e => e.Name.Contains(name));

    if (!string.IsNullOrEmpty(department))
        query = query.Where(e => e.Department == department);

    var result = query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToList();

    return Json(result);
}

[HttpGet]
public JsonResult GetById(int id)   // id comes from ?id=5
{
    var emp = _db.Employees.Find(id);
    return Json(emp);
}
```

---

## 💻 POST — Code and Controller

```javascript
// ── Simple POST — form-encoded ────────────────────────────────
$.post("/Employee/Create", {
    name:       "Alice",
    department: "IT",
    salary:     75000
}, function(response) {
    if (response.success) showNotification("Created!");
});

// ── POST with JSON body ───────────────────────────────────────
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify({
                     name:       "Alice",
                     department: "IT",
                     salary:     75000
                 }),
    success: function(response) {
        showNotification("Created! ID = " + response.id);
    }
});

// ── POST for delete (HTML forms can only do GET/POST) ─────────
$.post("/Employee/Delete", { id: 5 }, function(response) {
    if (response.success) removeRowFromTable(5);
});
```

```csharp
// Controller for POST — data comes from body
[HttpPost]
public JsonResult Create(Employee employee)
// ↑ model binding reads from POST body (form-encoded)
{
    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true, id = employee.Id });
}

[HttpPost]
public JsonResult Delete(int id)   // id comes from POST body
{
    var emp = _db.Employees.Find(id);
    if (emp != null) { _db.Employees.Remove(emp); _db.SaveChanges(); }
    return Json(new { success = true });
}
```

---

## 🔑 When to Use GET vs POST — Decision Guide

```
Use GET when:
  ✅ You are READING data — not changing anything
  ✅ The same request can safely be repeated many times
  ✅ Filtering / searching / loading a list
  ✅ Loading a single record to display
  ✅ Populating a dropdown with options
  ✅ You want the URL to be shareable/bookmarkable

Use POST when:
  ✅ You are WRITING data — creating, updating, deleting
  ✅ The data contains sensitive info (passwords, personal data)
  ✅ The data is large (long text, files, complex objects)
  ✅ You need to send a JSON body ([FromBody])
  ✅ The server state changes as a result
  ✅ ASP.NET requires [ValidateAntiForgeryToken]
     (only works on POST)
```

---

## 🔑 Why Kendo Grid Uses POST for Read

Kendo Grid's Read operation uses POST even though it's just fetching data. This surprises people:

```
Normal logic says: fetching data → use GET
Kendo's logic:     fetching data → use POST

Why?
──────────────────────────────────────────────────────────
When the Grid reads data it sends sorting, filtering,
and grouping information in the request body:

  sort[0][field]=Name
  sort[0][dir]=asc
  filter[logic]=and
  filter[filters][0][field]=Department
  filter[filters][0][operator]=eq
  filter[filters][0][value]=IT
  group[0][field]=Department
  page=1
  pageSize=10

This data is too large and complex for a URL query string.
So Kendo sends it as a POST body.
GET has URL length limits. POST does not.
```

---

## ⚠️ Common Mistakes

| Mistake                        | Symptom                                                     | Fix                         |
| ------------------------------ | ----------------------------------------------------------- | --------------------------- |
| Using GET to delete a record   | Works but insecure — delete can be triggered from any link | Use POST for all writes     |
| Sending sensitive data in GET  | Password/token visible in URL, browser history, server logs | Use POST for sensitive data |
| Large data in GET query string | 414 URI Too Long error                                      | Switch to POST with body    |
| Using POST for a simple read   | Works but unnecessary complexity                            | GET is fine for reads       |

---

## ❓ Interview Questions

**Q: What is the main difference between GET and POST?**

> GET is for reading data — parameters go in the URL query string, nothing on the server changes. POST is for sending data — parameters go in the request body, and the server state changes (create, update, delete).

**Q: Why should you never use GET for delete operations?**

> GET requests can be triggered by anything — a browser prefetch, a link in an email, a bot crawling your site. If delete uses GET, visiting `/Employee/Delete?id=5` in a browser would silently delete that employee. POST requires an intentional form submission or AJAX call.

**Q: Why does Kendo Grid use POST for its Read operation?**

> Because the Read request includes paging, sorting, filtering, and grouping parameters that are too large and complex for a URL query string. POST allows sending this data in the request body with no length limit.

**Q: When would you use GET with AJAX?**

> Loading a list, searching/filtering records, getting a single record to populate a form, populating a dropdown with options — any read operation where the server state doesn't change.
>
