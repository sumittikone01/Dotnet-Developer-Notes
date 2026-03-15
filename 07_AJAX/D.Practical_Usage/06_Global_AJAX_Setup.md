# 06 — Global AJAX Setup

---

## 🎯 One-Line Definition

> **Global AJAX setup means writing common AJAX configuration once — anti-forgery tokens, loading spinners, session expiry handling, error messages — so every single AJAX call in your app gets it automatically.**

---

## 🤔 The Problem It Solves

Without global setup, you repeat the same code in every AJAX call:

```javascript
// ❌ WITHOUT global setup — repeating in every call:

$.ajax({
    url:  "/Employee/Create",
    type: "POST",
    beforeSend: function(xhr) {                              // ← repeated
        xhr.setRequestHeader("RequestVerificationToken",    // ← repeated
            $('[name="__RequestVerificationToken"]').val());// ← repeated
    },
    success: function(data) { ... },
    error: function(xhr) {
        if (xhr.status === 401) window.location.href = "/Login"; // ← repeated
        else showError(xhr.status);                              // ← repeated
    },
    complete: function() { hideSpinner(); }   // ← repeated
});

// Then the SAME beforeSend + error + complete
// in EVERY other $.ajax call across your whole app
```

```javascript
// ✅ WITH global setup — write once, applies everywhere:

// setup.js — runs once on page load
$.ajaxSetup({ ... });    // covers ALL calls below

// Now every call is clean:
$.post("/Employee/Create", data, function(r) { ... });
$.get("/Employee/GetAll", function(data) { ... });
// ↑ Both automatically get the token, spinner, error handling
```

---

## 🔑 `$.ajaxSetup()` — The Global Configuration Method

```javascript
// $.ajaxSetup() sets default options for ALL future $.ajax calls
// Place this in your _Layout.cshtml scripts section or a shared .js file

$.ajaxSetup({
    // These settings apply to every $.ajax / $.get / $.post call
    // unless the individual call overrides them

    dataType: "json",   // expect JSON back from every endpoint

    error: function(xhr, textStatus, errorThrown) {
        // Global error handler — fires for every failed request
        handleGlobalError(xhr);
    }
});
```

---

## 🔑 Anti-Forgery Token — Set Once, Works Everywhere

The most important use of global setup in ASP.NET MVC:

```cshtml
@* In _Layout.cshtml — generates a hidden input with the token *@
@Html.AntiForgeryToken()
```

```javascript
// In your shared JS file or _Layout.cshtml scripts section
// This runs BEFORE every single AJAX request

$.ajaxSetup({
    beforeSend: function(xhr) {
        var token = $('input[name="__RequestVerificationToken"]').val();
        if (token) {
            xhr.setRequestHeader("RequestVerificationToken", token);
        }
    }
});

// Now EVERY $.post / $.ajax automatically includes the token
// You never write setRequestHeader again in individual calls

// Controller with [ValidateAntiForgeryToken] works automatically:
$.post("/Employee/Delete", { id: 5 }, function(r) {
    // ↑ Token is added automatically by the global setup
    showNotification("Deleted");
});
```

---

## 🔑 Global Loading Spinner

Show a spinner when ANY request starts. Hide it when ALL requests finish:

```html
<!-- In _Layout.cshtml — one spinner for the whole app -->
<div id="globalSpinner" style="display:none;">
    <div class="spinner-overlay">
        <div class="spinner"></div>
    </div>
</div>
```

```javascript
// ajaxStart fires when the FIRST of potentially many requests starts
$(document).on("ajaxStart", function() {
    $("#globalSpinner").show();
});

// ajaxStop fires when the LAST active request finishes
$(document).on("ajaxStop", function() {
    $("#globalSpinner").hide();
});

// With Kendo UI loading indicator
$(document).on("ajaxStart", function() {
    kendo.ui.progress($("body"), true);   // Kendo's built-in overlay spinner
});

$(document).on("ajaxStop", function() {
    kendo.ui.progress($("body"), false);
});
```

---

## 🔑 Global Error Handler — Handle Common Errors Once

```javascript
$(document).on("ajaxError", function(event, xhr, settings, error) {
    var code = xhr.status;

    // ── Session expired → redirect to login ────────────────
    if (code === 401) {
        showNotification(
            "Your session has expired. Redirecting to login...",
            "warning"
        );
        setTimeout(function() {
            window.location.href = "/Account/Login?returnUrl=" +
                encodeURIComponent(window.location.pathname);
        }, 2000);
        return;
    }

    // ── No permission ────────────────────────────────────────
    if (code === 403) {
        showNotification("You don't have permission for this action.", "error");
        return;
    }

    // ── Server crashed ────────────────────────────────────────
    if (code === 500) {
        showNotification(
            "A server error occurred. Please try again.",
            "error"
        );
        // Log to console for debugging
        console.error("500 Error on:", settings.url, xhr.responseText);
        return;
    }

    // ── Network failure ───────────────────────────────────────
    if (code === 0) {
        showNotification("No internet connection. Check your network.", "error");
        return;
    }
});
```

---

## 🔑 Putting It All Together — The Complete Setup File

