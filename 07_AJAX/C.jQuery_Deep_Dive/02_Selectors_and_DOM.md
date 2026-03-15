
# 02 — Selectors and DOM

---

## 🎯 One-Line Definition

> **A jQuery selector is the text inside `$("")` that tells jQuery which HTML elements to find — works just like CSS selectors, but gives you jQuery methods to work with the results.**

---

## 🏗️ What is the DOM?

Before selectors, understand what you're selecting from.

```
Your HTML file:
────────────────────────────────────────────
<html>
  <body>
    <div id="container">
      <h1 class="title">Employee List</h1>
      <table id="empTable">
        <tr class="row active">
          <td>Alice</td>
          <td class="dept">IT</td>
        </tr>
        <tr class="row">
          <td>Bob</td>
          <td class="dept">HR</td>
        </tr>
      </table>
      <button id="saveBtn" type="button">Save</button>
    </div>
  </body>
</html>


The DOM (Document Object Model):
────────────────────────────────────────────
The browser converts your HTML into a TREE
of objects (nodes) in memory.

document
└── html
    └── body
        └── div#container
            ├── h1.title
            ├── table#empTable
            │   ├── tr.row.active
            │   │   ├── td  ("Alice")
            │   │   └── td.dept  ("IT")
            │   └── tr.row
            │       ├── td  ("Bob")
            │       └── td.dept  ("HR")
            └── button#saveBtn
```

**jQuery selectors navigate this tree to find elements.**

---

## 🔑 The 3 Basic Selectors — Use These 90% of the Time

```javascript
// ── By ID  →  "#idName" ───────────────────────────────────────
// Finds the ONE element with that id
// Fastest selector — IDs are unique

$("#saveBtn")          // finds <button id="saveBtn">
$("#empTable")         // finds <table id="empTable">
$("#container")        // finds <div id="container">


// ── By Class  →  ".className" ────────────────────────────────
// Finds ALL elements with that class

$(".row")              // finds both <tr class="row ...">
$(".dept")             // finds both <td class="dept">
$(".active")           // finds <tr class="row active">


// ── By Tag  →  "tagName" ──────────────────────────────────────
// Finds ALL elements of that HTML tag

$("td")                // finds all 4 <td> elements
$("button")            // finds the <button>
$("tr")                // finds both <tr> elements
```

---

## 🔑 Combining Selectors

```javascript
// ── Tag + Class  →  "tag.class" ──────────────────────────────
$("tr.active")         // only <tr> elements that also have class "active"
$("td.dept")           // only <td> elements with class "dept"


// ── Multiple selectors  →  "a, b, c" ─────────────────────────
// Comma = OR — finds elements matching ANY of them
$("h1, h2, h3")        // all heading elements
$(".row, .active")     // elements with either class


// ── Descendant  →  "parent child" ────────────────────────────
// Space = find child ANYWHERE inside parent
$("#empTable td")      // all <td> inside #empTable
$(".row td.dept")      // <td class="dept"> inside any .row


// ── Direct Child  →  "parent > child" ────────────────────────
// Arrow = find DIRECT children only (not grandchildren)
$("#container > button")   // <button> that is a direct child of #container
```

---

## 🔑 Attribute Selectors

```javascript
// Find elements by their HTML attributes
$("input[type='text']")        // all text inputs
$("input[type='checkbox']")    // all checkboxes
$("a[href]")                   // all links that have an href
$("input[name='Email']")       // input with name="Email"
$("[data-id='5']")             // any element with data-id="5"
```

---

## 🔑 Filter Selectors — `:something`

```javascript
// ── Position filters ─────────────────────────────────────────
$("tr:first")          // first <tr>
$("tr:last")           // last <tr>
$("tr:eq(1)")          // <tr> at index 1 (second row — 0-based)
$("li:even")           // even-indexed list items (0, 2, 4...)
$("li:odd")            // odd-indexed list items (1, 3, 5...)

// ── State filters ─────────────────────────────────────────────
$(":checked")          // checked checkboxes and radio buttons
$(":disabled")         // disabled form fields
$(":enabled")          // enabled form fields
$(":visible")          // elements that are visible
$(":hidden")           // elements that are hidden
$(":focus")            // the currently focused element

// ── Content filters ───────────────────────────────────────────
$("td:contains('IT')")  // <td> cells containing the text "IT"
$("tr:has(.active)")    // <tr> that contains an element with class "active"
$("p:empty")            // <p> tags with no content
```

---

## 🔑 Traversal — Moving Around the DOM

Once you have an element, you can navigate to its relatives:

