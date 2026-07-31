
# 03 — IActionResult vs ActionResult`<T>`

---

## 🎯 One-Line Definition

> **`IActionResult` is the flexible return type for any response — `ActionResult<T>` is the typed version that tells the compiler what data you're returning, making it self-documenting and better for Web APIs.**

---

## 🔷 The Three Return Type Options

```
OPTION 1: IActionResult
  → Any response — View, Json, Redirect, NotFound, Ok
  → No type info — compiler doesn't know what data is inside
  → Best for: MVC actions returning Views

OPTION 2: ActionResult<T>
  → Any response — but also knows the data type T
  → Compiler knows: "this returns an Employee on success"
  → Best for: Web API actions returning typed data

OPTION 3: Specific result types (ViewResult, JsonResult etc.)
  → Only that specific response — very restrictive
  → Rarely used — loses flexibility
  → Example: ViewResult means ONLY a View, nothing else
```

---

## 🔷 `IActionResult` — The Flexible Interface

```csharp
// IActionResult can return ANYTHING:
public IActionResult Index()
{
    var emp = _bal.GetById(1);

    if (emp == null) return NotFound();       // ← 404 response
    if (!User.Identity.IsAuthenticated)
        return Unauthorized();                 // ← 401 response

    return View(emp);                          // ← HTML view
    // OR
    return Json(emp);                          // ← JSON response
    // OR
    return RedirectToAction("Create");         // ← redirect
    // OR
    return File(bytes, "application/pdf");     // ← file download

    // All above are valid IActionResult returns
}
```

---

## 🔷 `ActionResult<T>` — The Typed Version

```csharp
// ActionResult<Employee> tells the compiler:
// "On success, this returns an Employee object"

public ActionResult<Employee> GetEmployee(int id)
{
    var emp = _bal.GetById(id);

    if (emp == null)
        return NotFound();   // ← still valid (IActionResult result)

    return emp;              // ← implicit conversion: Employee → ActionResult<Employee>
    //     ↑ no need to wrap in Ok(emp) — compiler handles it
}

// The double benefit:
// 1. Can still return status codes (NotFound, BadRequest etc.)
// 2. On the happy path, just return the object directly
```

---

## 🔷 IActionResult vs ActionResult`<T>` — Side by Side

```csharp
// SAME action — two ways to write it:

// Way 1: IActionResult (no type info)
public IActionResult GetEmployee(int id)
{
    var emp = _bal.GetById(id);
    if (emp == null) return NotFound();
    return Ok(emp);   // must wrap in Ok()
}

// Way 2: ActionResult<Employee> (typed)
public ActionResult<Employee> GetEmployee(int id)
{
    var emp = _bal.GetById(id);
    if (emp == null) return NotFound();
    return emp;        // direct return — no Ok() needed
}

// Both produce identical HTTP responses.
// ActionResult<T> is cleaner and self-documenting.
```

---

## 🔷 The Implicit Conversion — Magic of ActionResult`<T>`

```csharp
public ActionResult<Employee> GetEmployee(int id)
{
    // These are ALL valid returns — ActionResult<T> accepts both:

    // Return the object directly (implicit conversion to ActionResult<T>)
    Employee emp = _bal.GetById(id);
    return emp;                          // ✅ Employee → ActionResult<Employee>

    // Return IActionResult responses (also implicit)
    return NotFound();                   // ✅ IActionResult → ActionResult<Employee>
    return BadRequest("Invalid id");     // ✅
    return Ok(emp);                      // ✅ (same as returning emp directly)
    return CreatedAtAction(..., emp);    // ✅
}

// The magic: ActionResult<T> implicitly converts from BOTH T and IActionResult
```

---

## 🔷 When to Use Which

```
USE IActionResult when:
  ✅ MVC actions that return Views
  ✅ Actions that might return different response types
  ✅ Actions returning both View() and Json() depending on logic
  ✅ Simple controller actions (Create, Edit, Delete)

USE ActionResult<T> when:
  ✅ Web API actions returning typed JSON data
  ✅ You want clear documentation of what the endpoint returns
  ✅ Using Swagger/OpenAPI — it reads the type from ActionResult<T>
  ✅ Actions where the happy path returns one specific type

USE specific types (ViewResult etc.) ALMOST NEVER:
  ❌ Too restrictive — can't return NotFound() from a ViewResult
  ❌ Only use if you're 100% sure the action always returns that type
```

