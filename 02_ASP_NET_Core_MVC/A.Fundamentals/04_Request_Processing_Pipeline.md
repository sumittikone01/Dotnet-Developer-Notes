# 04 — Request Processing Pipeline

## 📌 What is it?

The **request processing pipeline** is the ordered sequence of **middleware components** that every HTTP request passes through — from the moment it hits Kestrel to the moment a response is sent back to the client.

## 🤔 Why do we need it?

Every web app needs cross-cutting behavior applied to *every* (or *most*) request: logging, authentication, error handling, static file serving, routing. Instead of repeating this logic in every Controller, ASP.NET Core lets you configure it **once**, centrally, as a pipeline.

## 🧠 Intuition

Think of the pipeline as an **airport security line**: every passenger (request) passes through the same sequence of checkpoints (middleware) — ID check, bag scan, metal detector — before reaching the gate (your Controller/endpoint). Each checkpoint can:

- Let the passenger through to the next checkpoint
- Stop them entirely (short-circuit — e.g., reject unauthenticated requests)
- Do something on the way *back* too (e.g., logging the response status)

## 🖼 ASCII diagram — the pipeline

```
Request
   │
   ▼
┌─────────────────────┐
│ Exception Handling    │  ← catches errors from everything downstream
├─────────────────────┤
│ HTTPS Redirection     │
├─────────────────────┤
│ Static Files           │  ← serves wwwroot files, short-circuits if matched
├─────────────────────┤
│ Routing                │  ← determines which endpoint matches the URL
├─────────────────────┤
│ Authentication         │  ← who are you?
├─────────────────────┤
│ Authorization           │  ← are you allowed?
├─────────────────────┤
│ Custom Middleware       │  ← your own cross-cutting logic
├─────────────────────┤
│ Endpoint (Controller)   │  ← your actual code runs here
└─────────────────────┘
   │
   ▼
Response (flows back UP through the same middleware, in reverse)
```

**Key insight:** middleware order matters, and each middleware wraps around the ones after it — like nested layers of an onion. A request goes "in" through each layer, and the response comes back "out" through the same layers in reverse.

## 💻 Code example (Program.cs — minimal hosting model)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services (DI container setup) — happens BEFORE the pipeline
builder.Services.AddControllersWithViews();

var app = builder.Build();

// ↓↓↓ This is the actual pipeline — ORDER MATTERS ↓↓↓
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error"); // catch unhandled exceptions
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();       // must come before routing for static files to work

app.UseRouting();           // determines endpoint

app.UseAuthentication();    // must come AFTER UseRouting, BEFORE UseAuthorization
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## 📊 Why order matters — real consequences of getting it wrong

| Mistake                                                | Consequence                                                                             |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------- |
| `UseAuthorization()` before `UseAuthentication()`  | Authorization checks run before the user's identity is even established — always fails |
| `UseStaticFiles()` after `UseRouting()`            | Static file requests may get routed to a Controller instead of served directly          |
| `UseExceptionHandler()` placed too late              | Errors thrown by earlier middleware won't be caught                                     |
| Custom logging middleware placed after`UseRouting()` | Won't capture routing-related failures                                                  |

## ⚙️ Short-circuiting

A middleware can choose **not to call the next one**, effectively stopping the pipeline early:

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401;
        await context.Response.WriteAsync("Missing API key");
        return; // short-circuit — 'next' is never called
    }
    await next(); // continue to the next middleware
});
```

This is exactly how `UseStaticFiles()` works internally — if it finds a matching file, it serves it and **stops** the pipeline right there, never reaching your Controllers.

## 🚨 Common mistakes

- Assuming middleware order doesn't matter — it absolutely does; it's a linear, sequential pipeline.
- Forgetting to call `await next()` in custom middleware, accidentally short-circuiting every request.
- Placing custom exception-handling middleware too far down the pipeline, so it misses errors from earlier stages.

## 💡 Best practices

- Standard recommended order: **Exception Handling → HTTPS Redirection → Static Files → Routing → Authentication → Authorization → Custom middleware → Endpoints**.
- Keep custom middleware focused — one responsibility per middleware (logging, API key check, etc.), don't create a "god middleware."
- Use `app.UseWhen()` or `app.MapWhen()` when middleware should only apply conditionally (e.g., only for `/api` routes).

## 🎤 Interview questions

1. Why does middleware order matter in ASP.NET Core? Give a concrete example of a bug caused by wrong ordering.
2. What does "short-circuiting" mean in the middleware pipeline, and how would you implement it?
3. Why must `UseAuthentication()` come before `UseAuthorization()`?
4. How does the pipeline handle the *response*, not just the request?

## 📝 30-second revision cheat sheet

- Pipeline = ordered chain of middleware, each request passes through in sequence.
- Order matters — like an onion, request goes in through layers, response comes back out through the same layers reversed.
- Standard order: Exception Handling → HTTPS → Static Files → Routing → AuthN → AuthZ → Endpoints.
- Middleware can **short-circuit** by not calling `next()`.
