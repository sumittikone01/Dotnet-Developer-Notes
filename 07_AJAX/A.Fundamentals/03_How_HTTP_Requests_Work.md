
# 03 — How HTTP Requests Work

---

## 🎯 One-Line Definition

> **HTTP is the language browsers and servers use to talk to each other. Every AJAX call is an HTTP request sent in the background.**

---

## 📬 The Letter Analogy

HTTP works exactly like sending a letter:

```
YOU (Browser)                              POST OFFICE (Internet)              THEM (Server)
─────────────                              ──────────────────────              ─────────────

Write a letter          ─── sends ──►      travels over network    ──► Server reads it
  (HTTP Request)                                                        processes request
                                                                        writes a reply
                                                                        (HTTP Response)
Receive the reply       ◄── arrives ──     travels back            ◄── sends response
  (HTTP Response)
```

Your letter has:

* **To address** = the URL
* **Subject line** = the HTTP method (GET/POST)
* **The letter itself** = the request body (what you're sending)
* **Envelope info** = headers (metadata)

---

## 🔄 The Request-Response Rule

HTTP has ONE strict rule you must always remember:

```
┌─────────────────────────────────────────────────────┐
│                                                      │
│   ONE request  →  ALWAYS one response               │
│                                                      │
│   Browser ALWAYS initiates. Server ALWAYS responds. │
│   Server CANNOT send data unless browser asks first.│
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## 🗂️ Anatomy of an HTTP Request

Every HTTP request has 3 parts. Think of it like an envelope + letter:

```
┌──────────────────────────────────────────────────────────┐
│                      HTTP REQUEST                         │
├──────────────────────────────────────────────────────────┤
│  REQUEST LINE  (what do you want and where)              │
│  ─────────────                                           │
│  POST  /Employee/Create  HTTP/1.1                        │
│  ↑      ↑                 ↑                              │
│  Method Route/URL         Protocol version               │
├──────────────────────────────────────────────────────────┤
│  HEADERS  (information about the request)                │
│  ─────────                                               │
│  Host: yoursite.com                                      │
│  Content-Type: application/json   ← "my body is JSON"   │
│  Content-Length: 42               ← how many bytes       │
│  X-Requested-With: XMLHttpRequest ← "I'm an AJAX call"  │
├──────────────────────────────────────────────────────────┤
│  BODY  (the actual data — only for POST/PUT)             │
│  ─────                                                   │
│  {"name":"Alice","department":"IT","salary":75000}       │
└──────────────────────────────────────────────────────────┘
```

---

## 🗂️ Anatomy of an HTTP Response

```
┌──────────────────────────────────────────────────────────┐
│                      HTTP RESPONSE                        │
├──────────────────────────────────────────────────────────┤
│  STATUS LINE  (did it work?)                             │
│  ───────────                                             │
│  HTTP/1.1  200  OK                                       │
│            ↑    ↑                                        │
│            Code Short description                        │
├──────────────────────────────────────────────────────────┤
│  HEADERS                                                 │
│  Content-Type: application/json   ← "my body is JSON"   │
│  Content-Length: 128                                     │
├──────────────────────────────────────────────────────────┤
│  BODY  (the data the server sends back)                  │
│  [{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]         │
└──────────────────────────────────────────────────────────┘
```

---

## 📮 HTTP Methods — What Action Do You Want?

The method tells the server  **what you want to do** :

| Method           | Think of it as           | Has Body?     | Common Use                   |
| ---------------- | ------------------------ | ------------- | ---------------------------- |
| **GET**    | "Give me data"           | ❌ No         | Load a list, get one record  |
| **POST**   | "Here's new data"        | ✅ Yes        | Create employee, submit form |
| **PUT**    | "Replace this record"    | ✅ Yes        | Update entire employee       |
| **PATCH**  | "Change just this field" | ✅ Yes        | Update only the salary       |
| **DELETE** | "Remove this"            | ❌ Usually no | Delete employee by ID        |

### GET vs POST — Where Does the Data Go?

```
GET — data travels in the URL (visible to everyone):
──────────────────────────────────────────────────
/Employee/Search?name=Alice&dept=IT&page=2
                 ↑─────────────────────────
                 this is called the Query String

POST — data travels in the body (hidden):
──────────────────────────────────────────────────
URL:  /Employee/Create
Body: {"name":"Alice","department":"IT","salary":75000}
      ↑───────────────────────────────────────────────
      not visible in the browser address bar
```

```javascript
// GET — data in URL
$.ajax({
    url:  "/Employee/Search?name=Alice&dept=IT",
    type: "GET",
    success: function(data) { console.log(data); }
});

// POST — data in body
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",                    // tell server it's JSON
    data:        JSON.stringify({ name: "Alice" }),     // send as JSON string
    success:     function(data) { console.log(data); }
});
```

---

## 🚦 HTTP Status Codes — The Traffic Lights

The status code tells you  **what happened** . Think of it as the server's answer to your request.

```
2xx = ✅ Success   3xx = ↪️ Redirect   4xx = ❌ Your fault   5xx = 💥 Server's fault
```

### The Ones You'll Actually Use

| Code          | Name                  | What It Means             | You'll See This When      |
| ------------- | --------------------- | ------------------------- | ------------------------- |
| **200** | OK                    | Everything worked         | Normal data load          |
| **201** | Created               | New record saved          | POST to create            |
| **204** | No Content            | Worked, nothing to return | DELETE success            |
| **400** | Bad Request           | Your data was wrong       | Missing field, bad JSON   |
| **401** | Unauthorized          | Not logged in             | Session expired           |
| **403** | Forbidden             | Logged in, no permission  | Accessing admin as user   |
| **404** | Not Found             | URL/record doesn't exist  | Wrong URL, deleted record |
| **405** | Method Not Allowed    | Wrong HTTP method         | GET on a POST endpoint    |
| **500** | Internal Server Error | Server crashed            | Unhandled exception in C# |

### Easy Way to Remember

```
┌────────────────────────────────────────────────────┐
│  200 = "Yes, here's your data"                     │
│  400 = "You sent something wrong"                  │
│  401 = "Who are you? Log in first"                 │
│  403 = "I know who you are — you can't do this"    │
│  404 = "That doesn't exist"                        │
│  500 = "I broke. Not your fault."                  │
└────────────────────────────────────────────────────┘
```

### Handling Status Codes in AJAX

```javascript
$.ajax({
    url:  "/api/employees",
    type: "GET",
    success: function(data) {
        // Reached here = status 200-299
        renderTable(data);
    },
    error: function(xhr) {
        // Reached here = status 400-599
        var code = xhr.status;

        if      (code === 400) showError("Check your input — something is wrong");
        else if (code === 401) window.location.href = "/Login";
        else if (code === 403) showError("You don't have permission");
        else if (code === 404) showError("Record not found");
        else if (code === 500) showError("Server error — try again later");
    }
});
```

---

## 📋 HTTP Headers — The Key Ones You Need

Headers are extra pieces of information attached to a request or response.
Think of them as  **sticky notes on an envelope** .

### Request Headers You Set in AJAX

| Header                       | What You Set It To   | Why                                      |
| ---------------------------- | -------------------- | ---------------------------------------- |
| `Content-Type`             | `application/json` | "The body I'm sending is JSON"           |
| `Accept`                   | `application/json` | "I want JSON back from you"              |
| `RequestVerificationToken` | `<token>`          | Anti-forgery (required for ASP.NET POST) |

### Response Headers You Read

| Header           | Example Value        | What It Tells You             |
| ---------------- | -------------------- | ----------------------------- |
| `Content-Type` | `application/json` | "My body is JSON — parse it" |
| `Content-Type` | `text/html`        | "My body is HTML"             |

---

## 🌐 URL Structure — Know Every Part

```
  https://yoursite.com:443/Employee/Search?name=Alice&page=2#results
  ↑         ↑           ↑   ↑               ↑                ↑
  Protocol  Host        Port Path           Query String     Fragment

  Protocol:     https (secure) or http
  Host:         yoursite.com — the server address
  Port:         443 for https, 80 for http (usually hidden)
  Path:         /Employee/Search — maps to Controller/Action
  Query String: ?name=Alice&page=2 — data sent with GET requests
  Fragment:     #results — browser only, NEVER sent to server