```javascript
var row = $("tr.active");   // start here

// ── Moving UP the tree ────────────────────────────────────────
row.parent()               // the direct parent element (<table>)
row.parents()              // ALL ancestors (table, div, body, html)
row.parents("div")         // only ancestor <div> elements
row.closest(".container")  // nearest ancestor with class "container"
                           // (closest walks up and checks each parent)

// ── Moving DOWN the tree ──────────────────────────────────────
row.children()             // direct children only (the <td> cells)
row.children(".dept")      // direct children with class "dept"
row.find("td")             // ALL descendants that are <td>
row.find(".dept")          // all descendants with class "dept"

// ── Moving SIDEWAYS ───────────────────────────────────────────
row.next()                 // the next sibling element
row.prev()                 // the previous sibling element
row.siblings()             // all siblings (not itself)
row.siblings(".active")    // siblings with class "active"
```

---

## 🔑 Filtering a Selection

After selecting a group, narrow it down:

```javascript
// Start with all rows
var rows = $("tr");

rows.filter(".active")     // keep only rows with class "active"
rows.filter(":visible")    // keep only visible rows
rows.not(".active")        // remove rows with class "active"
rows.first()               // keep only the first row
rows.last()                // keep only the last row
rows.eq(2)                 // keep only the row at index 2
rows.has("td.dept")        // keep rows that contain a .dept cell
```

---

## 🔑 Reading Information About Elements

```javascript
var btn = $("#saveBtn");

// ── What is it? ───────────────────────────────────────────────
btn.is("button")           // true — is it a button?
btn.is(".active")          // true/false — does it have this class?
btn.is(":visible")         // true/false — is it visible?
btn.hasClass("active")     // true/false — cleaner than .is(".active")
btn.length                 // 1 — how many elements matched

// ── Get HTML attributes ───────────────────────────────────────
btn.attr("type")           // "button"
btn.attr("id")             // "saveBtn"

// ── Get data attributes ───────────────────────────────────────
// <tr data-employee-id="5">
$("tr").attr("data-employee-id")   // "5" (string)
$("tr").data("employeeId")         // 5   (number — jQuery converts type)
$("tr").data("employee-id")        // 5   (also works)
```

---

## 💻 Real Examples — Selectors in MVC + Kendo Projects

```javascript
$(function() {

    // Get value from a form input
    var name    = $("#Name").val();
    var deptId  = $("#DepartmentId").val();

    // Get value from a Kendo DropDownList
    var ddl     = $("#DepartmentDdl").data("kendoDropDownList");
    var deptVal = ddl.value();

    // Read a data attribute from a row to get its ID
    // <tr data-id="5"> ... </tr>
    var empId = $(this).closest("tr").data("id");

    // Check if a checkbox is checked
    var isActive = $("#IsActive").is(":checked");

    // Find all visible rows in the grid table
    var visibleRows = $("#empTable tr:visible");

    // Get all checked checkboxes and collect their values
    var selectedIds = [];
    $("input[type='checkbox']:checked").each(function() {
        selectedIds.push($(this).val());
    });

});
```

---

## 📊 Selector Speed — From Fastest to Slowest

```
Fastest  →  Slowest
────────────────────────────────────────────────────────────
#id           →  $("#saveBtn")     ID lookup is instant
tag           →  $("td")           uses native getElementsByTagName
.class        →  $(".row")         uses native getElementsByClassName
tag.class     →  $("tr.active")    tag first, then filter by class
[attribute]   →  $("[type='text']")slower — checks every element
:filter       →  $("tr:odd")       slowest — evaluated by jQuery
```

> **Tip:** For best performance, always narrow your search. `$("#table td")` is faster than `$("td")` because it starts from a specific element.

---

## ❓ Interview Questions

**Q: What is a jQuery selector?**

> The string inside `$("")` that identifies which HTML elements to find. It uses CSS-style syntax — `#id`, `.class`, `tag`, and combinations — and returns a jQuery object containing all matched elements.

**Q: What is the difference between `.find()` and `.children()`?**

> `.children()` returns only the direct children (one level down). `.find()` searches all descendants at any depth. `$("#table").children("tr")` finds direct `tr` children; `$("#table").find("td")` finds all `td` cells at any nesting level.

**Q: What is the difference between `.parent()` and `.closest()`?**

> `.parent()` returns only the immediate parent element. `.closest(selector)` walks up the DOM tree from the element and returns the first ancestor that matches the selector — useful when you don't know exactly how many levels up the target is.

**Q: What does `.data("key")` do differently than `.attr("data-key")`?**

> `.attr("data-employee-id")` always returns a string. `.data("employeeId")` reads the same attribute but automatically converts it to the correct JavaScript type — numbers become numbers, booleans become booleans.
>
