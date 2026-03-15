
# 02 — Action Methods

---

## 🎯 One-Line Definition

> **An Action Method is a public method inside a Controller that handles one specific HTTP request — it receives parameters, does work, and returns an `IActionResult` telling ASP.NET Core what to send back to the browser.**

---

## 🔷 What Makes a Method an Action

```
For a method to be an Action it must be:
  ✅ Public
  ✅ Inside a Controller class
  ✅ Returns IActionResult (or Task<IActionResult> for async)
  ✅ Not marked with [NonAction]

These are NOT actions:
  ❌ Private methods
  ❌ Methods returning void with no return type
  ❌ Methods marked with [NonAction]
  ❌ Static methods
```

---

## 🔷 Action Method Anatomy

```csharp
//  HTTP verb  Action name   Parameters
//     ↓           ↓             ↓
[HttpPost]
public IActionResult Create(Employee emp)
//     ↑                    ↑
// return type          model bound from form/body
{
    // do work
    return RedirectToAction(nameof(Index));  // ← return type
}
```

---

## 🔷 HTTP Verb Attributes — Controlling Which Method Handles What

```csharp
// No attribute = responds to ALL HTTP methods (usually GET)
public IActionResult Index() { }

// GET only — read data, show pages
[HttpGet]
public IActionResult Create() { }   // show empty form

// POST only — submit form, create data
[HttpPost]
public IActionResult Create(Employee emp) { }  // save form

// PUT only — full update (common in REST APIs)
[HttpPut]
public IActionResult Update(int id, Employee emp) { }

// PATCH only — partial update
[HttpPatch]
public IActionResult PartialUpdate(int id) { }

// DELETE only — delete data
[HttpDelete]
public IActionResult Delete(int id) { }
```

---

## 🔷 Same Name, Different Verb — GET + POST Pair

The most common pattern in MVC — same action name, one for GET (show form) and one for POST (save form):

```csharp
// GET /Employee/Create  → show empty form
[HttpGet]
public IActionResult Create()
{
    return View(new Employee());
}

// POST /Employee/Create  → receive and save form data
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
        return View(emp);           // ← go back to form with errors

    string result = _bal.Insert(emp);

    if (result == "success")
    {
        TempData["Success"] = "Employee added!";
        return RedirectToAction(nameof(Index));
    }

    ModelState.AddModelError("", result);
    return View(emp);
}
```

---

## 🔷 Action Parameters — How They Get Their Values

```csharp
// ── From route: /Employee/Details/5 ──────────────────────────────
public IActionResult Details(int id)
//                           ↑
//            matched from {id} in route pattern

// ── From query string: /Employee/Search?name=alice&dept=IT ───────
public IActionResult Search(string name, string dept)
//                          ↑ matched from ?name=    ?dept=

// ── From form body (POST) ─────────────────────────────────────────
[HttpPost]
public IActionResult Create(Employee emp)
//                          ↑ all form fields mapped to Employee

// ── From URL + query string + form combined ───────────────────────
[HttpPost]
public IActionResult Edit(int id, Employee emp, string returnUrl)
//                        ↑ route  ↑ form body  ↑ query string
```

---

## 🔷 Action Attributes — Controlling Behaviour

```csharp
// Restrict to POST only
[HttpPost]

// Prevent CSRF attacks — validates form token
[ValidateAntiForgeryToken]

// Require user to be authenticated
[Authorize]

// Require specific role
[Authorize(Roles = "Admin")]

// Exclude method from routing — treat as private helper
[NonAction]

// Cache the response
[ResponseCache(Duration = 60)]

// Override route for this action only
[Route("employees/all")]

// Accept multiple HTTP verbs
[AcceptVerbs("GET", "POST")]
```

---

## 🔷 Complete Action Method Patterns

### Pattern 1 — GET: Return View with data

```csharp
// GET /Employee
public IActionResult Index()
{
    var employees = _bal.GetAll();
    return View(employees);
}
```

### Pattern 2 — GET with ID: Null-safe single record

```csharp
// GET /Employee/Details/5
public IActionResult Details(int id)
{
    if (id <= 0) return BadRequest();

    var emp = _bal.GetById(id);
    if (emp == null) return NotFound();

    return View(emp);
}
```

### Pattern 3 — POST: Form submission with validation

```csharp
// POST /Employee/Create
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid)
        return View(emp);

    string result = _bal.Insert(emp);

    if (result == "success")
        return RedirectToAction(nameof(Index));

    ModelState.AddModelError("", result);
    return View(emp);
}
```

