
# 03 — DOM Manipulation with jQuery

---

## 🎯 One-Line Definition

> **DOM manipulation means changing what's on the page after it has loaded — updating text, changing styles, adding or removing elements, showing and hiding things — all without a page reload.**

---

## 🗂️ The 5 Types of DOM Manipulation

```
┌─────────────────────────────────────────────────────────┐
│  1. Content    → change text or HTML inside elements   │
│  2. Attributes → change id, class, href, src, data-*   │
│  3. Styles     → change CSS (show/hide, color, size)   │
│  4. Classes    → add/remove/toggle CSS classes         │
│  5. Structure  → add, move, copy, or remove elements   │
└─────────────────────────────────────────────────────────┘
```

---

## 🔑 1 — Content: Reading and Writing

```javascript
// ── .text() — plain text (safe, strips HTML tags) ─────────────
$("#title").text()                  // GET: "Employee List"
$("#title").text("Updated Title")   // SET: changes the text

// ── .html() — HTML content (renders HTML tags) ────────────────
$("#container").html()
// GET: "<h1>Hello</h1><p>World</p>"

$("#container").html("<b>New Bold Content</b>")
// SET: replaces everything inside with new HTML

// ── .val() — form field values ────────────────────────────────
$("#nameInput").val()               // GET: "Alice"
$("#nameInput").val("Bob")          // SET: fills the input with "Bob"

$("#salaryInput").val()             // GET: "75000" (always a string)
parseFloat($("#salaryInput").val()) // convert to number before using

// ── .text() vs .html() — know the difference ─────────────────
// Given: <p id="msg"></p>

$("#msg").text("<b>Hello</b>")
// Page shows:  <b>Hello</b>   ← shows the tag characters, not bold

$("#msg").html("<b>Hello</b>")
// Page shows:  Hello          ← renders as actual bold text

// Rule: use .text() when showing user-provided data (prevents XSS)
//       use .html() when you control the content and need HTML tags
```

---

## 🔑 2 — Attributes

```javascript
// ── .attr() — read and write HTML attributes ──────────────────
$("img").attr("src")                    // GET: "/images/photo.jpg"
$("img").attr("src", "/images/new.jpg") // SET: changes the image

$("a").attr("href")                     // GET: "https://..."
$("a").attr("href", "/Employee/Index")  // SET: changes link target

$("input").attr("disabled", true)       // SET: disables the input
$("input").attr("disabled", false)      // SET: enables it again

// Remove an attribute entirely
$("input").removeAttr("disabled")       // removes disabled attribute

// ── .data() — read and write data-* attributes ────────────────
// <tr id="row1" data-employee-id="5" data-dept="IT">

$("#row1").data("employeeId")           // GET: 5 (number, auto-converted)
$("#row1").data("dept")                 // GET: "IT"
$("#row1").data("employeeId", 10)       // SET in memory (not in HTML attr)

// ── .val() — form values (revisit from content section) ───────
$("#IsActive").prop("checked")          // GET: true/false for checkbox
$("#IsActive").prop("checked", true)    // SET: check the checkbox

// .prop() for true/false properties (checked, disabled, selected)
// .attr() for string attributes (id, name, href, src, class)
```

---

## 🔑 3 — Styles: Show, Hide, and CSS

```javascript
// ── Show and Hide ─────────────────────────────────────────────
$("#spinner").show()           // display: block (or original display)
$("#spinner").hide()           // display: none
$("#spinner").toggle()         // show if hidden, hide if visible

// With animation duration (milliseconds)
$("#panel").show(300)          // fade in over 300ms
$("#panel").hide("fast")       // "fast" = 200ms, "slow" = 600ms

// ── .css() — read and write inline styles ─────────────────────
$("#box").css("color")                       // GET: "rgb(0,0,0)"
$("#box").css("color", "red")                // SET: text colour
$("#box").css("background-color", "#f0f0f0") // SET: background
$("#box").css("font-size", "18px")           // SET: font size

// SET multiple styles at once (pass an object)
$("#box").css({
    "color":            "white",
    "background-color": "#2c3e50",
    "padding":          "10px",
    "border-radius":    "5px"
});

// ── Visibility vs display ─────────────────────────────────────
$("#box").hide()               // display: none  — element takes no space
$("#box").css("visibility", "hidden") // still takes space, just invisible
```

