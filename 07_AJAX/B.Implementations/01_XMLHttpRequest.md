
# 01 — XMLHttpRequest (XHR)

---

## 🎯 One-Line Definition

> **XMLHttpRequest is the original browser object that makes AJAX possible — it opens a connection to the server, sends a request, and gives you the response through event handlers.**

---

## 🏛️ Why Learn This?

You will rarely write XHR from scratch today — jQuery and Fetch are simpler.
But you **must** know XHR because:

```
├── Every interview asks "how does AJAX work under the hood?"
├── jQuery's $.ajax and $.get use XHR internally
├── You'll see XHR in legacy code at work
└── Understanding XHR makes everything else make sense
```

---

## 🔄 XHR — The Full Lifecycle

```
1. CREATE        → new XMLHttpRequest()
2. CONFIGURE     → .open(method, url)
3. SET HANDLERS  → .onload, .onerror (what to do when done)
4. SEND          → .send(body)
5. WAIT          → browser sends request, your code keeps running
6. RESPONSE      → handler fires with the result
```

---

## 💻 Basic GET Request — Step by Step

```javascript
// STEP 1: Create the XHR object
var xhr = new XMLHttpRequest();

// STEP 2: Configure it — method and URL
xhr.open("GET", "/api/employees");

// STEP 3: Set response type (optional but recommended)
xhr.responseType = "json";  // browser will parse JSON automatically

// STEP 4: Attach your handlers BEFORE sending
xhr.onload = function() {
    // Fires when response arrives successfully
    if (xhr.status === 200) {
        var employees = xhr.response;   // already parsed (because responseType = "json")
        console.log(employees);
        renderTable(employees);
    } else {
        console.error("Server returned:", xhr.status);
    }
};

xhr.onerror = function() {
    // Fires only on network failure (no internet, server unreachable)
    console.error("Network error — request never reached server");
};

// STEP 5: Send the request
xhr.send();
// Code CONTINUES here immediately — doesn't wait for response
console.log("Request sent, still running...");
```

---

## 💻 POST Request — Sending JSON to Controller

```javascript
var employee = {
    name:       "Alice",
    department: "IT",
    salary:     75000
};

var xhr = new XMLHttpRequest();

xhr.open("POST", "/Employee/Create");

// MUST set this header so controller knows body is JSON
xhr.setRequestHeader("Content-Type", "application/json");

xhr.responseType = "json";

xhr.onload = function() {
    if (xhr.status === 200 || xhr.status === 201) {
        console.log("Created:", xhr.response);
    } else if (xhr.status === 400) {
        console.error("Validation failed:", xhr.response);
    } else {
        console.error("Error:", xhr.status);
    }
};

xhr.onerror = function() {
    console.error("Network error");
};

// JSON.stringify converts object → string for the body
xhr.send(JSON.stringify(employee));
```

---

## 🔑 XHR readyState — The 5 States

XHR goes through 5 states during a request. `onload` fires at state 4.

```
State  Name             Meaning
─────  ───────────────  ─────────────────────────────────────────────
  0    UNSENT           xhr created, .open() not called yet
  1    OPENED           .open() called, connection configured
  2    HEADERS_RECEIVED Server sent response headers back
  3    LOADING          Response body is being received (downloading)
  4    DONE             Response fully received — use it now ✅
```

```javascript
// onreadystatechange fires at EVERY state change
xhr.onreadystatechange = function() {
    console.log("State:", xhr.readyState);

    if (xhr.readyState === 4) {
        // Response fully received — safe to read it now
        if (xhr.status === 200) {
            var data = JSON.parse(xhr.responseText);
            console.log(data);
        }
    }
};

// Modern way — onload fires only at state 4 (DONE)
// Use onload instead — cleaner and less code
xhr.onload = function() {
    // readyState is always 4 here
    if (xhr.status === 200) {
        var data = JSON.parse(xhr.responseText);
        console.log(data);
    }
};
```

---

## 🔑 Key XHR Properties

| Property             | What It Contains                      | Example                         |
| -------------------- | ------------------------------------- | ------------------------------- |
| `xhr.status`       | HTTP status code                      | `200`,`400`,`404`,`500` |
| `xhr.statusText`   | Status description                    | `"OK"`,`"Not Found"`        |
| `xhr.responseText` | Raw response as string                | `'[{"id":1,"name":"Alice"}]'` |
| `xhr.response`     | Parsed response (if responseType set) | `[{id:1, name:"Alice"}]`      |
| `xhr.readyState`   | Current stage (0-4)                   | `4`= done                     |