### Pattern 4 — POST: Delete (no confirmation page)

```csharp
// POST /Employee/Delete/5
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Delete(int id)
{
    _bal.Delete(id);
    TempData["Success"] = "Employee deleted.";
    return RedirectToAction(nameof(Index));
}
```

### Pattern 5 — JSON response for AJAX

```csharp
// POST /Employee/SaveJson
[HttpPost]
public IActionResult SaveJson([FromBody] Employee emp)
{
    if (!ModelState.IsValid)
        return BadRequest(new { success = false, errors = ModelState });

    string result = _bal.Insert(emp);

    return Json(new {
        success = result == "success",
        message = result
    });
}
```

### Pattern 6 — Async action

```csharp
// GET /Employee
public async Task<IActionResult> Index()
{
    var employees = await _bal.GetAllAsync();
    return View(employees);
}
```

---

## 🔷 `nameof()` — Use It for Action Names

```csharp
// ❌ Magic string — typo-prone
return RedirectToAction("Index");
return RedirectToAction("Edit");

// ✅ nameof() — compile-time safe, refactor-friendly
return RedirectToAction(nameof(Index));
return RedirectToAction(nameof(Edit));

// If you rename the action, nameof() catches it at compile time.
// With string literals, it silently breaks at runtime.
```

---

## 🔷 Passing Data from Action to View

```csharp
public IActionResult Index()
{
    var employees = _bal.GetAll();

    // Method 1: Strongly typed — best, use this
    return View(employees);

    // Method 2: ViewData — dictionary, loosely typed
    ViewData["Title"] = "All Employees";
    ViewData["Count"] = employees.Count;
    return View();

    // Method 3: ViewBag — dynamic, loosely typed
    ViewBag.Title = "All Employees";
    ViewBag.Count = employees.Count;
    return View();

    // Method 4: TempData — survives redirect
    TempData["Success"] = "Saved!";
    return RedirectToAction(nameof(Index));
}
```

---

## 🔷 Action Method — Return Type Options

```csharp
// All valid return types from actions:
public IActionResult        Action1() { return View(); }
public ActionResult         Action2() { return View(); }
public ActionResult<Employee> Action3() { return emp; }
public ViewResult           Action4() { return View(); }
public JsonResult           Action5() { return Json(data); }
public string               Action6() { return "hello"; }   // raw string

// Async versions:
public async Task<IActionResult>        Action7() { ... }
public async Task<ActionResult<Employee>> Action8() { ... }
```

---

## 🔷 [NonAction] — Hide a Method From Routing

```csharp
public class EmployeeController : Controller
{
    public IActionResult Index() { return View(); }    // ← action

    [NonAction]
    public string FormatName(string name)              // ← helper method
    {
        return name?.Trim().ToTitleCase();
        // NOT an action — cannot be called via URL
        // Used internally by other actions
    }
}
```

---

## ⚠️ Common Action Method Mistakes

| Mistake                                     | What Happens                                                 | Fix                                                  |
| ------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| Forgetting `[HttpPost]`on POST action     | Both GET and POST match the same action — ambiguous routing | Always add `[HttpPost]`on form submission handlers |
| Not checking `ModelState.IsValid`         | Invalid data reaches BAL/DAL                                 | Always check before processing                       |
| Using magic strings in `RedirectToAction` | Runtime error if action renamed                              | Use `nameof(ActionName)`                           |
| Returning `View()`after redirect          | Double response                                              | Return redirect OR view — never both                |
| Private action methods                      | Not reachable via HTTP                                       | Make actions `public`                              |

---

## ⭐ Interview Quick-Fire

| Question                                          | Answer                                                                                 |
| ------------------------------------------------- | -------------------------------------------------------------------------------------- |
| What is an Action Method?                         | A public method in a Controller that handles an HTTP request and returns IActionResult |
| What does `[HttpPost]`do on an action?          | Restricts that action to handle only POST requests                                     |
| What does `[ValidateAntiForgeryToken]`do?       | Validates the hidden anti-forgery token — prevents CSRF attacks                       |
| What does `[NonAction]`do?                      | Marks a public method as NOT an action — routing ignores it                           |
| Why use `nameof(Index)`instead of `"Index"`?  | Compile-time safety — if action is renamed, the compiler catches the break            |
| What happens when `ModelState.IsValid`is false? | Return the same View with the model — validation errors shown to user                 |
| Can two actions have the same name?               | ✅ Yes — if they handle different HTTP verbs (`[HttpGet]`+`[HttpPost]`)           |
