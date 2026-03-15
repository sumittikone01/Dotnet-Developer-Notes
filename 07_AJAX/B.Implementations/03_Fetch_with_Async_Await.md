
# 03 — Fetch with Async/Await

---

## 🎯 One-Line Definition

> **Async/await is syntax that makes Fetch code look like normal top-to-bottom code — no `.then()` chains, just write `await` and the next line runs after the result arrives.**

---

## 🔄 The Same Request — Three Ways

Read all three. See how the code evolves from messy to clean:

```javascript
// ── WAY 1: XHR (oldest, most verbose) ────────────────────────
var xhr = new XMLHttpRequest();
xhr.open("GET", "/api/employees");
xhr.responseType = "json";
xhr.onload = function() { renderTable(xhr.response); };
xhr.send();


// ── WAY 2: Fetch + .then() chains ────────────────────────────
fetch("/api/employees")
    .then(r => r.json())
    .then(data => renderTable(data))
    .catch(err => console.error(err));


// ── WAY 3: Fetch + async/await ───────────────────────────────
async function loadEmployees() {
    const response = await fetch("/api/employees");
    const data     = await response.json();
    renderTable(data);
}
```

Way 3 reads like plain English: *"wait for the response, then get the data from it, then render."*

---

## 🔑 Two Keywords to Learn

```
┌─────────────────────────────────────────────────────────┐
│  async                                                   │
│  ─────                                                   │
│  Put before a function declaration.                     │
│  Means: "this function contains async operations."      │
│  Makes the function return a Promise automatically.     │
│                                                         │
│  async function loadData() { ... }                      │
│  const loadData = async () => { ... }                   │
├─────────────────────────────────────────────────────────┤
│  await                                                   │
│  ─────                                                   │
│  Put before any Promise.                                │
│  Means: "pause THIS function here until Promise done."  │
│  Gives you the resolved value directly.                 │
│                                                         │
│  const response = await fetch("/api/employees");        │
│  const data     = await response.json();                │
│                                                         │
│  ⚠️  await can ONLY be used inside an async function    │
└─────────────────────────────────────────────────────────┘
```

---

## 💻 Basic GET with Async/Await

```javascript
async function loadEmployees() {
    const response = await fetch("/api/employees");
    //                     ↑
    //    pauses here until fetch resolves
    //    response = the HTTP response object

    const data = await response.json();
    //             ↑
    //    pauses here until JSON parsing is done
    //    data = your actual array of employees

    console.log(data);       // [{id:1, name:"Alice"}, ...]
    renderTable(data);
}

// Call the function
loadEmployees();
```

---

## 💻 Proper Error Handling — try/catch

```javascript
async function loadEmployees() {
    try {
        const response = await fetch("/api/employees");

        // Check HTTP status — fetch doesn't throw for 400/500
        if (!response.ok) {
            throw new Error("Request failed: " + response.status);
        }

        const data = await response.json();
        renderTable(data);

    } catch (error) {
        // Catches:
        //   1. Network failures (no internet etc.)
        //   2. The throw above (400, 500 responses)
        //   3. JSON parse errors
        console.error("Error:", error.message);
        showErrorMessage(error.message);

    } finally {
        // ALWAYS runs — success or failure
        hideLoadingSpinner();
    }
}
```

---

## 💻 POST Request — Sending JSON

```javascript
async function createEmployee(employeeData) {
    try {
        const response = await fetch("/Employee/Create", {
            method:  "POST",
            headers: { "Content-Type": "application/json" },
            body:    JSON.stringify(employeeData)
        });

        if (!response.ok) {
            const errorBody = await response.json();
            throw new Error("Validation failed: " + JSON.stringify(errorBody));
        }

        const saved = await response.json();
        console.log("Created with ID:", saved.id);
        return saved;

    } catch (error) {
        console.error("Create failed:", error.message);
        throw error;   // re-throw so the caller knows it failed
    }
}

// Call it
createEmployee({ name: "Alice", department: "IT", salary: 75000 });
```

---

## 💻 Chained Calls — await Makes It Clean

Need to load an employee, then load their department, then load the manager:

```javascript
// ── With .then() chains — gets hard to follow ─────────────────
fetch("/api/employees/1")
    .then(r => r.json())
    .then(emp => fetch("/api/departments/" + emp.deptId))
    .then(r => r.json())
    .then(dept => fetch("/api/employees/" + dept.managerId))
    .then(r => r.json())
    .then(manager => console.log(manager));


// ── With async/await — reads top to bottom ────────────────────
async function loadManagerInfo() {
    const empRes    = await fetch("/api/employees/1");
    const employee  = await empRes.json();

    const deptRes   = await fetch("/api/departments/" + employee.deptId);
    const dept      = await deptRes.json();

    const mgrRes    = await fetch("/api/employees/" + dept.managerId);
    const manager   = await mgrRes.json();

    console.log("Manager is:", manager.name);
}
```

