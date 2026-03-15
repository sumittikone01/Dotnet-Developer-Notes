
# 02 — Inspecting API Response with DevTools

---

## 🎯 One-Line Definition

> **DevTools Network tab is your single most powerful debugging tool — every AJAX request, every response, every header, every error is visible there in real time, telling you exactly what went wrong and where.**

---

## 🔑 Why DevTools Is Your Best Friend

```
Without DevTools:                   With DevTools:
──────────────────────────────      ──────────────────────────────────────
Grid shows nothing                  Grid shows nothing
  ↓                                   ↓
Guess what's wrong                  Open DevTools → Network tab
Modify controller                   Click the Read request
Refresh, still wrong                See Response tab:
Modify view                         { "Data": null, "Total": 0 }
Refresh, still wrong                Immediately know: controller returning null
...hours of guessing                ...2 minutes to identify the bug
```

---

## 🔑 Opening DevTools — The 3 Ways

```
Method 1:  Press F12            (works in all browsers)
Method 2:  Press Ctrl+Shift+I   (Windows/Linux)
Method 3:  Right-click on page → "Inspect"

Then click the "Network" tab at the top of DevTools.
```

---

## 🔑 The Network Tab — What You're Looking At

```
┌──────────────────────────────────────────────────────────────────┐
│  DevTools                                                         │
│  Elements  Console  Sources  Network  Application  ...           │
│                              ↑ click here                        │
├──────────────────────────────────────────────────────────────────┤
│  🚫 ○ ▶  Preserve log  Disable cache  No throttling ▼           │
│  Filter: [All] [Fetch/XHR] [JS] [CSS] [Img] [Media] [Other]    │
│                ↑ click this to show only AJAX requests           │
├────────────┬────────┬────────┬──────────┬──────────────────────┤
│  Name      │ Status │ Type   │ Size     │ Time                  │
├────────────┼────────┼────────┼──────────┼──────────────────────┤
│ Read       │ 200    │ fetch  │ 2.4 kB   │ 85 ms    ← click this│
│ Create     │ 400    │ fetch  │ 1.1 kB   │ 42 ms    ← red = error│
│ jquery.min │ 200    │ script │ 87 kB    │ 120 ms               │
└────────────┴────────┴────────┴──────────┴──────────────────────┘
```

---

## 🔑 Step-by-Step: How to Inspect a Kendo Grid Request

```
STEP 1 — Open DevTools
  Press F12

STEP 2 — Go to Network tab
  Click "Network" at the top

STEP 3 — Filter to AJAX only
  Click "Fetch/XHR" filter button
  (hides JS/CSS/image files — only shows AJAX calls)

STEP 4 — Trigger the action
  Load the page, click Edit, click Save — whatever causes the request

STEP 5 — Find your request in the list
  Look for "Read", "Create", "Update", "Destroy"
  OR your controller action name

STEP 6 — Click the request
  A panel opens on the right with 4 tabs:
  Headers | Payload | Preview | Response

STEP 7 — Read what's there
  (see next sections for what each tab tells you)
```

---

## 🔑 The 4 Tabs Inside a Request — What Each One Tells You

```
┌──────────────────────────────────────────────────────────────────┐
│  Read  ×                                                         │
│  Headers  │  Payload  │  Preview  │  Response                   │
└──────────────────────────────────────────────────────────────────┘
```

### Tab 1 — Headers

**What it shows:** Metadata about the request and response.

```
Request URL:     http://localhost:5000/Employee/Read   ← the endpoint called
Request Method:  POST                                  ← GET or POST
Status Code:     200 OK                               ← did it work?

Request Headers (what YOUR code sent):
  Content-Type:              application/x-www-form-urlencoded
  RequestVerificationToken:  CfDJ8LrV...              ← anti-forgery token present
  X-Requested-With:          XMLHttpRequest

Response Headers (what SERVER sent back):
  Content-Type:  application/json; charset=utf-8      ← JSON response confirmed
  Cache-Control: no-cache
```

**What to look for:**

```
✅ Status 200     → request reached server and succeeded
⚠  Status 400     → request reached server but was rejected (bad data)
⚠  Status 401     → not authenticated
⚠  Status 404     → wrong URL (controller action not found)
⚠  Status 500     → server threw an exception
⚠  Status 0       → request never left browser (CORS block or no internet)
```

---

### Tab 2 — Payload (Request Body)

**What it shows:** What YOUR code sent TO the server.

