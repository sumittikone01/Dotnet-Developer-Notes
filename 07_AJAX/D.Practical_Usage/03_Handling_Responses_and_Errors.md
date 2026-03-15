
# 03 — Handling Responses and Errors

---

## 🎯 One-Line Definition

> **Error handling means: always check what the server sent back, handle every possible outcome — success, validation failure, server crash, network failure — and give the user a clear, useful message.**

---

## 🗺️ Every AJAX Call Has 4 Possible Outcomes

```
You make an AJAX call...

  ┌─────────────────────────────────────────────────────┐
  │  Outcome 1 — SUCCESS (2xx)                          │
  │  Server processed your request, data returned      │
  │  → .done() / success callback fires                │
  ├─────────────────────────────────────────────────────┤
  │  Outcome 2 — CLIENT ERROR (4xx)                     │
  │  YOU sent something wrong                           │
  │  400 bad data, 401 not logged in, 404 not found    │
  │  → .fail() / error callback fires                  │
  ├─────────────────────────────────────────────────────┤
  │  Outcome 3 — SERVER ERROR (5xx)                     │
  │  SERVER crashed or has a bug                        │
  │  → .fail() / error callback fires                  │
  ├─────────────────────────────────────────────────────┤
  │  Outcome 4 — NETWORK FAILURE                        │
  │  Request never reached server (no internet, DNS)   │
  │  → .fail() / error callback fires, status = 0     │
  └─────────────────────────────────────────────────────┘
```

---

## 🔑 The Success Path — Reading What Came Back

```javascript
$.get("/Employee/GetAll", function(data) {
    // jQuery auto-parsed the JSON — data is ready to use

    // ── Data could be an array ─────────────────────────────
    if (Array.isArray(data)) {
        data.forEach(function(emp) {
            console.log(emp.name);
        });
    }

    // ── Data could be a single object ─────────────────────
    $("#Name").val(data.name);
    $("#Salary").val(data.salary);

    // ── Data could be a success/fail wrapper ──────────────
    if (data.success) {
        showNotification("Saved: ID = " + data.id, "success");
    } else {
        showNotification("Failed: " + data.message, "error");
    }
});
```

---

## 🔑 The Error Path — What's Inside the Error

```javascript
$.ajax({
    url:  "/Employee/Save",
    type: "POST",
    data: formData,

    error: function(xhr, textStatus, errorThrown) {
        // xhr         = the XMLHttpRequest object
        // textStatus  = "error", "timeout", "abort", "parseerror"
        // errorThrown = the HTTP status text ("Not Found", "Bad Request")

        var statusCode   = xhr.status;          // 400, 404, 500...
        var responseText = xhr.responseText;    // raw response body string
        var responseJSON = xhr.responseJSON;    // parsed JSON (if response was JSON)

        console.log("Status:", statusCode);
        console.log("Text:", textStatus);
        console.log("Error:", errorThrown);
        console.log("Body:", responseJSON);
    }
});
```

---

## 🔑 Handle Each Error Type Differently

```javascript
$.ajax({
    url:  "/Employee/Save",
    type: "POST",
    data: formData,

    success: function(response) {
        if (response.success) {
            showNotification("Saved successfully!", "success");
            refreshGrid();
        } else {
            // Server returned 200 but with a business logic error
            showNotification(response.message, "warning");
        }
    },

    error: function(xhr) {
        var code = xhr.status;

        // ── 400 Bad Request: validation errors ────────────
        if (code === 400) {
            var errors = xhr.responseJSON;

            if (errors && errors.errors) {
                // [ApiController] ProblemDetails format
                $.each(errors.errors, function(field, messages) {
                    showFieldError(field, messages[0]);
                });
            } else if (errors && errors.message) {
                // Custom format: { success: false, message: "..." }
                showNotification(errors.message, "error");
            } else {
                showNotification("Invalid data submitted", "error");
            }
        }

        // ── 401 Unauthorized: session expired ─────────────
        else if (code === 401) {
            showNotification("Session expired — redirecting to login...", "warning");
            setTimeout(function() {
                window.location.href = "/Account/Login";
            }, 1500);
        }

        // ── 403 Forbidden: no permission ──────────────────
        else if (code === 403) {
            showNotification("You don't have permission for this action", "error");
        }

        // ── 404 Not Found: record doesn't exist ───────────
        else if (code === 404) {
            showNotification("Record not found — it may have been deleted", "warning");
            refreshGrid();  // refresh to show current state
        }

        // ── 500 Server Error: something crashed ───────────
        else if (code === 500) {
            showNotification("Server error — please try again or contact support", "error");
            console.error("Server error details:", xhr.responseText);
        }

        // ── 0 Network failure: no internet ────────────────
        else if (code === 0) {
            showNotification("No connection — check your internet and try again", "error");
        }

        // ── Anything else ─────────────────────────────────
        else {
            showNotification("Unexpected error (" + code + ")", "error");
        }
    },

    complete: function() {
        hideSpinner();
        $("#saveBtn").prop("disabled", false);
    }
});
```

---

## 🔑 Displaying Validation Errors in a Form

When the server returns validation errors (from ModelState or `[ApiController]`), show them next to the right fields:

