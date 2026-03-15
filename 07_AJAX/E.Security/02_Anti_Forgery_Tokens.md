
# 02 — Anti-Forgery Tokens

---

## 🎯 One-Line Definition

> **An anti-forgery token is a unique secret value embedded in your page — your AJAX calls must send it back with every POST, and ASP.NET rejects any POST that doesn't include it.**

---

## 🔑 The Full Picture — How It All Connects

```
SERVER                                        BROWSER
──────                                        ───────

Page renders                                  Page loads
Token generated: "xK9m2..."
Embedded in hidden field
                    ──── HTML + token ────►
                                              Hidden field in DOM:
                                              <input type="hidden"
                                               name="__RequestVerificationToken"
                                               value="xK9m2..." />

                                              User clicks Save
                                              JS reads token: "xK9m2..."
                                              Adds to AJAX request header
                    ◄─── POST + token ────
Token in header: "xK9m2..."
Compare with session token
Match ✅ → process request
No match ❌ → reject with 400
```

---

## 🔑 Step 1 — Generate the Token in the View

```cshtml
@* Option A: using Html helper — adds a hidden input field *@
@Html.AntiForgeryToken()
@* Generates: <input type="hidden" name="__RequestVerificationToken" value="xK9m2pL..." /> *@


@* Option B: using Tag Helper on a form — added automatically *@
<form asp-action="Create" method="post">
    @* Token hidden field is injected here automatically *@
    <input asp-for="Name" />
    <button type="submit">Save</button>
</form>


@* Option C: for AJAX-only pages (no form) — add just the token *@
<div>
    @Html.AntiForgeryToken()
    @* Now JavaScript can find it with: *@
    @* $('input[name="__RequestVerificationToken"]').val() *@
</div>
```

---

## 🔑 Step 2 — Send the Token from JavaScript

```javascript
// ── Method A: In individual $.ajax call ───────────────────────
$.ajax({
    url:  "/Employee/Delete",
    type: "POST",
    headers: {
        "RequestVerificationToken":
            $('input[name="__RequestVerificationToken"]').val()
    },
    data: { id: empId },
    success: function(r) { removeRow(empId); }
});


// ── Method B: In $.ajaxSetup — applies to ALL calls ───────────
// Put this once in your layout or shared JS file

$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});

// Now every $.post and $.ajax automatically sends the token:
$.post("/Employee/Create", formData, function(r) { ... });  // ✅ token included
$.post("/Employee/Delete", { id: 5 }, function(r) { ... }); // ✅ token included


// ── Method C: As a form field (alternative to header) ────────
// Some older setups pass token as form data instead of header
$.post("/Employee/Create", {
    name:                       "Alice",
    __RequestVerificationToken: $('input[name="__RequestVerificationToken"]').val()
}, function(r) { ... });
```

---

## 🔑 Step 3 — Validate the Token in the Controller

```csharp
// ── Option A: Per-action attribute ───────────────────────────
[HttpPost]
[ValidateAntiForgeryToken]     // validates the token for this action only
public JsonResult Create(Employee employee)
{
    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Json(new { success = true });
}


// ── Option B: Per-controller attribute ───────────────────────
[ValidateAntiForgeryToken]     // all POST actions in this controller validated
public class EmployeeController : Controller
{
    [HttpPost]
    public JsonResult Create(Employee emp) { ... }

    [HttpPost]
    public JsonResult Update(Employee emp) { ... }

    [HttpPost]
    public JsonResult Delete(int id) { ... }
}


// ── Option C: Global for ALL controllers (recommended) ───────
// In Program.cs:
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
    // ↑ Every POST, PUT, PATCH, DELETE is validated automatically
    // No need to add [ValidateAntiForgeryToken] anywhere
});
```

---

## 🔑 Configuring the Token Header Name

ASP.NET Core can be configured to look for the token in a specific header:

```csharp
// Program.cs — configure which header name to look for
builder.Services.AddAntiforgery(options =>
{
    // Name of the hidden field in HTML
    options.FormFieldName = "__RequestVerificationToken";

    // Name of the HTTP header the AJAX call sends it in
    options.HeaderName = "RequestVerificationToken";

    // Cookie name (used internally by ASP.NET)
    options.Cookie.Name = "XSRF-TOKEN";
});
```

```javascript
// JavaScript must send it in the matching header name
xhr.setRequestHeader("RequestVerificationToken", token);
// ↑ Must match options.HeaderName above
```

---

## 🔑 Token Rotation — Each Request Gets a New Token

By default, ASP.NET generates a new anti-forgery token per-request. After a form submits and the page reloads, the new page has a new token.