```

---

## 🔍 Seeing HTTP Requests With Your Own Eyes — DevTools

The most important debugging skill. Do this right now on any website:

```
Step 1: Press F12   (opens DevTools)
Step 2: Click "Network" tab
Step 3: Click "XHR" or "Fetch/XHR" filter
Step 4: Trigger an AJAX call on the page
Step 5: Click any request in the list

You'll see:
┌─────────────────────────────────────────────────────┐
│  Headers tab                                        │
│  ──────────                                         │
│  Request URL:    http://yoursite.com/api/employees  │
│  Request Method: POST                               │
│  Status Code:    200 OK                             │
│  Content-Type:   application/json                   │
├─────────────────────────────────────────────────────┤
│  Payload tab  (what YOU sent)                       │
│  ────────────                                       │
│  {"sort":"name","page":1,"pageSize":10}             │
├─────────────────────────────────────────────────────┤
│  Response tab  (what SERVER sent back)              │
│  ─────────────                                      │
│  {"Data":[{"id":1,"name":"Alice"}...],"Total":50}   │
└─────────────────────────────────────────────────────┘
```

> **This is how you debug Kendo Grid data issues.** If the grid shows nothing, go to Network → click the Read request → check the Response tab. If the JSON is there, the problem is in the grid config. If it's not there or shows an error, the problem is in the controller.

---

## 🔄 A Complete AJAX Cycle — Real Example

Here is every step when your Kendo Grid loads employee data:

```
BROWSER                                              SERVER
───────                                              ──────

