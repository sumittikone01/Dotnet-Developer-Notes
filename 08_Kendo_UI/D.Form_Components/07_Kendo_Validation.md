
# 07 — Kendo Validation

---

## 🎯 One-Line Definition

> **Kendo Validation is a client-side form validator that reads HTML5 validation attributes and custom rules, shows styled error messages next to each field before anything is sent to the server, and works seamlessly with all Kendo widgets.**

---

## 🔑 Why Kendo Validation?

```
Without validation:                   With Kendo Validation:
────────────────────────────────      ──────────────────────────────────────
User submits blank form               User submits blank form
  ↓                                     ↓
Server returns error                  Client intercepts BEFORE server call
  ↓                                     ↓
Page reloads (or AJAX fails)          Error messages appear instantly
  ↓                                     next to each invalid field
User sees generic error               User sees specific, styled messages
  ↓                                     ↓
User must fix and retry               User fixes inline, no round-trip
(slow, frustrating)                   (fast, clear, professional)

┌─────────────────────────────────────┐
│  Name *                             │
│  [_______________________________]  │
│  ⚠ Name is required                │  ← Kendo error message
│                                     │
│  Salary                             │
│  [_______________________________]  │
│  ⚠ Salary must be between 10,000   │  ← Kendo error message
│    and 500,000                      │
└─────────────────────────────────────┘
```

---

## 🔑 How It Works — 3 Steps

```
Step 1: You attach HTML5 validation attributes to inputs
        required, min, max, minlength, type="email" etc.

Step 2: Kendo reads those attributes automatically
        No manual configuration needed for standard rules

Step 3: On submit (or on-demand): Kendo checks all fields
        → All valid   → your code runs (AJAX send etc.)
        → Any invalid → error messages appear, submit blocked
```

---

## 🔑 Setting Up the Validator

```html
{{!-- Wrap your form fields in a container element --}}
<div id="employeeForm">

    <div class="mb-3">
        <label>Full Name *</label>
        <input name="Name"
               type="text"
               class="k-textbox"
               required
               data-required-msg="Name is required" />
        {{!-- Error message appears here automatically --}}
    </div>

    <div class="mb-3">
        <label>Email *</label>
        <input name="Email"
               type="email"
               class="k-textbox"
               required
               data-required-msg="Email is required"
               data-email-msg="Enter a valid email address" />
    </div>

    <div class="mb-3">
        <label>Salary</label>
        <input name="Salary"
               type="number"
               min="10000"
               max="1000000"
               data-min-msg="Minimum salary is $10,000"
               data-max-msg="Maximum salary is $1,000,000" />
    </div>

    <button id="saveBtn" type="button">Save</button>

</div>
```

```javascript
$(function() {

    // ── Attach Kendo Validator to the container ───────────────
    var validator = $("#employeeForm").kendoValidator().data("kendoValidator");

    // ── Handle the Save button ───────────────────────────────
    $("#saveBtn").on("click", function() {

        if (validator.validate()) {
            // ✅ All fields valid — send to server
            $.post("/Employee/Save", $("#employeeForm").serialize(), function(r) {
                showNotification("Saved!", "success");
            });
        }
        // ❌ Invalid — error messages already shown automatically

    });
});
```

---

## 🔑 HTML5 Validation Attributes — What Kendo Reads Automatically

```html
{{!-- REQUIRED: field must have a value --}}
<input required
       data-required-msg="This field is required" />

{{!-- EMAIL: must be valid email format --}}
<input type="email"
       data-email-msg="Enter a valid email" />

{{!-- URL: must be a valid URL --}}
<input type="url"
       data-url-msg="Enter a valid URL" />

{{!-- MIN / MAX: numeric range --}}
<input type="number"
       min="0"
       max="100"
       data-min-msg="Must be at least 0"
       data-max-msg="Cannot exceed 100" />

{{!-- MINLENGTH / MAXLENGTH: text length --}}
<input type="text"
       minlength="3"
       maxlength="100"
       data-minlength-msg="At least 3 characters required"
       data-maxlength-msg="Maximum 100 characters" />

{{!-- PATTERN: regular expression --}}
<input type="text"
       pattern="[A-Z]{2}[0-9]{6}"
       data-pattern-msg="Format: 2 letters + 6 digits (e.g. AB123456)" />
```

---

## 🔑 Validation Attributes Reference Table

