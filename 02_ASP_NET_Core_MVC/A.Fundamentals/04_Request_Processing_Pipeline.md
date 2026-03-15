# 04 — Request Processing Pipeline

---

## 🎯 One-Line Definition

> **The request processing pipeline is the ordered chain of middleware components that every HTTP request passes through — from the moment it arrives at your server to the moment a response is sent back to the client.**

---

## 🔷 The Big Picture — What Happens When a Request Arrives

```
Browser / AJAX / Kendo
        │
        │  HTTP Request: GET /Products/Index
        ▼
┌───────────────────────────────────────────────────────────┐
│               Kestrel (Web Server)                        │
│  Accepts the TCP connection, reads raw HTTP bytes         │
└───────────────────────┬───────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────────┐
│              ASP.NET Core Pipeline                        │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Middleware 1 → Middleware 2 → ... → Your Action   │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────┬───────────────────────────────────┘
                        │
        HTTP Response travels BACK through the same chain
                        │
                        ▼
                    Browser
```

Every request goes **in** through all middleware → hits your controller → response goes **back out** through all middleware.

---

## 🔷 Middleware — What It Actually Is

> **Middleware** = a C# class/function that can:
>
> * Inspect the incoming request
> * Do something (auth check, log, redirect)
> * Pass it forward to the next middleware
> * Inspect the outgoing response on the way back

Think of it as  **airport security checkpoints** :

```
Passenger (Request) enters airport
         │
    ┌────▼──────────────────────────────────────┐
    │  Check 1: HTTPS Redirect                   │
    │  Is it HTTP? → Redirect to HTTPS          │
    │  Is it HTTPS? → Pass through              │
    └────┬──────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────┐
    │  Check 2: Static Files                     │
    │  Is URL a file (CSS/JS/image)?            │
    │  → Yes: Return file directly. DONE.       │
    │  → No: Pass through                       │
    └────┬──────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────┐
    │  Check 3: Routing                          │
    │  Which controller handles this URL?       │
    └────┬──────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────┐
    │  Check 4: Authentication                   │
    │  Who is this person? (Read JWT/cookie)    │
    └────┬──────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────┐
    │  Check 5: Authorization                    │
    │  Are they ALLOWED to be here?             │
    └────┬──────────────────────────────────────┘
         │
    ┌────▼──────────────────────────────────────┐
    │  YOUR CONTROLLER ACTION RUNS              │
    │  GetAll() → ADO.NET → SQL Server → JSON  │
    └────┬──────────────────────────────────────┘
         │
    Response travels BACK UP through each checkpoint
    (each middleware can also modify the response)
         │
         ▼
    HTTP Response sent to browser
```

---

## 🔷 The Standard Pipeline — Your Real Program.cs

```csharp
var app = builder.Build();

// ─── ORDER MATTERS. This is the correct order ─────────────────────

// 1. EXCEPTION HANDLING — must be FIRST to catch all errors
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

// 2. HTTPS REDIRECT — redirect HTTP to HTTPS
app.UseHttpsRedirection();

// 3. STATIC FILES — serve CSS/JS/images from wwwroot
//    Short-circuits here for static files (never hits routing/controllers)
app.UseStaticFiles();

// 4. ROUTING — figure out WHICH controller matches this URL
app.UseRouting();

// 5. CORS — add cross-origin headers (must be after Routing)
app.UseCors("MyPolicy");

// 6. AUTHENTICATION — READ identity (who are you?)
//    Must come BEFORE Authorization
app.UseAuthentication();

// 7. AUTHORIZATION — CHECK permission (are you allowed?)
app.UseAuthorization();

// 8. MAP CONTROLLERS — hand off to MVC controllers
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

---

## 🔷 Short-Circuiting — When a Middleware Stops the Chain

Not all requests reach your controller. Middleware can **short-circuit** and return a response immediately:

```
GET /wwwroot/css/site.css
         │
    UseStaticFiles() checks: "is this a static file?"
         │
         ├─ YES → Returns the file. Stops here.
         │         Authentication, Controllers NEVER run.
         │
         └─ NO  → Passes to next middleware
```

```
GET /api/products  (no JWT token)
         │
    UseAuthentication() — no token found, sets User = Anonymous
         │
    UseAuthorization() checks [Authorize] attribute
         │
         └─ FAILS → Returns 401 immediately.
                    Controller action NEVER runs.
```

---

## 🔷 Request vs Response — The Two-Way Flow

```
Request Direction (→ inward):
──────────────────────────────────────────────
UseHttpsRedirection
    └→ UseStaticFiles
           └→ UseRouting
                  └→ UseAuthentication
                         └→ UseAuthorization
                                └→ Controller Action
                                       └→ generates response

Response Direction (← outward):
──────────────────────────────────────────────
Controller Action returns Ok(data)
    ↑ JSON serialized
UseAuthorization  ← can add headers on the way back
UseAuthentication ← can modify response
UseRouting        ← no-op on way back
UseStaticFiles    ← no-op on way back
UseHttpsRedirection ← no-op on way back
    ↑
