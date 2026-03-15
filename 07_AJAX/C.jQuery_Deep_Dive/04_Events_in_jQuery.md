
# 04 — Events in jQuery

---

## 🎯 One-Line Definition

> **An event is something that happens on the page — a click, a keypress, a form submit. jQuery lets you listen for these events and run your own code when they happen.**

---

## 🔑 How Events Work — The Basic Idea

```
User does something          jQuery detects it        Your code runs
───────────────────          ─────────────────        ──────────────
Clicks a button          →   "click" event fires  →   saveEmployee()
Types in a search box    →   "keyup" event fires   →   filterTable()
Changes a dropdown       →   "change" event fires  →   loadCities()
Submits a form           →   "submit" event fires  →   validateAndSend()
Hovers over a row        →   "mouseenter" fires    →   highlightRow()
```

---

## 🔑 `.on()` — The One Method to Rule Them All

jQuery has one main way to attach event handlers: `.on()`

```javascript
// Pattern:
$(selector).on(eventName, handlerFunction)

// Examples:
$("#saveBtn").on("click", function() {
    saveEmployee();
});

$("#searchBox").on("keyup", function() {
    filterTable($(this).val());
});

$("#DepartmentId").on("change", function() {
    loadEmployeesForDept($(this).val());
});

$("form").on("submit", function(e) {
    e.preventDefault();   // stop normal form submission
    submitViaAjax();
});
```

---

## 🔑 `$(this)` — The Element That Was Clicked

Inside any event handler, `this` refers to the  **exact element that triggered the event** .
Wrap it in `$()` to get jQuery methods on it.

```javascript
$("tr").on("click", function() {
    // 'this' = the specific <tr> that was clicked
    // $(this) = that <tr> wrapped in jQuery

    $(this).addClass("selected");        // highlight THIS row
    var empId = $(this).data("id");      // read THIS row's data-id
    var name  = $(this).find("td:first").text();  // first cell of THIS row
    loadEmployeeDetails(empId);
});

$(".deleteBtn").on("click", function() {
    var row   = $(this).closest("tr");   // find the row THIS button is in
    var empId = row.data("id");          // get id from that row
    deleteEmployee(empId, row);
});
```

---

## 🔑 The Event Object — `e` or `event`

jQuery passes an event object to your handler. It contains useful information:

```javascript
$("#saveBtn").on("click", function(e) {
    // e = the event object

    e.preventDefault()     // stop default browser action
                           // (stops form submit, stops link navigation)

    e.stopPropagation()    // stop event from bubbling up to parent elements

    e.target               // the EXACT element clicked (could be a child)
    e.currentTarget        // the element the handler is attached to (same as this)
    e.type                 // "click", "keyup", "change"...
    e.which                // key code for keyboard events (13 = Enter)
    e.pageX / e.pageY      // mouse position on the page
});

// Most common use: prevent form from reloading the page
$("form").on("submit", function(e) {
    e.preventDefault();   // ← this one line is used constantly in MVC+AJAX
    submitViaAjax();
});
```

---

## 🔑 Common Events You'll Use Daily

```
MOUSE EVENTS
─────────────────────────────────────────────────────────────
click         → user clicks the element
dblclick      → user double-clicks
mouseenter    → mouse enters element area (doesn't bubble)
mouseleave    → mouse leaves element area (doesn't bubble)
mouseover     → mouse enters element or any child (bubbles)
contextmenu   → right-click


KEYBOARD EVENTS
─────────────────────────────────────────────────────────────
keydown       → key pressed down (fires first)
keyup         → key released (fires last — use for "after typing")
keypress      → key pressed (deprecated, use keydown)


FORM EVENTS
─────────────────────────────────────────────────────────────
submit        → form submitted
change        → value changed AND element lost focus
               (for select, checkbox, radio: fires immediately)
input         → value changes with every keystroke (real-time)
focus         → element receives focus (cursor enters)
blur          → element loses focus (cursor leaves)
focusin       → focus including children
focusout      → blur including children


DOCUMENT / WINDOW EVENTS
─────────────────────────────────────────────────────────────
$(document).ready()  → DOM is fully loaded
$(window).on("resize", fn)  → browser window resized
$(window).on("scroll", fn)  → page scrolled
```

---

## 🔑 Event Delegation — The Pattern You Need for Dynamic Content

**The problem:** You attach a click handler to `.deleteBtn` buttons. Then the Kendo Grid reloads — new rows, new buttons. The old handlers are gone. New buttons don't respond.

**The solution: Event Delegation**

```javascript
// ❌ WRONG — attaches handler only to buttons that exist RIGHT NOW
$(".deleteBtn").on("click", function() {
    deleteEmployee($(this).data("id"));
});
// After grid reloads: new .deleteBtn buttons won't respond

// ✅ CORRECT — delegate to a parent that always exists
$(document).on("click", ".deleteBtn", function() {
    deleteEmployee($(this).data("id"));
});
// Works for ALL .deleteBtn elements — current AND future ones

// Better: delegate to the closest stable parent, not document
$("#employeeTableBody").on("click", ".deleteBtn", function() {
    deleteEmployee($(this).data("id"));
});
```

**How delegation works:**

```
User clicks .deleteBtn
    │
    ▼
Click bubbles up to #employeeTableBody
    │
    ▼
jQuery checks: did the click start on ".deleteBtn"?
    │
    ├── Yes → run the handler, $(this) = the .deleteBtn
    └── No  → ignore
```

**Delegation syntax:**