| HTML Attribute      | Kendo Message Attribute | What It Validates       |
| ------------------- | ----------------------- | ----------------------- |
| `required`        | `data-required-msg`   | Field must have value   |
| `type="email"`    | `data-email-msg`      | Valid email format      |
| `type="url"`      | `data-url-msg`        | Valid URL               |
| `type="number"`   | —                      | Numeric only            |
| `min="5"`         | `data-min-msg`        | Value ≥ 5              |
| `max="100"`       | `data-max-msg`        | Value ≤ 100            |
| `minlength="3"`   | `data-minlength-msg`  | Length ≥ 3 characters  |
| `maxlength="50"`  | `data-maxlength-msg`  | Length ≤ 50 characters |
| `pattern="regex"` | `data-pattern-msg`    | Matches regex           |

> If you don't provide a `data-xxx-msg`, Kendo shows a generic default message.

---

## 🔑 Validating Kendo Widgets

HTML5 attributes work with plain inputs. For Kendo widgets (NumericTextBox, DatePicker, DropDownList), the hidden input that holds the value is what gets validated:

```html
{{!-- Kendo NumericTextBox with validation --}}
<div class="mb-3">
    <label>Salary *</label>
    {{!-- The Kendo widget renders on this input --}}
    <input name="Salary"
           required
           data-required-msg="Salary is required"
           min="10000"
           data-min-msg="Minimum salary is $10,000" />
    {{!-- Attach the widget AFTER the validation attributes --}}
</div>
```

```javascript
// Attach Kendo widget — validation attributes stay on the element
$("#Salary").kendoNumericTextBox({
    format: "c0",
    min:    10000,
    max:    1000000
});

// Then attach validator to the form
var validator = $("#employeeForm").kendoValidator().data("kendoValidator");
```

---

## 🔑 Custom Validation Rules

When the built-in attributes aren't enough, define your own rules:

```javascript
var validator = $("#employeeForm").kendoValidator({

    rules: {

        // Rule 1: Email uniqueness check (async-style using flag)
        uniqueEmail: function(input) {
            if (input.is("[name='Email']")) {
                // Must return true (valid) or false (invalid)
                // For async checks, use a flag pattern
                return input.val() !== "taken@example.com";  // example
            }
            return true;  // not the Email field — skip this rule
        },

        // Rule 2: Salary must be divisible by 1000
        salaryStep: function(input) {
            if (input.is("[name='Salary']")) {
                var val = parseInt(input.val().replace(/[^0-9]/g, ""));
                return isNaN(val) || val % 1000 === 0;
            }
            return true;
        },

        // Rule 3: Hire date cannot be in the future
        hireDateNotFuture: function(input) {
            if (input.is("[name='HireDate']")) {
                var dp   = input.data("kendoDatePicker");
                var date = dp ? dp.value() : null;
                if (!date) return true;  // empty is fine (required handles that)
                return date <= new Date();
            }
            return true;
        },

        // Rule 4: Password strength
        strongPassword: function(input) {
            if (input.is("[name='Password']")) {
                var val = input.val();
                // 8+ chars, uppercase, lowercase, digit
                return /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/.test(val);
            }
            return true;
        },

        // Rule 5: End date after start date
        endAfterStart: function(input) {
            if (input.is("[name='EndDate']")) {
                var start = $("#StartDate").data("kendoDatePicker").value();
                var end   = input.data("kendoDatePicker")
                              ? input.data("kendoDatePicker").value()
                              : null;
                if (!start || !end) return true;
                return end > start;
            }
            return true;
        }
    },

    messages: {
        uniqueEmail:       "This email address is already in use",
        salaryStep:        "Salary must be in multiples of $1,000",
        hireDateNotFuture: "Hire date cannot be in the future",
        strongPassword:    "Password needs 8+ chars with uppercase, lowercase, and a number",
        endAfterStart:     "End date must be after the start date"
    }

}).data("kendoValidator");
```

---

## 🔑 Validator API — Manual Control

```javascript
var validator = $("#employeeForm").kendoValidator().data("kendoValidator");

// ── Validate all fields at once ───────────────────────────────
validator.validate()
// Returns: true if all valid, false if any invalid
// Side effect: shows error messages for invalid fields

// ── Validate a single field ───────────────────────────────────
validator.validateInput($("#Email"))
// Validates just the Email field
// Returns: true / false

// ── Check if currently valid (no side effects) ────────────────
// Note: validate() shows messages. For silent check, there's no direct method.
// Common pattern:
var isValid = validator.validate();
// If you don't want to show messages, validate on demand only.

// ── Clear all error messages ──────────────────────────────────
validator.hideMessages()
// Removes all currently visible error messages

// ── Destroy the validator ─────────────────────────────────────
validator.destroy()
```

