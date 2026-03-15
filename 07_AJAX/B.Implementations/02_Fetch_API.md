
# 02 — Fetch API

---

## 🎯 One-Line Definition

> **Fetch is the modern, built-in browser function for making HTTP requests — cleaner than XHR, returns a Promise, and works without any library.**

---

## 🔄 XHR vs Fetch — The Same Job, Different Feel

```javascript
// ── Same GET request, two ways ────────────────────────────────

// XHR (old way) — verbose, event-based
var xhr = new XMLHttpRequest();
xhr.open("GET", "/api/employees");
xhr.responseType = "json";
xhr.onload = function() { console.log(xhr.response); };
xhr.send();

// Fetch (modern way) — concise, Promise-based
fetch("/api/employees")
    .then(response => response.json())
    .then(data => console.log(data));
```

Same result. Fetch is just cleaner.

---

## 🔑 The Golden Rule of Fetch

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  fetch() ALWAYS resolves — even for 400 and 500 errors. │
│                                                          │
│  .catch() only fires for NETWORK failures.              │
│                                                          │
│  You MUST check response.ok or response.status manually.│
│                                                          │
└──────────────────────────────────────────────────────────┘
```

This surprises everyone at first. Learn it now so it doesn't bite you later.

---

## 💻 Basic GET Request

```javascript
fetch("/api/employees")
    .then(function(response) {
        // Step 1: response is the HTTP response object
        // response.ok = true if status is 200-299
        // response.status = the actual status code
        // response.body is a ReadableStream — NOT the data yet
        return response.json();   // Step 2: parse the body → returns another Promise
    })
    .then(function(data) {
        // Step 3: data is now your actual JavaScript array/object
        console.log(data);
        renderTable(data);
    })
    .catch(function(error) {
        // ONLY fires for: no internet, DNS failure, CORS block
        // Does NOT fire for 400, 500 etc.
        console.error("Network error:", error);
    });
```

---

## 💻 GET with Error Handling Done Correctly

```javascript
fetch("/api/employees")
    .then(function(response) {
        // Check the status BEFORE parsing
        if (!response.ok) {
            // response.ok is false for 400, 404, 500 etc.
            throw new Error("Request failed: " + response.status);
            // Throwing here sends it to .catch()
        }
        return response.json();
    })
    .then(function(data) {
        renderTable(data);
    })
    .catch(function(error) {
        // Now catches BOTH network failures AND HTTP errors
        console.error("Error:", error.message);
        showErrorMessage(error.message);
    });
```

---

## 💻 POST Request — Sending JSON

```javascript
var newEmployee = {
    name:       "Alice",
    department: "IT",
    salary:     75000
};

fetch("/Employee/Create", {
    method:  "POST",
    headers: {
        "Content-Type": "application/json"   // REQUIRED — tells server body is JSON
    },
    body: JSON.stringify(newEmployee)        // REQUIRED — convert object to string
})
    .then(function(response) {
        if (!response.ok) {
            throw new Error("Save failed: " + response.status);
        }
        return response.json();
    })
    .then(function(saved) {
        console.log("Saved with ID:", saved.id);
    })
    .catch(function(error) {
        console.error("Error:", error.message);
    });
```

---

## 🔑 The `fetch()` Options Object

```javascript
fetch(url, {
    method:      "POST",           // GET, POST, PUT, PATCH, DELETE
    headers: {
        "Content-Type":  "application/json",
        "Authorization": "Bearer " + token,
        "Accept":        "application/json"
    },
    body:        JSON.stringify(data),  // only for POST, PUT, PATCH
    mode:        "cors",           // "cors", "no-cors", "same-origin"
    credentials: "include",        // "omit", "same-origin", "include"
                                   // "include" sends cookies with request
    cache:       "no-cache"        // "default", "no-store", "reload", "no-cache"
})
```

---

## 🔑 The Response Object — What You Get Back

```javascript
fetch("/api/employees")
    .then(function(response) {

        // ── Status info ───────────────────────────────────────
        response.status      // 200, 400, 404, 500...
        response.statusText  // "OK", "Not Found"...
        response.ok          // true if status is 200-299, false otherwise

        // ── Headers ───────────────────────────────────────────
        response.headers.get("Content-Type")  // "application/json"

        // ── Body — choose ONE method to read it ───────────────
        // (each can only be called once)
        return response.json();     // parse as JSON   → returns Promise
        return response.text();     // read as string  → returns Promise
        return response.blob();     // read as Blob    → for files/images
        return response.formData(); // parse as FormData
    })