---

## 🔑 4 — Classes: The Right Way to Change Styles

Using `.css()` mixes style into JavaScript. The cleaner approach: define styles in CSS, toggle classes with jQuery.

```css
/* In your site.css */
.highlight  { background-color: yellow; font-weight: bold; }
.error-row  { background-color: #ffe0e0; color: red; }
.hidden     { display: none; }
.active     { border-left: 4px solid #3498db; }
```

```javascript
// ── Add a class ───────────────────────────────────────────────
$("tr:first").addClass("highlight")      // adds highlight class
$("tr:first").addClass("highlight active")  // add multiple at once

// ── Remove a class ────────────────────────────────────────────
$("tr:first").removeClass("highlight")
$("tr:first").removeClass("highlight active")  // remove multiple

// ── Toggle a class ────────────────────────────────────────────
$("#row1").toggleClass("active")   // adds if not there, removes if there

// ── Check if a class exists ───────────────────────────────────
if ($("#row1").hasClass("active")) {
    console.log("Row is active");
}

// ── Real example: highlight a row when selected ───────────────
$("tr").on("click", function() {
    $("tr").removeClass("highlight");       // remove from all rows
    $(this).addClass("highlight");          // add to clicked row only
});

// ── Real example: show error state ───────────────────────────
function showFieldError(fieldId, message) {
    $("#" + fieldId).addClass("error-row");
    $("#" + fieldId + "-error").text(message).show();
}

function clearFieldError(fieldId) {
    $("#" + fieldId).removeClass("error-row");
    $("#" + fieldId + "-error").text("").hide();
}
```

---

## 🔑 5 — Structure: Adding and Removing Elements

```javascript
// ── Adding content INSIDE an element ──────────────────────────

// .append() — adds INSIDE, at the END
$("#list").append("<li>New Item</li>")
// <ul id="list">
//   <li>Item 1</li>
//   <li>New Item</li>   ← added here

// .prepend() — adds INSIDE, at the START
$("#list").prepend("<li>First Item</li>")
// <ul id="list">
//   <li>First Item</li>  ← added here
//   <li>Item 1</li>

// ── Adding content OUTSIDE/AROUND an element ──────────────────

// .after() — adds a sibling AFTER the element
$("#saveBtn").after('<button id="cancelBtn">Cancel</button>')

// .before() — adds a sibling BEFORE the element
$("#saveBtn").before('<span class="label">Action: </span>')

// ── Removing elements ─────────────────────────────────────────
$("#row5").remove()            // removes element AND its event handlers
$("#row5").detach()            // removes but KEEPS event handlers
                               // (use detach if you'll re-add it later)

// ── Emptying an element ───────────────────────────────────────
$("#tableBody").empty()        // removes all children, keeps the element itself

// ── Real example: build a table from data ─────────────────────
function renderEmployeeTable(employees) {
    var tbody = $("#empTableBody");
    tbody.empty();   // clear old rows first

    employees.forEach(function(emp) {
        var row = $("<tr>")
            .attr("data-id", emp.id)
            .append($("<td>").text(emp.name))
            .append($("<td>").text(emp.department))
            .append($("<td>").text("$" + emp.salary.toLocaleString()))
            .append(
                $("<td>").append(
                    $("<button>")
                        .addClass("k-button")
                        .text("Edit")
                        .on("click", function() { editEmployee(emp.id); })
                )
            );
        tbody.append(row);
    });
}
```

---

## 💻 Real Project Patterns — What You'll Actually Use

### Pattern 1: Collect form data and send via AJAX

```javascript
function getFormData() {
    return {
        name:         $("#Name").val().trim(),
        departmentId: parseInt($("#DepartmentId").val()),
        salary:       parseFloat($("#Salary").val()),
        isActive:     $("#IsActive").is(":checked"),
        hireDate:     $("#HireDate").val()
    };
}
```