---

## 🔑 Validation Events

```javascript
var validator = $("#employeeForm").kendoValidator({

    // ── validate: fires when validate() is called ─────────────
    validate: function(e) {
        // e.valid  = true if all valid, false if any invalid
        // e.sender = the validator instance
        if (!e.valid) {
            console.log("Form has errors — not submitting");
        }
    },

    // ── validateInput: fires per field when it's validated ─────
    validateInput: function(e) {
        // e.valid  = true/false for this specific input
        // e.input  = the jQuery element that was validated
        if (e.valid) {
            e.input.closest(".form-group").removeClass("has-error")
                   .addClass("has-success");
        } else {
            e.input.closest(".form-group").removeClass("has-success")
                   .addClass("has-error");
        }
    }

}).data("kendoValidator");
```

---

## 🔑 Real-World Complete Form With Validation

```cshtml
@Html.AntiForgeryToken()

<div id="employeeForm">

    <div class="row mb-3">
        <div class="col-md-6">
            <label>Full Name *</label>
            <input name="Name" class="k-textbox w-100"
                   required           minlength="2"   maxlength="100"
                   data-required-msg="Name is required"
                   data-minlength-msg="At least 2 characters"
                   data-maxlength-msg="Maximum 100 characters"
                   placeholder="Enter full name" />
        </div>
        <div class="col-md-6">
            <label>Email *</label>
            <input name="Email" type="email" class="k-textbox w-100"
                   required
                   data-required-msg="Email is required"
                   data-email-msg="Enter a valid email address" />
        </div>
    </div>

    <div class="row mb-3">
        <div class="col-md-4">
            <label>Department *</label>
            <select name="Department" required
                    data-required-msg="Please select a department">
                <option value="">-- Select --</option>
                <option>IT</option>
                <option>HR</option>
                <option>Finance</option>
            </select>
        </div>
        <div class="col-md-4">
            <label>Salary *</label>
            <input name="Salary" type="number"
                   required    min="10000"   max="1000000"
                   data-required-msg="Salary is required"
                   data-min-msg="Minimum is $10,000"
                   data-max-msg="Maximum is $1,000,000" />
        </div>
        <div class="col-md-4">
            <label>Hire Date *</label>
            <input name="HireDate" required
                   data-required-msg="Hire date is required" />
        </div>
    </div>

    <button id="saveEmployeeBtn" type="button"
            class="k-button k-button-solid-primary">
        Save Employee
    </button>
    <button id="cancelBtn" type="button" class="k-button">
        Cancel
    </button>

</div>
```

```javascript
$(function() {

    // Attach Kendo widgets
    $("[name='Salary']").kendoNumericTextBox({ format: "c0", min: 10000 });
    $("[name='HireDate']").kendoDatePicker({ format: "MM/dd/yyyy", max: new Date() });
    $("[name='Department']").kendoDropDownList({ optionLabel: "-- Select --" });

    // Attach validator with custom rules
    var validator = $("#employeeForm").kendoValidator({
        rules: {
            hireDateNotFuture: function(input) {
                if (input.is("[name='HireDate']")) {
                    var dp   = input.data("kendoDatePicker");
                    var date = dp ? dp.value() : null;
                    return !date || date <= new Date();
                }
                return true;
            }
        },
        messages: {
            hireDateNotFuture: "Hire date cannot be in the future"
        }
    }).data("kendoValidator");

    // Save button
    $("#saveEmployeeBtn").on("click", function() {
        if (!validator.validate()) {
            showNotification("Please fix the errors above", "error");
            return;
        }

        var formData = {
            name:       $("[name='Name']").val(),
            email:      $("[name='Email']").val(),
            department: $("[name='Department']").data("kendoDropDownList").value(),
            salary:     $("[name='Salary']").data("kendoNumericTextBox").value(),
            hireDate:   kendo.toString(
                            $("[name='HireDate']").data("kendoDatePicker").value(),
                            "yyyy-MM-dd"
                        )
        };

        $.post("/Employee/Create", formData)
            .done(function(r) {
                if (r.success) {
                    showNotification("Employee saved!", "success");
                    validator.hideMessages();
                } else {
                    showNotification(r.message, "error");
                }
            })
            .fail(function(xhr) {
                showNotification("Server error: " + xhr.status, "error");
            });
    });

    // Cancel button — clear messages
    $("#cancelBtn").on("click", function() {
        validator.hideMessages();
        $("#employeeForm")[0].reset();
    });
});
```

