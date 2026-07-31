
# 03 — TempData

---



## 🎯 One-Line Definition

> **`TempData` stores data that survives exactly one redirect — you set it before a `RedirectToAction()` and read it in the next request, after which it is automatically deleted.**

---

## 🔷 The Problem TempData Solves

```
The Redirect Problem:
─────────────────────────────────────────────────────────────
POST /Employee/Create  → saves employee → RedirectToAction("Index")
                                              ↓
                                    GET /Employee/Index

Between POST and GET:
  ❌ ViewBag  → gone (ViewBag dies when the response ends)
  ❌ ViewData → gone (same — dies at response end)
  ✅ TempData → SURVIVES the redirect — readable in the next request

Without TempData:
  User saves a record → redirect → Index page → no feedback
  User doesn't know if it worked!

With TempData:
  User saves → TempData["Success"] = "Saved!" → redirect →
  Index reads TempData["Success"] → shows green alert ✅
```

---

## 🔷 How TempData Works

```
REQUEST 1 (POST — Create action):
  TempData["Success"] = "Employee added!";
  return RedirectToAction("Index");
         ↓
  TempData stored in SESSION (server-side)
  Response sent — HTTP 302 redirect

REQUEST 2 (GET — Index action):
  Browser follows redirect → GET /Employee/Index
  TempData["Success"] read → "Employee added!"
  TempData["Success"] marked for deletion

END OF REQUEST 2:
  TempData["Success"] deleted from session automatically
  Next request → TempData["Success"] is null

Key: lives for ONE redirect — then gone.
```

---

## 🔷 Setting TempData — In the Controller

```csharp
// POST action — set TempData before redirect
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid) return View(emp);

    string result = _bal.Insert(emp);

    if (result == "success")
    {
        TempData["Success"] = "Employee added successfully!";
        return RedirectToAction(nameof(Index));   // ← redirect
    }

    ModelState.AddModelError("", result);
    return View(emp);
}

// Other common TempData messages:
TempData["Success"] = "Record updated.";
TempData["Error"]   = "Delete failed — employee has active projects.";
TempData["Warning"] = "Employee saved but email notification failed.";
TempData["Info"]    = "No changes were made.";
```

---

## 🔷 Reading TempData — In the View

```cshtml
@* In _Layout.cshtml or at the top of any view *@
@* Show different alert styles based on which key was set *@

@if (TempData["Success"] != null)
{
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        <i class="bi bi-check-circle-fill me-2"></i>
        @TempData["Success"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

@if (TempData["Error"] != null)
{
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        <i class="bi bi-x-circle-fill me-2"></i>
        @TempData["Error"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

@if (TempData["Warning"] != null)
{
    <div class="alert alert-warning alert-dismissible fade show" role="alert">
        @TempData["Warning"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}

@if (TempData["Info"] != null)
{
    <div class="alert alert-info alert-dismissible fade show" role="alert">
        @TempData["Info"]
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
}
```

---

## 🔷 Reading TempData in the Controller

```csharp
public IActionResult Index()
{
    // TempData can also be read in the Controller:
    if (TempData["Success"] != null)
    {
        string msg = TempData["Success"].ToString();
        Console.WriteLine($"Flash message: {msg}");
    }

    var employees = _bal.GetAll();
    return View(employees);
}
```

---

## 🔷 `Keep()` — Preserve TempData for One More Request

```csharp
// Normally: TempData is deleted after being read.
// TempData.Keep() stops it from being deleted.

public IActionResult Index()
{
    // Read the value
    string msg = TempData["Success"]?.ToString();

    // Preserve it — don't delete after this request
    TempData.Keep("Success");
    // OR keep ALL TempData:
    TempData.Keep();

    return View(_bal.GetAll());
}

// After TempData.Keep():
//   This request — "Success" is readable ✅
//   Next request  — "Success" still there ✅
//   Request after — "Success" deleted ✅ (normal lifecycle resumes)
```

---

## 🔷 `Peek()` — Read Without Marking for Deletion

```csharp
// TempData.Peek("key") reads the value WITHOUT
// marking it for deletion — it survives to the next request.

public IActionResult Index()
{
    // Read value — does NOT mark for deletion
    string msg = TempData.Peek("Success")?.ToString();

    // "Success" still in TempData — readable in next request too
    return View(_bal.GetAll());
}

// TempData["key"]  → reads AND marks for deletion
// TempData.Peek("key") → reads WITHOUT marking for deletion
```

---

## 🔷 TempData with Complex Objects

```csharp
// TempData only serializes simple types natively.
// For objects, serialize to JSON:

using System.Text.Json;

// Set in Controller:
var employee = _bal.GetById(id);
TempData["LastViewedEmployee"] = JsonSerializer.Serialize(employee);
return RedirectToAction("Index");

// Read in next Controller action:
if (TempData["LastViewedEmployee"] != null)
{
    var emp = JsonSerializer.Deserialize<Employee>(
        TempData["LastViewedEmployee"].ToString()
    );
    Console.WriteLine($"Last viewed: {emp.Name}");
}
```

---

## 🔷 TempData Storage — How It Works Internally