```javascript
// $(stableParent).on(event, targetSelector, handler)
//   ↑ always exists      ↑ dynamically created

$(document).on("click",  ".deleteBtn",   handleDelete);
$(document).on("click",  ".editBtn",     handleEdit);
$(document).on("change", ".statusDdl",   handleStatusChange);
```

---

## 🔑 Multiple Events, One Handler

```javascript
// Same handler for multiple events
$("#searchBox").on("keyup paste input", function() {
    filterTable($(this).val());
});

// Different handlers in one .on() call (object syntax)
$("#nameInput").on({
    focus: function() { $(this).addClass("focused"); },
    blur:  function() { $(this).removeClass("focused"); },
    input: function() { validateName($(this).val()); }
});
```

---

## 🔑 Removing and Triggering Events

```javascript
// Remove a specific handler
function handleClick() { ... }
$("#btn").on("click", handleClick);
$("#btn").off("click", handleClick);    // remove only this handler

// Remove ALL click handlers
$("#btn").off("click");

// Remove ALL handlers of ALL types
$("#btn").off();

// Manually trigger an event (as if user did it)
$("#saveBtn").trigger("click");         // fires the click handler
$("#myForm").trigger("submit");         // fires submit handler
$("#DeptDdl").trigger("change");        // fires change handler — useful to
                                        // force an initial load
```

---

## 🔑 Shorthand Methods (Old Style — Still Seen in Code)

These still work and you'll see them in older projects:

```javascript
// Old shorthand          Modern .on() equivalent
// ───────────────────    ──────────────────────────────────────
$("#btn").click(fn)    →  $("#btn").on("click", fn)
$("form").submit(fn)   →  $("form").on("submit", fn)
$("#ddl").change(fn)   →  $("#ddl").on("change", fn)
$("#inp").keyup(fn)    →  $("#inp").on("keyup", fn)
$("#inp").focus(fn)    →  $("#inp").on("focus", fn)
$("#inp").blur(fn)     →  $("#inp").on("blur", fn)
```

> **Stick with `.on()` in new code.** It's the one consistent method for all events.

---

## 💻 Real Patterns You'll Use in MVC + Kendo Projects

### Pattern 1 — Save button with AJAX

```javascript
$(function() {
    $("#saveBtn").on("click", function(e) {
        e.preventDefault();

        var data = {
            name:       $("#Name").val(),
            department: $("#Department").val(),
            salary:     parseFloat($("#Salary").val())
        };

        $.post("/Employee/Save", data, function(response) {
            if (response.success) {
                $("#employeeGrid").data("kendoGrid").dataSource.read();
                showNotification("Saved successfully", "success");
            }
        });
    });
});
```

### Pattern 2 — Cascading dropdown (Department → Employees)

```javascript
$("#DepartmentDdl").on("change", function() {
    var deptId = $(this).val();

    if (!deptId) return;

    $.get("/Employee/GetByDept", { departmentId: deptId }, function(employees) {
        var ddl = $("#EmployeeDdl").data("kendoDropDownList");
        ddl.setDataSource(employees);
    });
});
```

### Pattern 3 — Live search with keyup

```javascript
var searchTimer;

$("#searchInput").on("keyup", function() {
    clearTimeout(searchTimer);
    var term = $(this).val();

    // Wait 300ms after user stops typing before searching
    searchTimer = setTimeout(function() {
        if (term.length >= 2) {
            searchEmployees(term);
        }
    }, 300);
});
```

### Pattern 4 — Confirm before delete

```javascript
$(document).on("click", ".deleteBtn", function() {
    var row   = $(this).closest("tr");
    var empId = row.data("id");
    var name  = row.find("td:first").text();

    kendo.confirm("Delete " + name + "?")
        .then(function() {
            $.post("/Employee/Delete", { id: empId }, function() {
                row.remove();
                showNotification("Deleted", "success");
            });
        });
});
```

---

## 📊 Events Quick Reference

| Event          | Fires When                      | Common Use                      |
| -------------- | ------------------------------- | ------------------------------- |
| `click`      | Element is clicked              | Buttons, rows, links            |
| `submit`     | Form is submitted               | Intercept and use AJAX instead  |
| `change`     | Select/input value changes      | Cascade dropdowns, refresh grid |
| `keyup`      | Key is released                 | Live search, character count    |
| `input`      | Value changes (every keystroke) | Real-time validation            |
| `focus`      | Element receives cursor         | Highlight active field          |
| `blur`       | Element loses cursor            | Validate on exit                |
| `mouseenter` | Mouse enters element            | Hover effects                   |
| `mouseleave` | Mouse leaves element            | Undo hover effects              |

---

## ❓ Interview Questions

**Q: What is event delegation and why is it needed?**

> Event delegation attaches a handler to a stable parent element instead of the target element directly. When an event bubbles up to the parent, jQuery checks if it originated from the target selector. This is needed for dynamically created elements — like rows added by a Kendo Grid reload — because handlers attached directly to elements that didn't exist at page load won't work.

**Q: What does `e.preventDefault()` do?**

> It cancels the browser's default action for that event. For a form `submit` event, it stops the page from reloading. For a link `click`, it stops navigation. Essential when handling form submissions with AJAX.

**Q: What is the difference between `$(this)` and `e.target`?**

> `$(this)` refers to the element the handler is attached to. `e.target` is the exact element that was clicked — which could be a child element inside it. Usually the same, but differ when a parent has a click handler and user clicks a child inside.

**Q: What is the difference between `change` and `input` events?**

> `input` fires on every single keystroke as the value changes. `change` fires only when the element loses focus after its value has changed. For real-time feedback use `input`; for validation after user finishes typing use `change` or `blur`.
>
