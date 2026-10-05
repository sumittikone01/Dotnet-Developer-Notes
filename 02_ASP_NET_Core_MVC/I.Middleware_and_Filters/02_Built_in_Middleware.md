# 02_Built_in_Middleware

> A tour of the middleware components ASP.NET Core ships with out of the box — the ones you registered (in the correct order!) back in `01_Middleware_Concepts.md`'s pipeline example, now explained individually.

## 📌 What is it?

ASP.NET Core provides a set of ready-made middleware for the most common cross-cutting concerns, so you rarely need to write these from scratch (`03_Custom_Middleware.md` is for when you DO need something custom).

## 📊 Built-in Middleware Reference

| Middleware               | Method                                          | Purpose                                                                                   |
| ------------------------ | ----------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Exception Handling       | `UseExceptionHandler(path)`                   | Catches unhandled exceptions app-wide, redirects to a friendly error page/handler         |
| Developer Exception Page | `UseDeveloperExceptionPage()`                 | Shows detailed stack traces — DEVELOPMENT environment only                               |
| HTTPS Redirection        | `UseHttpsRedirection()`                       | Redirects HTTP requests to HTTPS automatically                                            |
| HSTS                     | `UseHsts()`                                   | Tells browsers to ALWAYS use HTTPS for this site (production only)                        |
| Static Files             | `UseStaticFiles()`                            | Serves files from`wwwroot` (CSS, JS, images) directly, bypassing MVC                    |
| Routing                  | `UseRouting()`                                | Matches the incoming URL to an endpoint (Controller/Action) — but doesn't execute it yet |
| CORS                     | `UseCors(policyName)`                         | Applies Cross-Origin Resource Sharing rules (full detail in`L.Security/10_CORS.md`)     |
| Authentication           | `UseAuthentication()`                         | Identifies WHO the caller is (reads cookies/JWT tokens, populates`HttpContext.User`)    |
| Authorization            | `UseAuthorization()`                          | Decides WHAT the identified caller is allowed to do (full detail in`L.Security`)        |
| Session                  | `UseSession()`                                | Enables session state (`M.State_Management/02_Sessions.md`)                             |
| Response Compression     | `UseResponseCompression()`                    | Compresses responses (gzip/brotli) to reduce payload size                                 |
| Endpoints                | `MapControllers()` / `MapControllerRoute()` | The FINAL step — actually executes the matched Controller action                         |

## 🌍 Real-world analogy

Think of these as **standard-issue equipment at each airport checkpoint** rather than something the airport had to invent itself — a metal detector (Static Files: quick, mechanical, no real decision-making) is very different from an immigration officer checking your passport (Authentication: identifies who you are) versus a customs officer deciding if you're allowed to bring something in (Authorization: decides what you can do).

## ⚙️ Internal working — a closer look at three commonly misunderstood ones

### `UseRouting()` vs `UseEndpoints()`/`MapControllers()` — matching vs executing

```
UseRouting():
   Looks at the incoming URL ("/api/products/5") and figures out
   WHICH Controller/Action it corresponds to — but does NOT run it yet.
   This is why Authentication/Authorization can run AFTER routing but
   BEFORE the actual action executes — they can inspect WHICH endpoint
   was matched and make decisions before it actually runs.

MapControllers() (implicit "UseEndpoints"):
   ACTUALLY invokes the matched Controller action.
```

### `UseStaticFiles()` — why it short-circuits

```csharp
app.UseStaticFiles(); // if this request is for e.g. "/css/site.css"...

// ...UseStaticFiles() finds the file in wwwroot, writes it DIRECTLY to the response,
// and does NOT call the next middleware — the request NEVER reaches Routing,
// Authentication, or any Controller at all.
```

### `UseExceptionHandler()` — only catches what happens AFTER it

```
app.UseExceptionHandler("/error");   // registered 1st
app.UseAuthentication();              // if THIS throws, UseExceptionHandler catches it (runs "after" in the wrapping sense)
app.MapControllers();                 // if a Controller throws, UseExceptionHandler catches it too

// If UseExceptionHandler were registered LAST instead, exceptions from
// Authentication/Routing/etc. registered BEFORE it would NEVER be caught by it.
```

## 💻 Code examples