### Pattern 2: Show/hide sections based on dropdown

```javascript
$("#EmploymentType").on("change", function() {
    var type = $(this).val();

    if (type === "Contract") {
        $("#contractFields").show();
        $("#permanentFields").hide();
    } else {
        $("#contractFields").hide();
        $("#permanentFields").show();
    }
});
```

### Pattern 3: Display server response in the page

```javascript
$.post("/Employee/Save", formData, function(response) {
    if (response.success) {
        $("#messageBox")
            .removeClass("error")
            .addClass("success")
            .text("Employee saved successfully!")
            .show();
    } else {
        $("#messageBox")
            .removeClass("success")
            .addClass("error")
            .html("Error: " + response.message)
            .show();
    }
});
```

### Pattern 4: Update a specific Grid row after save

```javascript
function updateRowDisplay(empId, updatedData) {
    // Find the row with data-id matching the employee
    var row = $("tr[data-id='" + empId + "']");

    // Update its cells
    row.find("td:eq(0)").text(updatedData.name);
    row.find("td:eq(1)").text(updatedData.department);
    row.find("td:eq(2)").text("$" + updatedData.salary.toLocaleString());
}
```

---

## 📊 Method Quick Reference

| Category             | Method                 | What It Does                                 |
| -------------------- | ---------------------- | -------------------------------------------- |
| **Content**    | `.text()`            | Get/set plain text                           |
|                      | `.html()`            | Get/set HTML content                         |
|                      | `.val()`             | Get/set form field value                     |
| **Attributes** | `.attr(key)`         | Get/set HTML attribute                       |
|                      | `.removeAttr(key)`   | Remove an attribute                          |
|                      | `.prop(key)`         | Get/set boolean property (checked, disabled) |
|                      | `.data(key)`         | Get/set data-* attribute (type-safe)         |
| **Styles**     | `.show()`            | Make element visible                         |
|                      | `.hide()`            | Make element invisible                       |
|                      | `.toggle()`          | Switch between show/hide                     |
|                      | `.css(key, val)`     | Get/set inline style                         |
| **Classes**    | `.addClass(name)`    | Add CSS class                                |
|                      | `.removeClass(name)` | Remove CSS class                             |
|                      | `.toggleClass(name)` | Add if absent, remove if present             |
|                      | `.hasClass(name)`    | Returns true/false                           |
| **Structure**  | `.append(html)`      | Add inside, at the end                       |
|                      | `.prepend(html)`     | Add inside, at the start                     |
|                      | `.after(html)`       | Add as next sibling                          |
|                      | `.before(html)`      | Add as previous sibling                      |
|                      | `.remove()`          | Remove element completely                    |
|                      | `.empty()`           | Remove all children                          |

---

## ❓ Interview Questions

**Q: What is the difference between `.text()` and `.html()`?**

> `.text()` reads or writes plain text — HTML tags are treated as literal characters and are not rendered. `.html()` reads or writes HTML — tags are parsed and rendered. Use `.text()` for user-generated content to prevent XSS attacks.

**Q: What is the difference between `.remove()` and `.detach()`?**

> Both remove the element from the DOM. `.remove()` also destroys all attached event handlers and data. `.detach()` preserves them — use `.detach()` when you plan to re-insert the element later and need the handlers to still work.

**Q: Why use `.addClass()` instead of `.css()`?**

> Separation of concerns — styles belong in CSS, logic belongs in JavaScript. `.addClass()` keeps styles in your stylesheet where they're easy to maintain. `.css()` mixes styles into JavaScript and is harder to manage.

**Q: What is the difference between `.attr()` and `.prop()`?**

> `.attr()` works with HTML attributes — string values like `id`, `name`, `href`. `.prop()` works with DOM properties — boolean states like `checked`, `disabled`, `selected`. For checkboxes, always use `.prop("checked")` not `.attr("checked")`.
>