Create one `ajax-setup.js` file. Include it in every page via `_Layout.cshtml`:

```javascript
// wwwroot/js/ajax-setup.js
// Include this ONCE in _Layout.cshtml after jQuery and Kendo

$(function() {

    // ── 1. Anti-forgery token on every POST ───────────────────
    $.ajaxSetup({
        beforeSend: function(xhr) {
            var token = $('input[name="__RequestVerificationToken"]').val();
            if (token) {
                xhr.setRequestHeader("RequestVerificationToken", token);
            }
        }
    });

    // ── 2. Global loading indicator ───────────────────────────
    $(document).on("ajaxStart", function() {
        kendo.ui.progress($("#pageContent"), true);
    });

    $(document).on("ajaxStop", function() {
        kendo.ui.progress($("#pageContent"), false);
    });

    // ── 3. Global error handler ───────────────────────────────
    $(document).on("ajaxError", function(e, xhr, settings) {

        // Don't handle if the individual call has its own error handler
        // (Individual handlers take priority over global ones)

        var code = xhr.status;

        if (code === 401) {
            showNotification("Session expired — please log in again.", "warning");
            setTimeout(function() {
                window.location.href = "/Account/Login";
            }, 2000);
        }
        else if (code === 403) {
            showNotification("Access denied.", "error");
        }
        else if (code === 500) {
            showNotification("Server error. Please try again.", "error");
            console.error("Error on " + settings.url + ":", xhr.responseText);
        }
        else if (code === 0) {
            showNotification("Connection lost. Check your internet.", "error");
        }
    });

    // ── 4. Shared notification helper ─────────────────────────
    window.showNotification = function(message, type) {
        // type: "success", "error", "warning", "info"
        var notifier = $("#globalNotification").data("kendoNotification");
        if (notifier) {
            notifier[type](message);
        } else {
            // Fallback if Kendo Notification not initialised
            console.log("[" + type.toUpperCase() + "] " + message);
        }
    };

});
```

```html
<!-- In _Layout.cshtml — include after jQuery and Kendo -->
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/kendo/js/kendo.all.min.js"></script>
<script src="~/lib/kendo/js/kendo.aspnetmvc.min.js"></script>
<script src="~/js/ajax-setup.js"></script>   <!-- ← your global setup -->

<!-- Kendo Notification widget for toast messages -->
<span id="globalNotification"></span>
<script>
    $("#globalNotification").kendoNotification({
        position: { top: 20, right: 20 },
        autoHideAfter: 4000,
        stacking: "down"
    });
</script>

@Html.AntiForgeryToken()   <!-- ← token picked up by ajaxSetup -->
```

---

## 🔑 Excluding Specific Calls from Global Setup

Sometimes one call needs different behaviour — skip the global error handler for it:

```javascript
// Option 1: Override in individual call
$.ajax({
    url:   "/Employee/Check",
    type:  "GET",
    global: false,   // ← excludes from ajaxStart/ajaxStop/ajaxError
    success: function(r) { ... },
    error:   function(xhr) {
        // Custom handling for this call only
    }
});

// Option 2: Handle errors locally — global handler still fires
// but you can use a flag to prevent double-handling
$.ajax({
    url:   "/Employee/Save",
    type:  "POST",
    error: function(xhr) {
        // Display field-level validation errors here
        displayFieldErrors(xhr.responseJSON);
        // Global handler will ALSO fire for this call
        // unless you call event.stopPropagation() — rarely needed
    }
});
```

---

## 📊 Global Events Quick Reference

| Event            | When It Fires                      | Use For                 |
| ---------------- | ---------------------------------- | ----------------------- |
| `ajaxStart`    | First request begins               | Show spinner            |
| `ajaxStop`     | All requests finished              | Hide spinner            |
| `ajaxError`    | Any request fails                  | Global error messages   |
| `ajaxSuccess`  | Any request succeeds               | Global success logging  |
| `ajaxComplete` | Any request ends (success or fail) | Cleanup                 |
| `ajaxSend`     | Before each request is sent        | Add headers per-request |

---

## ❓ Interview Questions

**Q: What is `$.ajaxSetup()` and why is it useful?**

> It sets default options that apply to every `$.ajax`, `$.get`, and `$.post` call made after it runs. It's useful for setting the anti-forgery token header once instead of in every individual call.

**Q: What is the difference between `ajaxStart` and `ajaxSend`?**

> `ajaxSend` fires before every individual request — even if other requests are already running. `ajaxStart` fires only when the first request begins after a period of no activity. Use `ajaxStart`/`ajaxStop` for a spinner that represents "something is loading" — it won't flicker on and off for every one of multiple parallel requests.

**Q: How do you prevent the global error handler from firing for a specific call?**

> Set `global: false` in that call's options: `$.ajax({ ..., global: false, ... })`. This excludes it from all global jQuery AJAX events.

**Q: Why should the global AJAX setup file be loaded after jQuery but before your page-specific scripts?**

> The setup must be in place before any AJAX calls are made. Page scripts make AJAX calls — so setup must run first. jQuery must be loaded before any jQuery code runs, so the order is: jQuery → Kendo → ajax-setup.js → page scripts.
>
