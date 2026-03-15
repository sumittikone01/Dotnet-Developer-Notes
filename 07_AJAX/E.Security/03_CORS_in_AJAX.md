
# 03 — CORS in AJAX

---

## 🎯 One-Line Definition

> **CORS (Cross-Origin Resource Sharing) is a browser security rule that blocks AJAX calls to a different domain — unless that domain explicitly says "I allow requests from your origin."**

---

## 🔑 What is an "Origin"?

An origin = protocol + domain + port. All three must match.

```
URL                              Origin
──────────────────────────────   ──────────────────────────────
https://myapp.com/page           https://myapp.com
https://myapp.com/other          https://myapp.com   ← SAME origin
http://myapp.com                 http://myapp.com    ← DIFFERENT (http vs https)
https://myapp.com:8080           https://myapp.com:8080 ← DIFFERENT (port)
https://api.myapp.com            https://api.myapp.com  ← DIFFERENT (subdomain)
https://otherapp.com             https://otherapp.com   ← DIFFERENT (domain)
```

---

## 🔑 The Same-Origin Policy — Why CORS Exists

Browsers have a built-in rule called the  **Same-Origin Policy** :

```
JavaScript on page A can only make AJAX calls to the SAME origin as page A.

https://myapp.com  →  https://myapp.com/api/data     ✅ same origin, allowed
https://myapp.com  →  https://api.myapp.com/data      ❌ different origin, BLOCKED
https://myapp.com  →  https://otherapp.com/data       ❌ different origin, BLOCKED
https://myapp.com  →  http://myapp.com/data            ❌ different protocol, BLOCKED
```

This protects users — without it, any website could make AJAX calls to your bank, read your emails, etc. using your cookies.

---

## 🔑 What a CORS Error Looks Like

You open the browser console and see:

```
Access to XMLHttpRequest at 'https://api.myapp.com/employees'
from origin 'https://myapp.com' has been blocked by CORS policy:
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

This happens entirely in the  **browser** . The server did receive and process the request — the browser just refuses to give you the response.

---

## 🔑 How CORS Works — The Browser Checks Headers

```
SIMPLE REQUEST (GET with no custom headers):
─────────────────────────────────────────────────────────────────
Browser sends request to api.myapp.com
Server processes it, sends response back
Browser checks response for:
  Access-Control-Allow-Origin: https://myapp.com  ← is your origin listed?

  Yes → give response to JavaScript ✅
  No  → block response, show CORS error ❌


PREFLIGHT REQUEST (POST, PUT, DELETE, or custom headers):
─────────────────────────────────────────────────────────────────
Browser FIRST sends OPTIONS request to api.myapp.com:
  Origin: https://myapp.com
  Access-Control-Request-Method: POST
  Access-Control-Request-Headers: Content-Type, RequestVerificationToken

Server responds with what it allows:
  Access-Control-Allow-Origin: https://myapp.com
  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  Access-Control-Allow-Headers: Content-Type, RequestVerificationToken

If allowed → browser sends the REAL request ✅
If not     → browser blocks, CORS error ❌
```

---

## 🔑 Configuring CORS in ASP.NET Core

```csharp
// Program.cs

// ── Step 1: Define a CORS policy ──────────────────────────────
builder.Services.AddCors(options =>
{
    // Policy 1: specific origins (most secure — use in production)
    options.AddPolicy("AllowMyFrontend", policy =>
    {
        policy
            .WithOrigins(
                "https://myapp.com",
                "https://www.myapp.com",
                "https://staging.myapp.com"
            )
            .WithMethods("GET", "POST", "PUT", "DELETE")
            .WithHeaders("Content-Type", "Authorization", "RequestVerificationToken")
            .AllowCredentials();    // allow cookies to be sent
    });

    // Policy 2: allow any origin (development only — NEVER production)
    options.AddPolicy("AllowAll", policy =>
    {
        policy
            .AllowAnyOrigin()
            .AllowAnyMethod()
            .AllowAnyHeader();
        // Note: AllowAnyOrigin() cannot be combined with AllowCredentials()
    });

    // Policy 3: allow local development origins
    options.AddPolicy("DevelopmentOnly", policy =>
    {
        policy
            .WithOrigins("http://localhost:3000", "https://localhost:7001")
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials();
    });
});

var app = builder.Build();

// ── Step 2: Apply CORS middleware ─────────────────────────────
// IMPORTANT: UseCors() must come BEFORE UseAuthentication() and UseAuthorization()
app.UseRouting();
app.UseCors("AllowMyFrontend");    // ← apply the named policy
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
```

---

## 🔑 Applying CORS at Different Levels

```csharp
// ── Global (all controllers) — set in Program.cs ──────────────
app.UseCors("AllowMyFrontend");


// ── Per-controller ────────────────────────────────────────────
[ApiController]
[Route("api/[controller]")]
[EnableCors("AllowMyFrontend")]    // apply specific policy to this controller
public class EmployeeApiController : ControllerBase
{...}


// ── Per-action ────────────────────────────────────────────────
[HttpGet]
[EnableCors("AllowAll")]           // one action allows more origins
public IActionResult GetPublicData() {...}

[HttpPost]
[EnableCors("AllowMyFrontend")]    // another action is more restrictive
public IActionResult CreateEmployee() {...}