```javascript
// For Single Page App patterns — if the page doesn't reload,
// get the refreshed token from the server when needed:

function refreshToken() {
    $.get("/Account/GetToken", function(response) {
        // Update the hidden field with the fresh token
        $('input[name="__RequestVerificationToken"]').val(response.token);
    });
}

// Controller to return a fresh token
[HttpGet]
[IgnoreAntiforgeryToken]    // this GET doesn't need token validation
public IActionResult GetToken()
{
    var tokens = _antiforgery.GetAndStoreTokens(HttpContext);
    return Json(new { token = tokens.RequestToken });
}
```

---

## 🔑 Ignoring Validation When Needed

```csharp
// For public API endpoints that don't use cookies (JWT instead)
[HttpPost]
[IgnoreAntiforgeryToken]   // skip validation for this action
public IActionResult WebhookReceiver([FromBody] WebhookPayload payload)
{
    // External service sends to this endpoint — no token possible
    ProcessWebhook(payload);
    return Ok();
}

// For API controllers with JWT auth — entire controller
[ApiController]
[Route("api/[controller]")]
[IgnoreAntiforgeryToken]   // JWT doesn't need CSRF protection
public class EmployeeApiController : ControllerBase
{
    ...
}
```

---

## ⚠️ Common Mistakes

| Mistake                                                                                                                                   | Symptom                                        | Fix                                                                              |
| ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------- |
| No `@Html.AntiForgeryToken()`in view                                                                                                    | Token hidden field missing, AJAX can't find it | Add `@Html.AntiForgeryToken()`to the view                                      |
| Wrong header name                                                                                                                         | 400 Bad Request — token present but not found | Header name must match `options.HeaderName`in `AddAntiforgery()`             |
| `$.ajaxSetup`runs before DOM loads  | Token value is empty string                    | Wrap `$.ajaxSetup`in `$(function() { ... })` |                                                |                                                                                  |
| `[ValidateAntiForgeryToken]`on GET                                                                                                      | GET requests fail                              | Only apply to state-changing methods                                             |
| Token not refreshed in SPA                                                                                                                | 400 after first action (token stale)           | Refresh token after each action or use `AutoValidateAntiforgeryTokenAttribute` |
| Webhook endpoints validating token                                                                                                        | External service gets 400                      | Add `[IgnoreAntiforgeryToken]`to webhook receivers                             |

---

## 💻 Complete Working Example

```cshtml
@* Index.cshtml *@
@Html.AntiForgeryToken()

<kendo-grid name="employeeGrid">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read    url="@Url.Action("Read",    "Employee")" type="POST" />
            <create  url="@Url.Action("Create",  "Employee")" type="POST" />
            <update  url="@Url.Action("Update",  "Employee")" type="POST" />
            <destroy url="@Url.Action("Destroy", "Employee")" type="POST" />
        </transport>
    </datasource>
</kendo-grid>

@section Scripts {
<script>
    $(function() {
        // Global setup — token on every AJAX call
        $.ajaxSetup({
            beforeSend: function(xhr) {
                xhr.setRequestHeader(
                    "RequestVerificationToken",
                    $('input[name="__RequestVerificationToken"]').val()
                );
            }
        });
    });
</script>
}
```

```csharp
// Program.cs
builder.Services.AddControllersWithViews(options => {
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});

builder.Services.AddAntiforgery(options => {
    options.HeaderName = "RequestVerificationToken";
});
```

```csharp
// EmployeeController.cs — NO [ValidateAntiForgeryToken] needed
// because AutoValidateAntiforgeryTokenAttribute handles it globally
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    if (ModelState.IsValid) { _db.Employees.Add(emp); _db.SaveChanges(); }
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
}
```

---

## ❓ Interview Questions

**Q: What is an anti-forgery token?**

> A unique secret value generated by the server and embedded in the page HTML. AJAX requests must include it in a header, and the server compares it against the value stored in the session. A request without a matching token is rejected as a potential CSRF attack.

**Q: Where does the token live on the client side?**

> In a hidden `<input>` field with name `__RequestVerificationToken`, generated by `@Html.AntiForgeryToken()`. JavaScript reads it with `$('input[name="__RequestVerificationToken"]').val()`.

**Q: What is the difference between `[ValidateAntiForgeryToken]` and `AutoValidateAntiforgeryTokenAttribute`?**

> `[ValidateAntiForgeryToken]` must be added to each action or controller manually. `AutoValidateAntiforgeryTokenAttribute` registered globally in `Program.cs` applies to all state-changing requests (POST, PUT, PATCH, DELETE) automatically — nothing to add per-controller.

**Q: Why do API controllers often skip anti-forgery validation?**

> API controllers typically use JWT tokens in the `Authorization` header rather than cookies. JWT-based auth is inherently CSRF-resistant because browsers don't automatically send headers cross-site. Anti-forgery tokens are only needed for cookie-based authentication.
>