```
TempData uses Session storage by default.
Session = server-side key/value store per user.

Request 1 (POST Create):
  TempData["Success"] = "Saved!"
  → stored in Session["__ControllerTempData"]
  → HTTP 302 Redirect response sent

Between requests:
  Data sits in server session

Request 2 (GET Index):
  TempData reads Session["__ControllerTempData"]
  → "Saved!" available
  → session entry marked for deletion

End of Request 2:
  Session entry deleted
  Gone forever

Requirement: Session must be enabled in Program.cs
```

```csharp
// Program.cs — enable session for TempData:
builder.Services.AddSession();
// ...
app.UseSession();

// Without this, TempData defaults to CookieTempDataProvider
// (stores data in a cookie instead of session — limited to ~4KB)
```

---

## 🔷 ViewData vs ViewBag vs TempData — The Full Comparison

```
┌────────────────────────────────────────────────────────────────┐
│  VIEWDATA           VIEWBAG             TEMPDATA               │
│  ─────────────────  ───────────────     ─────────────────────  │
│  Dictionary<string, dynamic wrapper    Survives redirect       │
│  object>            over ViewData       (one extra request)    │
│                                                                 │
│  ViewData["Name"]   ViewBag.Name        TempData["Msg"]        │
│  = "Alice"          = "Alice"           = "Saved!"             │
│                                                                 │
│  Needs cast         No cast needed      Needs null check        │
│  when reading       when reading        and cast if complex     │
│                                                                 │
│  Dies at end        Dies at end         Survives ONE redirect   │
│  of request         of request          then deleted           │
│                                                                 │
│  Best for:          Best for:           Best for:              │
│  Pass title,        Quick debug,        Post-Redirect-Get      │
│  metadata to        convenience         success/error msgs     │
│  layout             pass-through        flash notifications    │
└────────────────────────────────────────────────────────────────┘
```

|                   | ViewData                       | ViewBag                  | TempData                |
| ----------------- | ------------------------------ | ------------------------ | ----------------------- |
| Type              | `Dictionary<string, object>` | `dynamic`over ViewData | `ITempDataDictionary` |
| Syntax            | `ViewData["key"]`            | `ViewBag.key`          | `TempData["key"]`     |
| Scope             | Current request only           | Current request only     | Current + next request  |
| Survives redirect | ❌                             | ❌                       | ✅                      |
| Needs null check  | ✅                             | ✅                       | ✅                      |
| Needs cast        | ✅                             | ❌                       | ✅                      |
| Best use          | Page title, layout data        | Quick extras             | Flash messages          |

---

## 🔷 Complete Controller Pattern — CRUD with TempData

```csharp
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    public IActionResult Index()
        => View(_bal.GetAll());

    [HttpPost, ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);
        string r = _bal.Insert(emp);
        TempData[r == "success" ? "Success" : "Error"] =
            r == "success" ? "Employee added!" : r;
        return r == "success"
            ? RedirectToAction(nameof(Index))
            : View(emp);
    }

    [HttpPost, ValidateAntiForgeryToken]
    public IActionResult Edit(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);
        string r = _bal.Update(emp);
        TempData[r == "success" ? "Success" : "Error"] =
            r == "success" ? "Employee updated!" : r;
        return r == "success"
            ? RedirectToAction(nameof(Index))
            : View(emp);
    }

    [HttpPost, ValidateAntiForgeryToken]
    public IActionResult Delete(int id)
    {
        string r = _bal.Delete(id);
        TempData[r == "success" ? "Success" : "Error"] =
            r == "success" ? "Employee deleted." : r;
        return RedirectToAction(nameof(Index));
    }
}
```

---

## ⚠️ Common TempData Mistakes

| Mistake                                | What Happens                                  | Fix                                               |
| -------------------------------------- | --------------------------------------------- | ------------------------------------------------- |
| Reading TempData without null check    | `NullReferenceException`                    | Always `if (TempData["key"] != null)`           |
| Setting TempData WITHOUT redirecting   | TempData dies with the response — never read | TempData is for redirect scenarios only           |
| Storing large objects in TempData      | Session overflow / cookie too large           | Store minimal data — just a message string       |
| Setting TempData in View               | Not allowed — Views cannot set TempData      | Only set in Controller                            |
| Expecting TempData to last 2 redirects | Deleted after first read                      | Use `TempData.Keep()`to extend one more request |

---

## ⭐ Interview Quick-Fire

| Question                                     | Answer                                                                     |
| -------------------------------------------- | -------------------------------------------------------------------------- |
| What is TempData?                            | Stores data that survives exactly one redirect — deleted after being read |
| When would you use TempData?                 | Flash messages after POST-Redirect-GET — success/error notifications      |
| What happens to TempData after it's read?    | Marked for deletion — gone at end of that request                         |
| How to keep TempData for one more request?   | `TempData.Keep("key")`or `TempData.Keep()`for all                      |
| What is `TempData.Peek()`?                 | Reads without marking for deletion — survives to next request             |
| Where is TempData stored internally?         | Session (server-side) by default                                           |
| Can TempData store complex objects directly? | Not reliably — serialize to JSON first                                    |
| Difference: TempData vs ViewBag?             | ViewBag dies at end of current request. TempData survives one redirect.    |
