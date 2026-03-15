
# 01 — CSRF Protection

---

## 🎯 One-Line Definition

> **CSRF (Cross-Site Request Forgery) is an attack where a malicious website tricks your browser into making a request to your app — using your logged-in session — without your knowledge.**

---

## 🔑 The Attack — How CSRF Works

To understand the protection, first understand the attack:

```
NORMAL REQUEST (what should happen):
──────────────────────────────────────────────────────────
1. You log into yourbank.com
2. Browser stores your session cookie
3. You click "Transfer $100"
4. Browser sends request to yourbank.com WITH your cookie
5. Bank processes it — you authorised it ✅


CSRF ATTACK (what the attacker does):
──────────────────────────────────────────────────────────
1. You log into yourbank.com
2. Browser stores your session cookie
3. You visit evil.com (in another tab)
4. evil.com silently loads this HTML:
   <img src="https://yourbank.com/Transfer?amount=10000&to=attacker">
5. Your browser sees an <img> tag — fetches the URL automatically
6. The request goes to yourbank.com WITH your cookie (browser adds it)
7. Bank sees: valid session + valid request = processes transfer 💀
8. You lost $10,000. You clicked nothing. You authorised nothing.
```

```
The key insight:
────────────────────────────────────────────────────────────
Browsers AUTOMATICALLY attach cookies to every request
to a domain — even requests triggered by OTHER websites.

The server can't tell the difference between:
  "User clicked a button on MY page"     ← legitimate
  "Evil site tricked browser to call me" ← CSRF attack
```

---

## 🔑 What CSRF Can and Cannot Do

```
CSRF CAN:                              CSRF CANNOT:
──────────────────────────────────     ──────────────────────────────────
Trigger any action the user            Read the response (same-origin
can do while logged in                 policy blocks this)

Transfer money                         Steal your cookies directly

Change email/password                  Read your banking balance

Delete records                         See what happened as a result

Post content on your behalf            Access response data
```

---

## 🔑 Who is Vulnerable?

```
VULNERABLE:
  ✅ Cookie-based authentication (session cookies)
  ✅ Any state-changing request (POST, PUT, DELETE)
  ✅ Forms and AJAX calls that use cookies

NOT VULNERABLE:
  ✅ GET requests — should never change state
  ✅ JWT in Authorization header — header not auto-sent
  ✅ Requests from the same origin (not cross-site)
```

---

## 🔑 The Fix — Synchronizer Token Pattern

The most common CSRF defence. ASP.NET Core uses this by default.

```
HOW IT WORKS:
──────────────────────────────────────────────────────────
1. Server generates a unique random token per session
2. Server embeds it in the page HTML (hidden field)
3. JavaScript reads the token from the page
4. JavaScript sends it with every POST request (header or form field)
5. Server checks: does the token in the request match
                  the token stored for this session?
6. If yes → legitimate request ✅
   If no  → CSRF attack, reject with 400 ❌

WHY CSRF FAILS WITH THIS:
──────────────────────────────────────────────────────────
evil.com cannot read the token from yourbank.com's HTML
(Same-Origin Policy blocks JavaScript on evil.com
from reading content from yourbank.com)

So evil.com can trigger the request but cannot include
the correct token → server rejects it
```

---

## 🔑 CSRF in ASP.NET Core — Three Ways It's Handled

### Way 1 — Forms (automatic with Tag Helpers)

```cshtml
@* Razor form with Tag Helper — token injected automatically *@
<form asp-action="Create" method="post">
    @* ASP.NET Core automatically adds a hidden token field here *@
    @* <input type="hidden" name="__RequestVerificationToken" value="..." /> *@

    <input name="Name" type="text" />
    <button type="submit">Save</button>
</form>
```

```csharp
[HttpPost]
[ValidateAntiForgeryToken]   // validates the hidden token automatically
public IActionResult Create(Employee emp)
{
    ...
}
```

### Way 2 — AJAX (manual token header)

```javascript
// Get token from hidden field in the page
var token = $('input[name="__RequestVerificationToken"]').val();

$.ajax({
    url:  "/Employee/Delete",
    type: "POST",
    headers: {
        "RequestVerificationToken": token   // send as header
    },
    data: { id: empId },
    success: function(r) { ... }
});
```

### Way 3 — Global setup (once for all AJAX)

```javascript
// Write once — all $.ajax / $.post calls include the token
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(
            "RequestVerificationToken",
            $('input[name="__RequestVerificationToken"]').val()
        );
    }
});
```

---

## 🔑 Which Requests Need CSRF Protection?

```
Needs protection:              Does NOT need protection:
───────────────────────────    ──────────────────────────────────
POST  /Employee/Create   ✅    GET /Employee/GetAll       ❌
POST  /Employee/Update   ✅    GET /Employee/GetById      ❌
POST  /Employee/Delete   ✅    GET requests in general    ❌
PUT   /api/employee/5    ✅
DELETE /api/employee/5   ✅
```

**Rule: any request that changes server state needs protection.**

---

## 📊 CSRF Protection Methods Compared

| Method                         | How It Works                             | Used In              |
| ------------------------------ | ---------------------------------------- | -------------------- |
| **Synchronizer Token**   | Random token in page, verified on server | ASP.NET Core default |
| **SameSite Cookie**      | Cookie only sent to same site            | Modern browsers      |
| **Double Submit Cookie** | Token in cookie AND in header            | Stateless APIs       |
| **Origin Header Check**  | Verify `Origin`header matches          | Simple APIs          |

ASP.NET Core uses **Synchronizer Token** (anti-forgery token). The next chapter covers it in detail.

---

## ❓ Interview Questions

**Q: What is CSRF?**

> Cross-Site Request Forgery — an attack where a malicious website tricks your logged-in browser into sending a request to a target site. The browser automatically includes cookies, so the server can't distinguish the legitimate user's request from the attacker's.

**Q: Why does CSRF only work with cookies and not with JWT tokens in headers?**

> Browsers automatically attach cookies to every request to a domain, even cross-site requests. But headers like `Authorization: Bearer <token>` are never automatically added — JavaScript must explicitly set them. A malicious site can trigger a request but cannot add custom headers, so JWT-in-header authentication is naturally CSRF-resistant.

**Q: Why doesn't CSRF work against GET requests?**

> GET requests should never change server state — they only read data. If a GET causes a state change, that's a design flaw. CSRF protection focuses on state-changing methods (POST, PUT, DELETE).

**Q: Can CSRF steal your data?**

> No. CSRF can trigger actions but cannot read the response — the Same-Origin Policy prevents JavaScript on evil.com from reading the response from yourbank.com. CSRF can cause damage (transfers, deletes) but not data theft.
>