---

## 🔷 Real-World Usage Patterns

### MVC Controller — Use `IActionResult`

```csharp
public class EmployeeController : Controller
{
    // MVC actions — Views, Redirects, not typed data
    // IActionResult is the standard choice here

    public IActionResult Index()
        => View(_bal.GetAll());

    public IActionResult Details(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    [HttpPost]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);
        _bal.Insert(emp);
        return RedirectToAction(nameof(Index));
    }
}
```

### Web API Controller — Use `ActionResult<T>`

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeApiController : ControllerBase
{
    // API actions — typed JSON responses
    // ActionResult<T> is the standard choice here

    [HttpGet]
    public ActionResult<List<Employee>> GetAll()
        => _bal.GetAll();                // direct return

    [HttpGet("{id}")]
    public ActionResult<Employee> GetById(int id)
    {
        var emp = _bal.GetById(id);
        if (emp == null) return NotFound();
        return emp;                      // direct return
    }

    [HttpPost]
    public ActionResult<Employee> Create(Employee emp)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);

        int newId = _bal.InsertGetId(emp);
        emp.Id = newId;

        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
        //     ↑ 201 Created with Location header
    }
}
```

---

## 🔷 Async Versions

```csharp
// Async IActionResult
public async Task<IActionResult> Index()
{
    var employees = await _bal.GetAllAsync();
    return View(employees);
}

// Async ActionResult<T>
public async Task<ActionResult<Employee>> GetById(int id)
{
    var emp = await _bal.GetByIdAsync(id);
    if (emp == null) return NotFound();
    return emp;
}
```

---

## 🔷 Comparison Table

|                      | `IActionResult`        | `ActionResult<T>` | `ViewResult` |
| -------------------- | ------------------------ | ------------------- | -------------- |
| Return type          | Any action result        | Any + typed data    | Views only     |
| Direct object return | ❌ Must wrap in `Ok()` | ✅ Direct return    | ❌             |
| Status codes         | ✅                       | ✅                  | ❌             |
| Swagger type info    | ❌ Unknown               | ✅ Shows type       | ❌             |
| Best for             | MVC Views                | Web API endpoints   | Almost never   |
| Flexibility          | High                     | High + typed        | Low            |

---

## ⚠️ Common Mistakes

| Mistake                                      | What Happens                                      | Fix                                           |
| -------------------------------------------- | ------------------------------------------------- | --------------------------------------------- |
| Using `ActionResult<T>`in MVC View actions | Not wrong, but unusual — Views don't need typing | Use `IActionResult`for MVC actions          |
| Returning `null`from `ActionResult<T>`   | Serializes as JSON null — may confuse caller     | Return `NotFound()`instead of null          |
| Using `ViewResult`return type              | Can't return `NotFound()`if record missing      | Use `IActionResult`                         |
| Forgetting `Task<>`wrapper on async        | Compiler error — async method must return Task   | `public async Task<IActionResult> Action()` |

---

## ⭐ Interview Quick-Fire

| Question                                                    | Answer                                                                                                              |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| What is `IActionResult`?                                  | Interface that any action result implements — flexible, can return View/Json/Redirect/NotFound etc.                |
| What is `ActionResult<T>`?                                | Return type that can be either IActionResult (status codes) or T directly — used in Web APIs                       |
| Why use `ActionResult<T>`over `IActionResult`?          | Type safety — caller knows what data to expect; Swagger reads the type; direct return without wrapping in `Ok()` |
| Can `ActionResult<T>`return `NotFound()`?               | ✅ Yes — it accepts any IActionResult result                                                                       |
| What does implicit conversion mean for `ActionResult<T>`? | You can `return emp`directly — compiler converts `Employee`to `ActionResult<Employee>`automatically          |
| When is `IActionResult`preferred?                         | MVC actions returning Views, Redirects — any action where the return type varies                                   |
| When is `ActionResult<T>`preferred?                       | Web API endpoints where the happy path returns a specific typed response                                            |