```csharp
// Controller returns validation errors
[HttpPost]
public JsonResult Save(Employee emp)
{
    if (!ModelState.IsValid)
    {
        // Return errors as a dictionary: { "Name": ["required"], "Salary": ["invalid"] }
        var errors = ModelState
            .Where(x => x.Value.Errors.Any())
            .ToDictionary(
                kvp => kvp.Key,
                kvp => kvp.Value.Errors.Select(e => e.ErrorMessage).ToArray()
            );

        return Json(new { success = false, errors = errors });
    }

    _db.Employees.Add(emp);
    _db.SaveChanges();
    return Json(new { success = true, id = emp.Id });
}
```

```javascript
function displayValidationErrors(errors) {
    // First clear all previous errors
    $(".field-error").text("").hide();
    $(".form-field").removeClass("has-error");

    if (!errors) return;

    // Show each error next to its field
    $.each(errors, function(fieldName, messages) {
        var errorEl = $("#" + fieldName + "-error");
        var fieldEl = $("#" + fieldName);

        if (errorEl.length) {
            errorEl.text(messages[0]).show();   // show first error message
        }
        if (fieldEl.length) {
            fieldEl.addClass("has-error");      // highlight the field
        }
    });
}

// Usage:
$.post("/Employee/Save", formData, function(response) {
    if (response.success) {
        showNotification("Saved!", "success");
    } else {
        displayValidationErrors(response.errors);
    }
});
```

---

## 🔑 Handling Timeout

```javascript
$.ajax({
    url:     "/Report/Generate",
    type:    "POST",
    data:    filters,
    timeout: 30000,   // 30 seconds — fail if no response

    success: function(data) {
        renderReport(data);
    },

    error: function(xhr, textStatus) {
        if (textStatus === "timeout") {
            showNotification(
                "Report is taking too long — try a smaller date range",
                "warning"
            );
        } else {
            showNotification("Request failed: " + xhr.status, "error");
        }
    }
});
```

---

## 🔑 Retry Pattern — Try Again on Failure

For transient failures (network blip, temporary server issue):

```javascript
function ajaxWithRetry(options, retries) {
    retries = retries || 3;

    $.ajax(options)
        .fail(function(xhr) {
            // Only retry on server errors or network failures
            // Don't retry on 400 (bad request) — same request will fail again
            if (retries > 0 && (xhr.status === 0 || xhr.status >= 500)) {
                console.log("Retrying... attempts left: " + (retries - 1));
                setTimeout(function() {
                    ajaxWithRetry(options, retries - 1);
                }, 2000);  // wait 2 seconds before retrying
            } else {
                showNotification("Failed after multiple attempts", "error");
            }
        });
}

// Usage
ajaxWithRetry({
    url:     "/Employee/GetAll",
    type:    "GET",
    success: function(data) { renderTable(data); }
});
```

---

## 🔑 Always Block — Clean Up No Matter What

```javascript
$.ajax({
    url:  "/Employee/Save",
    type: "POST",
    data: formData,

    beforeSend: function() {
        // Runs before the request is sent
        $("#saveBtn").prop("disabled", true).text("Saving...");
        showSpinner();
    },

    success: function(response) {
        showNotification("Saved!", "success");
        refreshGrid();
    },

    error: function(xhr) {
        handleError(xhr);
    },

    complete: function() {
        // ALWAYS runs — success or error
        // Put cleanup here so you don't repeat it in success AND error
        $("#saveBtn").prop("disabled", false).text("Save");
        hideSpinner();
    }
});
```

---

## 📊 Error Handling Quick Reference

| Status        | Meaning                         | Action to Take                              |
| ------------- | ------------------------------- | ------------------------------------------- |
| `0`         | Network failure                 | Show "check your connection" message        |
| `400`       | Bad request / validation failed | Show field errors from `xhr.responseJSON` |
| `401`       | Not authenticated               | Redirect to login page                      |
| `403`       | No permission                   | Show "access denied" message                |
| `404`       | Not found                       | Show "record not found", refresh list       |
| `408`       | Request timeout                 | Show "try again" message                    |
| `500`       | Server crashed                  | Show "server error", log to console         |
| `"timeout"` | Client-side timeout             | Show "request taking too long"              |

---

## ❓ Interview Questions

**Q: What is the difference between the `error` callback and `complete` callback?**

> `error` fires only when the request fails (non-2xx status or network failure). `complete` fires after EVERY request — success or failure — making it the right place to put cleanup code like hiding a spinner or re-enabling a button, so you don't repeat it in both `success` and `error`.

**Q: When does `xhr.status` equal 0 in the error callback?**

> When the request never received a response — a network failure, no internet connection, DNS resolution failure, or the request was aborted. It does NOT mean a server error; it means the server was never reached.

**Q: How do you display server-side validation errors next to the correct fields?**

> The controller returns errors as a dictionary keyed by field name. In the AJAX error or failure callback, iterate the dictionary and find each field's corresponding error element by naming convention, set its text, and highlight the field.

**Q: Should you retry a request that returned a 400 error?**

> No. A 400 means the data you sent was invalid — sending the same request again will produce the same 400. Only retry on status 0 (network failure) or 500+ (server errors) where the problem was not in your request data.
>