```
Kendo Grid Read sends:
  page:                           1
  pageSize:                       10
  sort[0][field]:                 Name
  sort[0][dir]:                   asc
  filter[logic]:                  and
  filter[filters][0][field]:      Department
  filter[filters][0][operator]:   eq
  filter[filters][0][value]:      IT

Kendo Grid Create sends:
  Name:       Alice
  Department: IT
  Salary:     75000
  HireDate:   2021-06-15T00:00:00.000Z
  Id:         0
```

**What to look for:**

```
❓ Is the data there?
   If you expected Salary=75000 but Payload shows Salary= (empty)
   → the form field is not being read correctly

❓ Is the anti-forgery token present?
   If RequestVerificationToken is missing from headers
   → that's why you're getting 400 Bad Request

❓ Is JSON sent when [FromBody] is expected?
   Content-Type should be "application/json"
   If it says "application/x-www-form-urlencoded" → [FromBody] will be null
```

---

### Tab 3 — Preview (Formatted Response)

**What it shows:** The server's response parsed and pretty-printed — easiest to read.

```
{                                ← expand by clicking ▶
  "Data": [
    {
      "id": 1,
      "name": "Alice Johnson",
      "department": "IT",
      "salary": 75000
    },
    {
      "id": 2,
      "name": "Bob Smith",
      "department": "HR",
      "salary": 55000
    }
  ],
  "Total": 47,
  "AggregateResults": null,
  "Errors": null
}
```

**What to look for:**

```
❓ Is Data an array?
   If Data is null or empty → controller returned no data

❓ Does Total match what you expect?
   If Total: 0 but you know there are records → query filter is wrong

❓ Are the Errors field populated?
   { "Errors": { "Name": { "errors": ["Name is required"] } } }
   → server-side ModelState validation failed

❓ Do field names match your schema?
   Schema says field: "name" but response has "Name" (capital N)
   → case mismatch, grid shows empty
```

---

### Tab 4 — Response (Raw Text)

**What it shows:** The exact raw text the server sent — before any parsing.

```json
{"Data":[{"id":1,"name":"Alice","department":"IT","salary":75000}],"Total":47,"AggregateResults":null,"Errors":null}
```

**When to use Response tab instead of Preview:**

```
→ When Preview shows an error message or HTML instead of JSON
→ When you need to copy the exact response to paste somewhere
→ When Preview fails to parse (malformed JSON)

COMMON: Response shows HTML (full error page) instead of JSON:
<!DOCTYPE html>
<html>
  <head><title>500 - Internal Server Error</title></head>
  <body>...</body>
</html>
→ Server threw an unhandled exception
→ Look for the error message in the HTML to find the bug
```

---

## 🔑 Diagnosing Common Bugs Using DevTools

### Diagnosis 1 — Grid Shows Empty (No Records)

```
Open Network → Fetch/XHR → click Read request

Check 1 — Status Code:
  200? → request worked, data should be there
  404? → wrong URL in transport read
  500? → controller crashed

Check 2 — Preview tab:
  { "Data": [], "Total": 0 }
  → Query returning no results — check server-side filter

  { "Data": null, ... }
  → Controller returning null — check the query

  { "Data": [{"id":1,"name":"Alice",...}] }  ← data IS there
  → Problem is in schema mapping — check field names and schema.data
```

---

### Diagnosis 2 — Save Returns 400

```
Open Network → click the Create or Update request

Check Headers tab:
  Status Code: 400 Bad Request

Check Response tab:
  {"type":"https://tools.ietf.org/html/rfc7231#section-6.5.1",
   "title":"One or more validation errors occurred.",
   "errors":{"Name":["The Name field is required"]}}
  → Server-side validation failed — check the model you're sending

  {"message":"The required antiforgery request token was not supplied."}
  → Anti-forgery token missing — add $.ajaxSetup

Check Payload tab:
  Is the data there?
  name: Alice ✓   salary: 75000 ✓   but department is missing?
  → The Department field is not being sent — check your form field name
```

---

### Diagnosis 3 — Save Returns 500

```
Check Response tab:
  If it shows JSON error:
  { "message": "Invalid column name 'ManagerId'" }
  → SQL error — column mismatch in EF model

  If it shows full HTML error page:
  → Unhandled exception in controller
  → Read the error message in the HTML
  → OR check Visual Studio Output window for the full stack trace
```

---

### Diagnosis 4 — Kendo Grid Sends Request but Controller Not Hit