### Basic — a complete, correctly-ordered built-in middleware pipeline

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();      // detailed errors — DEV ONLY
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();                     // serves wwwroot content, short-circuits for matches

app.UseRouting();                         // determines WHICH endpoint matches the URL

app.UseCors("AllowSpecificOrigin");       // CORS must be between Routing and Auth/Endpoints

app.UseAuthentication();                  // WHO is calling?
app.UseAuthorization();                   // WHAT are they allowed to do?

app.UseSession();                         // if using session state

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Intermediate — enabling response compression for a Kendo-Grid-heavy app

```csharp
// Program.cs — service registration
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true;
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
});

var app = builder.Build();
app.UseResponseCompression(); // register EARLY, before content is generated
// ... rest of pipeline
```

### Practical — serving a SPECIFIC static folder outside wwwroot

```csharp
app.UseStaticFiles(); // default: serves from wwwroot

app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(builder.Environment.ContentRootPath, "GeneratedReports")),
    RequestPath = "/reports" // accessible at /reports/filename.pdf
});
```

## ⚡ Performance considerations

- `UseStaticFiles()` should be registered EARLY — the earlier a static file request short-circuits, the less unnecessary work (routing, auth) is done for content that needs none of it.
- `UseResponseCompression()` trades a small amount of CPU (compressing the response) for significantly reduced payload size over the network — usually a clear net win, especially for larger JSON payloads (Kendo Grid data) or text-heavy responses.
- `UseDeveloperExceptionPage()` must NEVER be used in production — it exposes detailed stack traces and potentially sensitive internal information to end users; always gate it behind `IsDevelopment()`.

## 🚨 Common mistakes

- ❌ Leaving `UseDeveloperExceptionPage()` active in production — a real security/information-disclosure risk.
- ❌ Placing `UseCors()` in the wrong position relative to `UseRouting()`/`UseAuthorization()` — CORS middleware has a specific required position (after Routing, before Authorization) to work correctly.
- ❌ Registering `UseStaticFiles()` AFTER `UseRouting()`/`MapControllers()` — static file requests get routed through MVC unnecessarily, potentially causing 404s if no matching controller route exists.
- ❌ Forgetting `UseHttpsRedirection()` — leaves the app silently accepting insecure HTTP traffic.

## 💡 Best practices

- ✅ Use the well-established standard order (shown in the Practical example above) as your default template for every new ASP.NET Core MVC project.
- ✅ Gate `UseDeveloperExceptionPage()` strictly behind `env.IsDevelopment()`.
- ✅ Enable `UseResponseCompression()` for apps serving significant JSON/text payloads (very relevant for a Kendo-Grid-heavy application).
- ✅ Register `UseStaticFiles()` early, before routing, so static content is served as cheaply and quickly as possible.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                          | Answer                                                                                                                                                       |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| What's the difference between`UseRouting()` and `MapControllers()`?           | `UseRouting()` matches the URL to an endpoint but doesn't run it; `MapControllers()` (endpoint execution) actually invokes the matched Controller action |
| Why should`UseStaticFiles()` be registered early in the pipeline?               | So static file requests short-circuit immediately, without unnecessarily going through routing, authentication, or authorization                             |
| What does`UseHsts()` do, and where should it be used?                           | Tells browsers to always use HTTPS for the site going forward; used in production, not typically needed in development                                       |
| Why is it dangerous to leave`UseDeveloperExceptionPage()` active in production? | It exposes detailed stack traces and internal application details to end users — a security/information-disclosure risk                                     |
| What does`UseAuthentication()` do, as distinct from `UseAuthorization()`?     | Authentication identifies WHO the caller is (populates`HttpContext.User`); Authorization decides WHAT that identified caller is allowed to do              |

## 📝 30-second Revision Cheat Sheet

- Built-in middleware covers the standard concerns: Exception Handling, HTTPS Redirect, Static Files, Routing, CORS, Authentication, Authorization, Session, Compression, Endpoints.
- `UseRouting()` MATCHES the endpoint; `MapControllers()` EXECUTES it — two separate steps.
- `UseStaticFiles()` should be early — lets static content short-circuit cheaply.
- `UseDeveloperExceptionPage()` = development ONLY; `UseExceptionHandler()` = production.
- Standard order: Exception Handling → HTTPS → Static Files → Routing → CORS → Authentication → Authorization → Endpoints.