---

## 🔑 Kendo Grid Validation (Server-Side Errors)

Inside the Kendo Grid, server errors are returned automatically via `ModelState`:

```csharp
// Controller
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    // Custom server rule
    if (_db.Employees.Any(e => e.Email == emp.Email))
        ModelState.AddModelError("Email", "Email already exists");

    if (ModelState.IsValid) { _db.Employees.Add(emp); _db.SaveChanges(); }

    // ModelState errors flow back in the Errors field
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
}
```

```
Grid popup shows:
┌──────────────────────────────────────┐
│  Edit Employee                   [×] │
│                                      │
│  Name:   [___Alice_____________]     │
│                                      │
│  Email:  [___alice@test.com____]     │
│  ⚠ Email already exists             │  ← from ModelState
│                                      │
│              [Update]  [Cancel]      │
└──────────────────────────────────────┘
```

---

## 📊 Validation Quick Reference

| Rule         | HTML                        | Custom Rule              | Message Attribute                 |
| ------------ | --------------------------- | ------------------------ | --------------------------------- |
| Required     | `required`                | —                       | `data-required-msg`             |
| Email        | `type="email"`            | —                       | `data-email-msg`                |
| Number range | `min`/`max`             | —                       | `data-min-msg`/`data-max-msg` |
| Length       | `minlength`/`maxlength` | —                       | `data-minlength-msg`            |
| Pattern      | `pattern`                 | —                       | `data-pattern-msg`              |
| Custom logic | —                          | function in `rules:{}` | string in `messages:{}`         |

---

## ⚠️ Common Mistakes

| Mistake                                              | Symptom                            | Fix                                                                   |
| ---------------------------------------------------- | ---------------------------------- | --------------------------------------------------------------------- |
| Validator on `<form>`when using AJAX               | Form submits normally on Enter key | Use a `<div>`container OR call `e.preventDefault()`on submit      |
| `kendoValidator()`before widgets are initialized   | Kendo widgets not validated        | Init all widgets first, then call `kendoValidator()`                |
| Custom rule returns `undefined`instead of `true` | Rule always fails                  | Always explicitly `return true`for fields that don't match the rule |
| `validate()`not called before AJAX                 | Invalid data sent to server        | Always call `if (!validator.validate()) return;`before posting      |
| Not calling `hideMessages()`after success          | Old error messages stay visible    | Call `validator.hideMessages()`after successful save                |

---

## ❓ Interview Questions

**Q: What is the Kendo Validator and how do you attach it?**

> A client-side form validation component that reads HTML5 attributes (`required`, `min`, `max`, etc.) and custom rules. Attach it by calling `.kendoValidator()` on a container element. Use `validator.validate()` to trigger validation — returns `true` if all fields pass.

**Q: How do you create a custom validation rule?**

> Pass a `rules` object and `messages` object to `kendoValidator()`. Each rule is a function receiving the `input` element — return `true` if valid, `false` if invalid. Check `input.is("[name='fieldName']")` to target specific fields; return `true` for all others.

**Q: What is the difference between client-side Kendo Validation and server-side ModelState validation?**

> Client-side Kendo Validation runs in the browser before any server call — it's fast and prevents unnecessary requests. Server-side ModelState validation runs in the C# controller after data arrives — it's the safety net for business rules that can't run client-side (like database uniqueness checks). Both are needed.

**Q: How do you show server-side ModelState errors in a Kendo Grid popup?**

> The controller returns `new[] { model }.ToDataSourceResult(request, ModelState)`. Kendo reads the `Errors` field in the response and displays each error next to its corresponding field in the edit popup automatically.

**Q: Why should you attach Kendo widgets before calling `kendoValidator()`?**

> The validator inspects the DOM at init time to find form elements. If widgets are attached afterward, their underlying `<input>` elements may be replaced or moved, causing the validator to miss them. Init order: widgets first → validator second.
>
