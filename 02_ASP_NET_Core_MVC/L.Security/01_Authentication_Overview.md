# 01_Authentication_Overview

> **Authentication** = proving WHO a caller is. It answers "who are you?" — distinct from **Authorization** (`06_Authorization_Overview.md`), which answers "what are you allowed to do?" This is the single most important distinction in the entire Security chapter.

> New chapter: **L.Security** — the largest in your roadmap (14 files). Everything from here builds on this foundational WHO vs WHAT distinction.

## 📌 What is it?

```
Authentication: "Who are you?"           Authorization: "What can you do?"
       │                                         │
       ▼                                         ▼
 Login with username/password          Is THIS authenticated user
 → server verifies identity              allowed to DELETE this product?
 → issues a cookie/token                → checked AFTER authentication,
   proving "this IS Sumit"                using roles/claims/policies
```

`UseAuthentication()` (recall `I.Middleware_and_Filters/02_Built_in_Middleware.md`) populates `HttpContext.User` with the caller's identity. `UseAuthorization()` then uses THAT identity to make yes/no access decisions.

## 🤔 Why do we need to understand the distinction so precisely?

| Confusing the two                                                                                                              | Understanding them separately                                                                                          |
| ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------- |
| "Authentication failed" errors get treated the same as "Authorization failed" — confusing for debugging and for API consumers | 401 (not authenticated) vs 403 (authenticated, not allowed) are clearly different problems, requiring different fixes  |
| Hard to reason about WHERE a security check belongs                                                                            | Authentication = "who is this," checked once per request; Authorization = "can they do X," checked per-resource/action |
| Can't design a system with multiple AUTH schemes (cookies for the website, JWT for the API) cleanly                            | Understanding authentication as a pluggable SCHEME (covered below) makes this natural                                  |

## 🌍 Real-world analogy

**Authentication** is like showing your ID at a building's front desk — it proves you ARE who your ID says you are. **Authorization** is like a separate check at EACH specific door inside the building — having a valid ID (being authenticated) doesn't automatically mean every door unlocks for you; each door has its OWN rule about who's allowed through.

## 📊 Authentication SCHEMES — the pluggable mechanism

ASP.NET Core supports multiple, independently-configurable authentication SCHEMES — each a different WAY of proving identity.

| Scheme                              | How identity is proven                                                    | Typical use                                        | Detailed in                                                           |
| ----------------------------------- | ------------------------------------------------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------- |
| **Cookie Authentication**     | An encrypted cookie, set after login, sent automatically on every request | Traditional server-rendered web apps (Razor Views) | `02_Cookie_Authentication.md`                                       |
| **JWT Bearer Authentication** | A signed token, sent explicitly in the`Authorization` header            | APIs, SPAs, mobile apps                            | `03_JWT_Authentication.md`, `04_JWT_Generation_and_Validation.md` |
| **OAuth / External Login**    | Delegating identity verification to Google, Microsoft, etc.               | "Sign in with Google" style flows                  | `05_OAuth_and_External_Login.md`                                    |

> A SINGLE application can register MULTIPLE schemes simultaneously — e.g., Cookie auth for the Razor-View-rendered pages, JWT auth for the AJAX-called Web API endpoints that your Kendo Grid hits.

## ⚙️ Internal working — the authentication flow, end to end

```
1. User submits credentials (login form, or an API login endpoint)
        │
        ▼
2. Server VERIFIES credentials against the database (usually via a hashed password check — 11_Password_Hashing_BCrypt_vs_Identity.md)
        │
        ▼
3. If valid, server ISSUES a credential:
     - Cookie scheme → sets an encrypted authentication cookie
     - JWT scheme     → returns a signed token in the response body
        │
        ▼
4. On SUBSEQUENT requests, the caller presents that credential:
     - Cookie → sent AUTOMATICALLY by the browser with every request
     - JWT     → the CLIENT must explicitly attach it: Authorization: Bearer <token>
        │
        ▼
5. UseAuthentication() middleware reads the credential, VALIDATES it,
   and populates HttpContext.User with the identified user's CLAIMS
        │
        ▼
6. Later code (Controllers, [Authorize], UseAuthorization()) can now ask:
   "who is User?" and "what claims/roles does User have?"
```

## 📊 Cookie vs JWT — the Core Trade-off (previewed here, fully detailed in `02` and `03`)

| Aspect                         | Cookie Authentication                                                             | JWT Authentication                                                               |
| ------------------------------ | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Storage                        | Server-managed session state OR self-contained encrypted cookie                   | Self-contained token, no server-side session needed                              |
| Sent automatically by browser? | Yes                                                                               | No — client must explicitly attach it to each request                           |
| Best fit                       | Server-rendered web apps (your Razor View pages)                                  | APIs, mobile apps, SPAs — especially across DIFFERENT domains                   |
| CSRF risk                      | Higher — needs explicit anti-forgery tokens (`13_Anti_Forgery_Tokens_CSRF.md`) | Lower — not automatically sent, so CSRF is less of a direct concern             |
| Revocation                     | Easier — server can invalidate server-side session                               | Harder — a signed token is valid until it expires, unless you build a blocklist |