Kestrel sends response bytes to browser
```

---

## 🔷 Middleware Internals — How It Works in Code

Every middleware is just a function that receives the request and calls `next()` to pass forward:

```csharp
// What middleware looks like internally:
app.Use(async (HttpContext context, RequestDelegate next) =>
{
    // ← CODE HERE runs BEFORE the next middleware (on the way IN)
    Console.WriteLine($"[IN]  {context.Request.Method} {context.Request.Path}");

    await next(context);   // ← calls the NEXT middleware in the chain

    // ← CODE HERE runs AFTER the next middleware returns (on the way BACK)
    Console.WriteLine($"[OUT] {context.Response.StatusCode}");
});
```

```
Your logging middleware above wraps everything like this:

  [IN] GET /products/index
         │
         ▼ (next(context) called)
         │
  UseStaticFiles → UseRouting → ... → Controller runs
         │
         ▼ (returns)
         │
  [OUT] 200
```

---

## 🔷 app.Use vs app.Run vs app.Map

```csharp
// app.Use — runs and PASSES to next middleware
app.Use(async (context, next) =>
{
    // do something
    await next(context);  // ← passes forward
});

// app.Run — TERMINAL middleware. Never calls next. Ends the pipeline.
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello World");
    // ← nothing after this runs
});

// app.Map — branches pipeline for specific URL prefix
app.Map("/api", apiApp =>
{
    apiApp.Run(async context =>
    {
        await context.Response.WriteAsync("API branch");
    });
});
```

---

## 🔷 Custom Middleware — Real Example

```csharp
// Custom middleware as a class — logs every API call
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next,
        ILogger<RequestLoggingMiddleware> logger)
    {
        _next   = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // ON THE WAY IN:
        var start = DateTime.UtcNow;
        _logger.LogInformation(
            "[REQ] {Method} {Path}",
            context.Request.Method,
            context.Request.Path);

        await _next(context);   // ← hand off to next middleware

        // ON THE WAY BACK:
        var elapsed = DateTime.UtcNow - start;
        _logger.LogInformation(
            "[RES] {Status} in {Ms}ms",
            context.Response.StatusCode,
            elapsed.TotalMilliseconds);
    }
}

// Register in Program.cs — BEFORE the middleware you want to wrap:
app.UseMiddleware<RequestLoggingMiddleware>();
app.UseRouting();
// ...
```

---

## 🔷 Middleware Order — The Rules

```
RULE 1: Exception handling MUST be first
        → so it can catch errors from ALL other middleware

RULE 2: UseAuthentication MUST come before UseAuthorization
        → you must identify BEFORE you can check permission

RULE 3: UseRouting MUST come before UseAuthorization
        → authorization needs to know which endpoint is being hit

RULE 4: UseStaticFiles can come early (before routing)
        → for performance: skip auth/routing for static files

CORRECT:                         WRONG (breaks auth):
──────────────────────────       ──────────────────────────
UseExceptionHandler              UseAuthentication
UseHttpsRedirection              UseAuthorization          ← no UseRouting yet
UseStaticFiles                   UseRouting                ← too late
UseRouting                       UseStaticFiles
UseAuthentication                UseExceptionHandler
UseAuthorization                 (authorization doesn't know endpoint)
MapControllerRoute
```

---

## 🔷 Full Request Lifecycle — Your Stack

```
Browser / Kendo Grid / $.ajax call
         │
         │  GET /api/products?skip=0&take=10
         ▼
Kestrel reads HTTP bytes
         │
UseExceptionHandler  (wraps everything in try-catch)
         │
UseHttpsRedirection  (HTTP → HTTPS)
         │
UseStaticFiles       (static file? serve it. else pass through)
         │
UseRouting           (match URL to ProductsController.GetAll)
         │
UseAuthentication    (read JWT → set context.User = "John, Role=Admin")
         │
UseAuthorization     ([Authorize] attribute check → passed)
         │
ProductsController.GetAll()
   ↓
   ADO.NET → SqlConnection → SqlCommand → SqlDataReader
   ↓
   List<Product> built
   ↓
   return Ok(list);  → serialized to JSON
         │
Response travels back up the chain
         │
Kestrel sends HTTP response bytes
         │
         ▼
Kendo DataSource receives JSON
Grid renders rows
```

---

## ⭐ Interview Quick-Fire

| Question                                                    | Answer                                                                                                    |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| What is the ASP.NET Core pipeline?                          | Ordered chain of middleware that every HTTP request passes through before reaching your controller        |
| What is middleware?                                         | A component that inspects/modifies the request or response and either passes it forward or short-circuits |
| What does `next()`do in middleware?                       | Calls the next middleware in the chain                                                                    |
| What is short-circuiting?                                   | When a middleware returns a response immediately without calling `next()`— pipeline stops              |
| Must `UseAuthentication`come before `UseAuthorization`? | ✅ Yes — you must identify first, then check permission                                                  |
| What middleware serves CSS/JS/images?                       | `UseStaticFiles()`— short-circuits before auth for performance                                         |
| Where do you put exception handling middleware?             | FIRST — so it wraps the entire pipeline in a try-catch                                                   |
| What is the difference between `app.Use`and `app.Run`?  | `Use`calls next middleware.`Run`is terminal — ends the pipeline.                                     |
| Can middleware modify the response on the way back?         | ✅ Yes — code after `await next(context)`runs during response                                          |