// ── Disable CORS for a specific action ────────────────────────
[HttpGet("internal")]
[DisableCors]                      // no cross-origin allowed for this one
public IActionResult GetInternalData() {...}
```

---

## 🔑 The CORS Headers — What They Mean

```
Response headers the server sends back:

  Access-Control-Allow-Origin: https://myapp.com
  → Only this origin can read the response
  → Use * to allow any origin (development only)

  Access-Control-Allow-Methods: GET, POST, PUT, DELETE
  → These HTTP methods are allowed from other origins

  Access-Control-Allow-Headers: Content-Type, Authorization
  → These request headers are allowed in cross-origin requests

  Access-Control-Allow-Credentials: true
  → Cookies can be sent with cross-origin requests
  → Requires WithOrigins (specific) — not AllowAnyOrigin()

  Access-Control-Max-Age: 86400
  → Cache the preflight response for this many seconds
  → Reduces OPTIONS preflight requests (86400 = 1 day)
```

---

## 🔑 CORS vs CSRF — They Are Different Problems

People confuse these. They are completely separate:

```
┌───────────────────────────────────────────────────────────────┐
│  CSRF                          │  CORS                        │
├────────────────────────────────┼──────────────────────────────┤
│  Attack: tricks browser to     │  Browser policy: blocks      │
│  make a request using YOUR     │  reading responses from      │
│  session                       │  other origins               │
│                                │                              │
│  Attacker: malicious website   │  Not an attack — a browser   │
│                                │  security feature            │
│                                │                              │
│  Fix: anti-forgery token       │  Fix: server sends           │
│                                │  Allow-Origin header         │
│                                │                              │
│  Who is protected: server      │  Who is protected: user      │
│  (from unauthorized actions)   │  (from data theft)           │
└───────────────────────────────────────────────────────────────┘
```

---

## 💻 CORS in Practice — When You Actually Hit It

```
You won't hit CORS when:
  ✅ Your MVC view calls your own MVC controller
     (both on https://myapp.com — same origin)
  ✅ Kendo Grid calls /Employee/Read on the same server
  ✅ Any AJAX call where the frontend and backend share a domain

You WILL hit CORS when:
  ❌ Frontend: https://myapp.com
     Backend:  https://api.myapp.com     ← different subdomain
  ❌ Frontend: http://localhost:3000 (React dev server)
     Backend:  https://localhost:7001    ← different port
  ❌ Frontend on CDN calling a separate API server
  ❌ Mobile app calling your API
```

---

## 💻 CORS for Local Development

```csharp
// Program.cs — different CORS for dev vs production
if (app.Environment.IsDevelopment())
{
    app.UseCors("DevelopmentOnly");   // allow localhost ports
}
else
{
    app.UseCors("AllowMyFrontend");   // only your real domain
}
```

```javascript
// In development: your frontend on localhost:3000
// calls your API on localhost:7001
// Without CORS setup → browser blocks it

// With the DevelopmentOnly policy above → works ✅
fetch("https://localhost:7001/api/employees")
    .then(r => r.json())
    .then(data => renderTable(data));
```

---

## ⚠️ Common Mistakes

| Mistake                                          | Symptom                                         | Fix                                               |
| ------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------- |
| `AllowAnyOrigin()`+`AllowCredentials()`      | Runtime error: these two cannot be combined     | Use `WithOrigins(...)`when credentials needed   |
| `UseCors()`after `UseAuthorization()`        | CORS headers not added for authenticated routes | Move `UseCors()`before `UseAuthentication()`  |
| Not including the AJAX header in `WithHeaders` | Preflight blocked, OPTIONS request fails        | Add all custom headers to `WithHeaders()`       |
| `AllowAnyOrigin()`in production                | Any website can call your API                   | Always use `WithOrigins(...)`in production      |
| CORS configured but not applied                  | Same CORS error                                 | Call `app.UseCors("PolicyName")`in the pipeline |

---

## ❓ Interview Questions

**Q: What is CORS?**

> Cross-Origin Resource Sharing — a browser security mechanism that blocks AJAX responses from origins different from the page's origin, unless the server explicitly allows it with `Access-Control-Allow-Origin` headers.

**Q: What is an "origin"?**

> Protocol + domain + port. `https://myapp.com`, `http://myapp.com`, `https://myapp.com:8080`, and `https://api.myapp.com` are all different origins.

**Q: Does CORS protect the server or the browser user?**

> The browser user. CORS prevents malicious websites from reading responses from another origin using your credentials. Note: the server still processes the request — CORS only blocks the browser from giving the response to the JavaScript.

**Q: What is a preflight request?**

> An automatic OPTIONS request the browser sends before the real request for non-simple requests (POST with custom headers, PUT, DELETE). The server must respond with allowed origins, methods, and headers. If the preflight fails, the real request is never sent.

**Q: Why can't you use `AllowAnyOrigin()` with `AllowCredentials()`?**

> When credentials (cookies) are included, `Access-Control-Allow-Origin: *` is not allowed by the CORS spec — a wildcard with credentials would be a security hole. You must specify the exact origins with `WithOrigins(...)` when allowing credentials.

**Q: When do you NOT encounter CORS in an ASP.NET MVC project?**

> When the frontend (Razor views) and backend (controllers) are served from the same domain and port. Kendo Grid AJAX calls to controllers on the same server never trigger CORS — they're same-origin requests.
>