[1] Page loads, Kendo Grid initialises
[2] Grid tells DataSource to read data

[3] Browser sends:
    POST /Employee/Read HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Body: page=1&pageSize=10&sort[0][field]=Name&sort[0][dir]=asc

    ──────────────── travels over internet ──────────────►

                                        [4] ASP.NET receives request
                                        [5] [DataSourceRequest] reads params
                                        [6] .ToDataSourceResult() queries DB
                                        [7] Returns JSON

    ◄──────────────── travels back ──────────────────────

[8] Browser receives:
    HTTP/1.1 200 OK
    Content-Type: application/json
    Body: {"Data":[{"id":1,"name":"Alice"},...],"Total":50}

[9] jQuery parses JSON automatically
[10] Kendo Grid renders 10 rows in the table

TOTAL TIME: ~100-300ms. Page never reloaded.
```

---

## ✅ Key Takeaways

```
┌────────────────────────────────────────────────────────┐
│  Remember These                                         │
│                                                         │
│  • HTTP = the language of web communication            │
│  • Every AJAX call = one HTTP request + one response   │
│  • GET = data in URL   POST = data in body             │
│  • 2xx = success   4xx = your fault   5xx = their fault│
│  • Content-Type: application/json → critical for POST  │
│  • F12 → Network → XHR = your best debugging friend   │
└────────────────────────────────────────────────────────┘
```

---

## ❓ Interview Questions

**Q: What is HTTP?**

> The communication protocol browsers and servers use. Every AJAX request is an HTTP request sent in the background.

**Q: What is the difference between GET and POST?**

> GET sends data in the URL (visible, limited size, for fetching). POST sends data in the request body (hidden, no size limit, for sending/saving data).

**Q: What does status code 400 mean?**

> Bad Request — the server received your request but something you sent was wrong. Common causes: missing required fields, invalid JSON, validation failure.

**Q: What does status code 401 vs 403 mean?**

> 401 = not authenticated (you haven't logged in). 403 = authenticated but not authorized (you're logged in, but you don't have permission for this action).

**Q: What is `Content-Type: application/json`?**

> A header that tells the server the request body contains JSON data. Without it, ASP.NET's `[FromBody]` won't deserialize your JSON.

**Q: How do you debug an AJAX request that isn't working?**

> Open DevTools (F12) → Network tab → filter by XHR → trigger the action → click the request → check Headers for the status code → check Payload for what you sent → check Response for what the server returned.

---