```
Check Network tab:
  Is the request there?

  NO request at all:
  → JavaScript error prevents AJAX from firing
  → Check Console tab for JS errors

  Request shows CORS error (red):
  → Cross-origin issue — controller and page on different origins
  → Check controller CORS configuration

  Request shows (canceled):
  → Request was aborted — possibly by another DataSource.read() call
```

---

## 🔑 The Console Tab — JavaScript Errors

The Network tab shows HTTP issues. The **Console tab** shows JavaScript errors.

```
Open DevTools → Console tab

Common errors you'll see:

  Uncaught ReferenceError: $ is not defined
  → jQuery not loaded or loaded after Kendo

  Uncaught TypeError: Cannot read properties of undefined (reading 'dataSource')
  → grid.dataSource called when grid is null
  → $().data("kendoGrid") returned undefined

  Uncaught TypeError: grid.dataSource.read is not a function
  → Same as above — widget not initialized

  [kendo] Error:  Template compilation failed
  → Syntax error in a column template (#= ... #)
  → Check the template string for missing # or typos
```

---

## 🔑 Checking Response in Console — Quick Debug Trick

When you want to see what a controller returns without opening a full form:

```javascript
// Quick test — paste in browser Console tab:
$.get("/Employee/Read", { page: 1, pageSize: 5 }, function(data) {
    console.log(data);
});

// OR test with POST:
$.post("/Employee/Read", { page: 1, pageSize: 5 }, function(data) {
    console.log(data);
});

// This lets you see the raw response JSON immediately
// without needing to trigger the grid
```

---

## 🔑 The Application Tab — Checking Cookies

If you're debugging authentication issues (401 errors):

```
DevTools → Application tab → Cookies → your domain

Check:
  .AspNetCore.Session     → session cookie present?
  .AspNetCore.Antiforgery → anti-forgery cookie present?
  Authentication cookie   → are you logged in?

If cookies are missing → session expired or auth not configured
```

---

## 🔑 DevTools Workflow — The Full Debugging Process

```
Grid not working?
        │
        ▼
F12 → Network → Fetch/XHR
        │
        ▼
Trigger the failing action (load, save, delete)
        │
        ▼
Find the request in the list
        │
        ├── Red (4xx or 5xx)?
        │       │
        │       ├── 400 → check Payload (what was sent)
        │       │         check Response (validation errors)
        │       │
        │       ├── 401 → check Application → Cookies (logged in?)
        │       │
        │       ├── 404 → check Headers → Request URL (wrong endpoint?)
        │       │
        │       └── 500 → check Response tab (HTML error page)
        │                  read the error message
        │
        └── Green (200)?
                │
                ▼
            Preview tab → check the JSON
                │
                ├── Data is null/empty?
                │   → fix the controller query
                │
                ├── Data is there but grid empty?
                │   → field name mismatch in schema
                │
                └── Errors field populated?
                    → ModelState validation failed
                    → fix the data being sent
```

---

## 📊 Network Tab Quick Reference

| Tab                | Shows                                                      | Use To Debug                                      |
| ------------------ | ---------------------------------------------------------- | ------------------------------------------------- |
| **Headers**  | Request URL, method, status code, request/response headers | Wrong URL, 400/401/404/500 cause, Content-Type    |
| **Payload**  | Data YOUR code sent to server                              | Missing fields, wrong data format, no token       |
| **Preview**  | Server's response (formatted)                              | What data was returned, Errors field, Total count |
| **Response** | Server's raw response text                                 | Full error pages, malformed JSON                  |

---

## ❓ Interview Questions

**Q: How do you check what a Kendo Grid is sending to the server?**

> F12 → Network tab → Fetch/XHR filter → click the Read/Create/Update request → Payload tab. This shows every field sent in the request body.

**Q: How do you check what the server returned to the Grid?**

> F12 → Network tab → click the request → Preview tab. This shows the parsed JSON response including the Data array, Total count, and Errors field.

**Q: Grid shows empty but status is 200 OK — how do you diagnose it?**

> Open DevTools → Network → Preview tab for the Read request. If data is there, the problem is in the Grid's schema configuration (field names or `schema.data` mapping). If data is missing or empty, the problem is in the controller query.

**Q: What does it mean when the Response tab shows HTML instead of JSON?**

> The server threw an unhandled exception. ASP.NET returned a 500 error page as HTML instead of JSON. Read the error message in the HTML to identify the bug, or check Visual Studio's Output window for the full stack trace.
>