## 💻 Code examples

### Basic — registering an authentication scheme (the setup every later chapter builds on)

```csharp
// Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/AccessDenied";
    });

var app = builder.Build();

app.UseAuthentication(); // populates HttpContext.User
app.UseAuthorization();   // uses HttpContext.User to make access decisions
```

### Intermediate — registering BOTH Cookie (for Views) and JWT (for the API) simultaneously

```csharp
builder.Services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme; // default for Razor pages
})
.AddCookie()
.AddJwtBearer(options => // used explicitly by API endpoints via [Authorize(AuthenticationSchemes = "Bearer")]
{
    options.TokenValidationParameters = new TokenValidationParameters { /* ... see 04_JWT_Generation_and_Validation.md */ };
});
```

```csharp
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)] // this endpoint REQUIRES JWT specifically
[Route("api/products")]
public class ProductsApiController : ControllerBase { ... }
```

### Practical — reading the authenticated identity in a Controller/BAL

```csharp
[Authorize]
[HttpGet("my-orders")]
public IActionResult GetMyOrders()
{
    // HttpContext.User was populated by UseAuthentication() — this is the result of that work
    string userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value
        ?? throw new InvalidOperationException("User identity missing.");

    return Ok(_service.GetOrdersForUser(userId));
}
```

## ⚡ Performance considerations

- Cookie authentication validation is typically very fast — decrypting/validating a cookie is cheap, no database round trip needed for basic identity checks.
- JWT validation (signature check) is also fast and doesn't require a database round trip — this is part of why JWTs scale well across multiple stateless API servers (ties back to `12_System_Design/I.API_Design/01_REST_API_Principles.md`'s statelessness discussion).

## 🚨 Common mistakes

- ❌ Confusing a 401 (not authenticated) error with a 403 (authenticated, not authorized) and debugging the wrong layer entirely.
- ❌ Assuming ONE authentication scheme fits every part of an app — a Razor-View-heavy MVC app with an attached Web API very often needs BOTH Cookie and JWT schemes side by side.
- ❌ Treating "the user is logged in" (authentication) as equivalent to "the user can do this specific thing" (authorization) — they're separate questions requiring separate checks.
- ❌ Forgetting `UseAuthentication()` must be registered BEFORE `UseAuthorization()` in the middleware pipeline (recap from `I.Middleware_and_Filters/01_Middleware_Concepts.md`) — authorization decisions depend on authentication having already run.

## 💡 Best practices

- ✅ Always think "who is this?" (authentication) BEFORE "what can they do?" (authorization) — keep the two concerns conceptually and architecturally separate.
- ✅ Choose the authentication scheme that fits the CALLER: Cookie for browser-rendered pages, JWT for APIs/SPAs/mobile.
- ✅ Register multiple schemes side by side when a single app genuinely serves both use cases (common in your team's MVC + Web API hybrid setup).
- ✅ Never roll your own authentication/token-signing logic from scratch — use ASP.NET Core's built-in authentication handlers and established libraries.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                 | Answer                                                                                                                            |
| ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| What's the fundamental difference between Authentication and Authorization?              | Authentication answers "who are you?"; Authorization answers "what are you allowed to do?" — authentication always happens first |
| What does`UseAuthentication()` actually do?                                            | Reads the caller's credential (cookie/token) and populates`HttpContext.User` with their identity and claims                     |
| Can an ASP.NET Core app support multiple authentication schemes at once?                 | Yes — e.g., Cookie authentication for Razor View pages and JWT Bearer authentication for API endpoints, registered side by side  |
| Why must`UseAuthentication()` be registered before `UseAuthorization()`?             | Authorization decisions depend on knowing WHO the caller is, which authentication establishes                                     |
| What HTTP status code indicates a failed AUTHENTICATION vs a failed AUTHORIZATION check? | 401 Unauthorized = authentication failure (not logged in); 403 Forbidden = authorization failure (logged in, but not allowed)     |

## 📝 30-second Revision Cheat Sheet

- Authentication = "who are you?" (identity); Authorization = "what can you do?" (permissions) — always in that order.
- `UseAuthentication()` populates `HttpContext.User`; `UseAuthorization()` uses it to make access decisions.
- Schemes are pluggable: Cookie (browser apps), JWT (APIs/SPAs/mobile), OAuth (external login) — can coexist in one app.
- 401 = not authenticated; 403 = authenticated but not authorized.
- Middleware order matters: `UseAuthentication()` must come before `UseAuthorization()`.