---

## 💻 Parallel Requests with async/await

When requests don't depend on each other — run them at the same time:

```javascript
async function loadDashboard() {
    // ❌ Sequential — slow (waits for each before starting next)
    const employees   = await fetch("/api/employees").then(r => r.json());
    const departments = await fetch("/api/departments").then(r => r.json());
    const stats       = await fetch("/api/stats").then(r => r.json());
    // Total time = time1 + time2 + time3

    // ✅ Parallel — fast (all fire at once)
    const [employees2, departments2, stats2] = await Promise.all([
        fetch("/api/employees").then(r => r.json()),
        fetch("/api/departments").then(r => r.json()),
        fetch("/api/stats").then(r => r.json())
    ]);
    // Total time = max(time1, time2, time3)
}
```

---

## 💻 Reusable Fetch Helper

Build this once, use it everywhere in your project:

```javascript
// helpers/api.js

async function apiGet(url) {
    const response = await fetch(url);
    if (!response.ok) throw new Error(response.status + " " + response.statusText);
    return response.json();
}

async function apiPost(url, data) {
    const response = await fetch(url, {
        method:  "POST",
        headers: { "Content-Type": "application/json" },
        body:    JSON.stringify(data)
    });
    if (!response.ok) throw new Error(response.status + " " + response.statusText);
    return response.json();
}

async function apiDelete(url) {
    const response = await fetch(url, { method: "DELETE" });
    if (!response.ok) throw new Error(response.status);
    return response.ok;
}


// Usage anywhere in your app — clean and simple:
async function loadPage() {
    try {
        const employees   = await apiGet("/api/employees");
        const departments = await apiGet("/api/departments");
        renderGrid(employees);
        populateDdl(departments);
    } catch (error) {
        showError(error.message);
    }
}
```

---

## 🔑 async/await vs .then() — When to Use Which

| Situation                                  | Better Choice                         |
| ------------------------------------------ | ------------------------------------- |
| Simple single request                      | Either — both fine                   |
| Multiple dependent requests (step by step) | `async/await`— reads linearly      |
| Multiple independent requests (parallel)   | `Promise.all`(same in both styles)  |
| Existing `.then()`codebase               | Stick with `.then()`for consistency |
| New code you're writing today              | `async/await`— cleaner             |
| Error handling with try/catch              | `async/await`— natural             |

---

## ⚠️ Common Mistakes

| Mistake                                     | What Happens                                            | Fix                                                |
| ------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| Using `await`outside `async`function    | `SyntaxError: await is only valid in async functions` | Always wrap in `async function`                  |
| Forgetting `await`before `fetch()`      | `response`is a Promise object, not the response       | Add `await`                                      |
| Forgetting `await`before `.json()`      | `data`is a Promise, not your array                    | Add `await`before `.json()`                    |
| Not checking `response.ok`                | 400/500 silently continue to next line                  | Check `if (!response.ok) throw new Error(...)`   |
| Sequential `await`when parallel is faster | Page loads slowly                                       | Use `Promise.all([...])`for independent requests |
| No try/catch                                | Unhandled Promise rejection error in console            | Always wrap async calls in try/catch               |

---

## 📊 All Three Approaches — Final Comparison

```
                XHR              .then()           async/await
                ───────────────  ───────────────   ─────────────────
Readability     Hard             Medium            Easy
Error handling  Manual           .catch()          try/catch
Chaining        Nested callbacks Flat chain        Top-to-bottom
Parallel        Manual           Promise.all       Promise.all
Use in          Legacy code      All modern JS     All modern JS ✅
```

---

## ❓ Interview Questions

**Q: What does `async` do to a function?**

> It marks the function as asynchronous, allowing `await` to be used inside it. The function automatically returns a Promise, even if you return a plain value.

**Q: What does `await` do?**

> It pauses the execution of the `async` function until the awaited Promise resolves, then returns the resolved value. It does NOT block the main thread — other code continues running.

**Q: Can you use `await` outside of an `async` function?**

> No. Using `await` outside an `async` function causes a SyntaxError. You must always be inside an `async` function.

**Q: Why use `Promise.all` with async/await?**

> When you have multiple independent requests, using sequential `await` makes them run one-by-one. `Promise.all` fires them simultaneously and waits for all to finish — much faster.

**Q: Does async/await replace `.then()` completely?**

> No. Async/await is syntax built on top of Promises — they work the same way. `async/await` is generally preferred for readability, but `.then()` is still valid and both approaches are used in real projects.
>