```

---

## 💻 Multiple Requests — Promise Chaining

```javascript
// Chain: load employee, then load their department
fetch("/api/employees/1")
    .then(r => r.json())
    .then(function(employee) {
        return fetch("/api/departments/" + employee.deptId);
    })
    .then(r => r.json())
    .then(function(department) {
        console.log("Employee's dept:", department.name);
    })
    .catch(error => console.error(error));
```

---

## 💻 Multiple Requests in Parallel — `Promise.all`

```javascript
// Fire ALL requests at once, wait for ALL to finish
Promise.all([
    fetch("/api/employees").then(r => r.json()),
    fetch("/api/departments").then(r => r.json()),
    fetch("/api/stats").then(r => r.json())
])
    .then(function(results) {
        var employees   = results[0];
        var departments = results[1];
        var stats       = results[2];

        // All 3 are available here simultaneously
        renderGrid(employees);
        populateDropdown(departments);
        updateStats(stats);
    })
    .catch(error => console.error("One of the requests failed:", error));
```

---

## 📊 Response Parsing Methods

| Method                  | Use For               | Returns                 |
| ----------------------- | --------------------- | ----------------------- |
| `response.json()`     | JSON data from API    | Promise → Object/Array |
| `response.text()`     | HTML, plain text, XML | Promise → String       |
| `response.blob()`     | Images, files, PDFs   | Promise → Blob         |
| `response.formData()` | Form submissions      | Promise → FormData     |

> Each method returns a Promise and can only be called **once** per response.

---

## 📊 Common Fetch Patterns

```javascript
// ── GET ──────────────────────────────────────────────────────
fetch("/api/employees")
    .then(r => r.json())
    .then(data => console.log(data));

// ── POST ─────────────────────────────────────────────────────
fetch("/api/employees", {
    method:  "POST",
    headers: { "Content-Type": "application/json" },
    body:    JSON.stringify({ name: "Alice" })
}).then(r => r.json()).then(data => console.log(data));

// ── PUT ──────────────────────────────────────────────────────
fetch("/api/employees/1", {
    method:  "PUT",
    headers: { "Content-Type": "application/json" },
    body:    JSON.stringify({ name: "Alice Updated" })
}).then(r => r.json());

// ── DELETE ───────────────────────────────────────────────────
fetch("/api/employees/1", {
    method: "DELETE"
}).then(r => {
    if (r.ok) console.log("Deleted");
});
```

---

## ⚠️ Common Mistakes

| Mistake                                    | Symptom                                             | Fix                                                     |
| ------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------- |
| Not checking `response.ok`               | 400/500 silently treated as success                 | Always check `if (!response.ok) throw new Error(...)` |
| Forgetting `Content-Type`header on POST  | Controller receives null model /`[FromBody]`fails | Add `"Content-Type": "application/json"`to headers    |
| Forgetting `JSON.stringify()`in body     | Body is `[object Object]`string, not JSON         | Wrap data in `JSON.stringify()`                       |
| Reading body twice                         | Error: "body already used"                          | Call only one body method (`.json()`or `.text()`)   |
| Treating `.catch()`as HTTP error handler | 400/500 not caught                                  | Check `response.ok`in `.then()`and throw if needed  |

---

## ❓ Interview Questions

**Q: What is the Fetch API?**

> A modern browser built-in function for making HTTP requests. It returns a Promise, has a cleaner syntax than XHR, and requires no external library.

**Q: Does `.catch()` fire when the server returns a 500 error?**

> No. Fetch only rejects (triggers `.catch()`) for network failures — no internet, DNS errors, CORS blocks. HTTP error responses (400, 500, etc.) resolve normally. You must check `response.ok` in the first `.then()` and throw manually to route them to `.catch()`.

**Q: What does `response.ok` mean?**

> It's `true` if the HTTP status code is between 200 and 299, and `false` for anything outside that range (400, 404, 500, etc.).

**Q: What is `Promise.all` used for with Fetch?**

> To fire multiple fetch requests at the same time and wait until all of them complete. More efficient than chaining them one after another when the requests are independent.

**Q: Why must you call `response.json()` instead of reading the response directly?**

> The fetch response body is a ReadableStream, not the parsed data. You must call `.json()` (or `.text()`, `.blob()`) to read and parse it — each returns a Promise that resolves with the actual data.
>
