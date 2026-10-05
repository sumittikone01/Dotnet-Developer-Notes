# 02_Cookie_Authentication

> **Cookie Authentication** = the traditional, browser-native authentication scheme where the server issues an encrypted cookie after login, and the browser automatically attaches it to every subsequent request — the natural fit for your team's Razor-View-rendered MVC pages.

> Continues **L.Security**, drilling into the first scheme previewed in `01_Authentication_Overview.md`'s comparison table.

## 📌 What is it?

```
1. User submits login form (username/password)
2. Server verifies credentials against the database
3. Server creates a ClaimsPrincipal (the user's identity + claims)
4. Server calls HttpContext.SignInAsync(...) → issues an ENCRYPTED cookie
5. Browser stores the cookie, sends it AUTOMATICALLY on every future request
6. Server decrypts/validates the cookie each request → populates HttpContext.User
```

## 🤔 Why do we need it, specifically for a Razor-View MVC app?

| Need                                                                          | How Cookie Authentication helps                                                               |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| A browser-rendered app needs to "remember" a logged-in user across page loads | The cookie persists across requests automatically — no manual token management needed        |
| Simplicity for a traditional server-rendered app                              | The browser handles attaching the credential — no client-side JavaScript needed to manage it |
| Built-in logout/session-expiry support                                        | `SignOutAsync()` clears the cookie; expiration is configurable                              |

## 🌍 Real-world analogy

A **wristband at an all-day festival**. Once you check in (log in) and get the wristband (cookie), security staff at every subsequent gate (page/request) can just LOOK at your wrist — you don't need to show your ID again at every single gate. The wristband itself proves you already checked in.

## ⚙️ Internal working — signing in and reading the resulting identity

```csharp
public async Task<IActionResult> Login(LoginViewModel model)
{
    var user = _userService.ValidateCredentials(model.Username, model.Password); // checks hashed password
    if (user == null)
    {
        ModelState.AddModelError("", "Invalid username or password.");
        return View(model);
    }

    // Build the identity: a set of CLAIMS describing who this user is
    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.Username),
        new Claim(ClaimTypes.Role, user.Role) // used later by Role-Based Authorization (07_Role_Based_Authorization.md)
    };

    var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
    var principal = new ClaimsPrincipal(identity);

    // THIS line actually issues the encrypted cookie to the browser
    await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);

    return RedirectToAction("Index", "Home");
}
```

```
┌─────────────────────────────────────────────────────────────┐
│  SignInAsync(scheme, principal)                                │
│         │                                                      │
│         ▼                                                      │
│  Serializes the ClaimsPrincipal, ENCRYPTS it                    │
│         │                                                      │
│         ▼                                                      │
│  Sets an HTTP "Set-Cookie" response header                       │
│         │                                                      │
│         ▼                                                      │
│  Browser stores the cookie, sends it automatically on EVERY     │
│  future request to this domain                                  │
└─────────────────────────────────────────────────────────────┘
```

## 📊 Key Cookie Authentication Options

| Option                  | Purpose                                                                                                           |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `LoginPath`           | Where unauthenticated users are redirected (e.g.,`/Account/Login`)                                              |
| `AccessDeniedPath`    | Where authenticated-but-unauthorized users are redirected                                                         |
| `ExpireTimeSpan`      | How long the cookie stays valid                                                                                   |
| `SlidingExpiration`   | If`true`, the cookie's expiration resets on activity (like `01_Cache_Basics.md`'s sliding expiration concept) |
| `Cookie.HttpOnly`     | Prevents JavaScript from reading the cookie — critical XSS protection (default:`true`)                         |
| `Cookie.SecurePolicy` | Ensures the cookie is only sent over HTTPS                                                                        |
| `Cookie.SameSite`     | Controls cross-site cookie behavior — CSRF-relevant (`13_Anti_Forgery_Tokens_CSRF.md`)                         |

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.LoginPath = "/Account/Login";
        options.AccessDeniedPath = "/Account/AccessDenied";
        options.ExpireTimeSpan = TimeSpan.FromHours(8);
        options.SlidingExpiration = true;
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.SameSite = SameSiteMode.Lax;
    });
```

## 💻 Code examples

### Basic — a complete login/logout flow

```csharp
[HttpGet]
public IActionResult Login() => View();

[HttpPost]
public async Task<IActionResult> Login(LoginViewModel model)
{
    if (!ModelState.IsValid) return View(model);

    var user = _userService.ValidateCredentials(model.Username, model.Password);
    if (user == null)
    {
        ModelState.AddModelError("", "Invalid credentials.");
        return View(model);
    }

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.Username)
    };
    var principal = new ClaimsPrincipal(new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme));
    await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, principal);

    return RedirectToAction("Index", "Home");
}