---

## 🔑 Key XHR Methods

| Method                             | What It Does           | When to Use           |
| ---------------------------------- | ---------------------- | --------------------- |
| `xhr.open(method, url)`          | Configure the request  | Before send           |
| `xhr.setRequestHeader(key, val)` | Add a header           | Before send           |
| `xhr.send(body)`                 | Fire the request       | Last step             |
| `xhr.abort()`                    | Cancel the request     | User cancels, timeout |
| `xhr.getResponseHeader(key)`     | Read a response header | After onload fires    |

---

## 🔑 Key XHR Event Handlers

| Handler                    | When It Fires                                                         |
| -------------------------- | --------------------------------------------------------------------- |
| `xhr.onload`             | Response received (any status code — 200, 400, 500 all trigger this) |
| `xhr.onerror`            | Network failure only (no internet, DNS fail, server unreachable)      |
| `xhr.ontimeout`          | Request took longer than `xhr.timeout`ms                            |
| `xhr.onprogress`         | While response body is downloading (useful for large files)           |
| `xhr.onabort`            | When `xhr.abort()`is called                                         |
| `xhr.onreadystatechange` | Every time readyState changes (0 → 1 → 2 → 3 → 4)                 |

> ⚠️ `onerror` does NOT fire for 400 or 500 responses. Those are still valid HTTP responses — they fire `onload`. `onerror` only fires when the request never gets a response at all.

---

## 💻 XHR with Timeout

```javascript
var xhr = new XMLHttpRequest();
xhr.open("GET", "/api/employees");
xhr.timeout = 5000;   // 5 seconds — if no response by then, abort

xhr.onload    = function() { console.log("Got response:", xhr.status); };
xhr.onerror   = function() { console.error("Network error"); };
xhr.ontimeout = function() { console.error("Request timed out after 5 seconds"); };

xhr.send();
```

---

## 💻 Reusable XHR Helper Function

Rather than writing all this every time, wrap it:

```javascript
function makeRequest(method, url, data, onSuccess, onError) {
    var xhr = new XMLHttpRequest();
    xhr.open(method, url);
    xhr.responseType = "json";

    if (data) {
        xhr.setRequestHeader("Content-Type", "application/json");
    }

    xhr.onload = function() {
        if (xhr.status >= 200 && xhr.status < 300) {
            onSuccess(xhr.response);
        } else {
            onError(xhr.status, xhr.response);
        }
    };

    xhr.onerror = function() {
        onError(0, "Network error");
    };

    xhr.send(data ? JSON.stringify(data) : null);
}

// Usage:
makeRequest("GET", "/api/employees", null,
    function(data)        { renderTable(data); },
    function(code, error) { showError(code + ": " + error); }
);

makeRequest("POST", "/api/employees", { name: "Alice", dept: "IT" },
    function(data)        { showSuccess("Created: " + data.id); },
    function(code, error) { showError("Failed: " + error); }
);
```

---

## 📊 XHR vs jQuery AJAX vs Fetch

|                 | XHR         | jQuery $.ajax        | Fetch API     |
| --------------- | ----------- | -------------------- | ------------- |
| Verbosity       | High        | Low                  | Medium        |
| Promise support | ❌ No       | ❌ No (callbacks)    | ✅ Yes        |
| Browser support | All         | All (needs jQuery)   | All modern    |
| Auto JSON parse | ❌ Manual   | ✅ Yes               | ✅`.json()` |
| Used today      | Legacy code | MVC + Kendo projects | Modern JS     |

---

## ❓ Interview Questions

**Q: What is XMLHttpRequest?**

> The built-in browser object that enables AJAX. It opens a connection to a server, sends an HTTP request asynchronously, and delivers the response through event handlers like `onload` and `onerror`.

**Q: What is the difference between `onload` and `onerror` in XHR?**

> `onload` fires whenever a response is received — even for 400 and 500 error responses. `onerror` fires only when the request fails to reach the server at all — network failure, DNS error, server unreachable.

**Q: What does readyState 4 mean?**

> The request is complete — the full response has been received and is ready to use. This is the state where you safely read `xhr.response` or `xhr.responseText`.

**Q: Why do you call `xhr.send()` last?**

> All configuration — `open()`, `setRequestHeader()`, event handlers — must be set before `send()`. Once `send()` is called, the request is fired immediately.
>