[HttpPost]
public async Task<IActionResult> Logout()
{
    await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme); // clears the cookie
    return RedirectToAction("Login");
}
```

### Intermediate — protecting a Controller/Action, and reading the identity

```csharp
[Authorize] // requires a valid authentication cookie — redirects to LoginPath otherwise
public class OrdersController : Controller
{
    public IActionResult MyOrders()
    {
        string userId = User.FindFirst(ClaimTypes.NameIdentifier)!.Value;
        var orders = _service.GetOrdersForUser(int.Parse(userId));
        return View(orders);
    }
}
```

### Practical — checking authentication status in a Razor View

```html
@* _Layout.cshtml *@
@if (User.Identity?.IsAuthenticated == true)
{
    <span>Welcome, @User.Identity.Name!</span>
    <form asp-controller="Account" asp-action="Logout" method="post">
        <button type="submit">Logout</button>
    </form>
}
else
{
    <a asp-controller="Account" asp-action="Login">Login</a>
}
```

## ⚡ Performance considerations

- Decrypting/validating a cookie on every request is fast — no database round trip required for the basic identity check itself (though a request might still query the DB for OTHER reasons).
- `SlidingExpiration` adds a small write cost (re-issuing a refreshed cookie periodically) — negligible for typical traffic levels.
- Cookies ARE sent with every single request to the domain automatically (including for static assets like images/CSS unless scoped carefully) — slightly increases request size, generally not a meaningful concern for typical business apps.

## 🚨 Common mistakes

- ❌ Setting `Cookie.HttpOnly = false` — makes the cookie readable by JavaScript, a serious XSS vulnerability (recap from `14_XSS_and_SQL_Injection_Prevention.md`).
- ❌ Forgetting `Cookie.SecurePolicy = CookieSecurePolicy.Always` in production — allows the cookie to be sent over plain HTTP, risking interception.
- ❌ Not calling `SignOutAsync()` on logout — leaves a valid cookie active even after the user believes they've logged out.
- ❌ Storing SENSITIVE data directly inside the cookie's claims beyond what's needed for identity/authorization — keep the cookie payload minimal.

## 💡 Best practices

- ✅ Always set `HttpOnly = true` and `SecurePolicy = CookieSecurePolicy.Always` (in production) for authentication cookies.
- ✅ Use `SlidingExpiration` for typical web apps, so active users aren't logged out mid-session, while inactive sessions still eventually expire.
- ✅ Keep claims minimal — just enough to identify the user and support authorization checks (user ID, username, role) — not a full profile dump.
- ✅ Pair Cookie Authentication with anti-forgery tokens (`13_Anti_Forgery_Tokens_CSRF.md`) for any state-changing POST/PUT/DELETE requests, since cookies are automatically sent and therefore CSRF-exposed.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                   | Answer                                                                                                                                                                  |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What method actually issues the authentication cookie?                     | `HttpContext.SignInAsync(scheme, principal)`                                                                                                                          |
| What does`Cookie.HttpOnly = true` protect against?                       | Prevents client-side JavaScript from reading the cookie, mitigating XSS-based credential theft                                                                          |
| What's the difference between`ExpireTimeSpan` and `SlidingExpiration`? | `ExpireTimeSpan` sets how long the cookie lasts; `SlidingExpiration` resets that countdown on each active request, rather than using a fixed expiry from login time |
| Why is Cookie Authentication particularly exposed to CSRF?                 | Because the browser sends the cookie AUTOMATICALLY with every request to the domain, including ones a malicious site could trick the browser into making                |
| How do you log a user out?                                                 | Call`HttpContext.SignOutAsync(scheme)`, which clears the authentication cookie                                                                                        |

## 📝 30-second Revision Cheat Sheet

- Cookie Authentication issues an encrypted cookie via `SignInAsync()`; the browser sends it automatically on every request.
- Key options: `LoginPath`, `ExpireTimeSpan`, `SlidingExpiration`, `Cookie.HttpOnly`, `Cookie.SecurePolicy`, `Cookie.SameSite`.
- `HttpOnly = true` + `SecurePolicy = Always` are non-negotiable security defaults for production.
- Natural fit for Razor-View-rendered MVC apps — the browser handles attaching the credential automatically.
- Pair with anti-forgery tokens for state-changing requests, since automatic cookie attachment creates CSRF exposure.
