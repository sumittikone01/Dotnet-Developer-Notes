
# 01 — Kendo Window

---

## 🎯 One-Line Definition

> **The Kendo Window is a floating, draggable popup panel that can display static HTML, load AJAX content, or act as a modal overlay — all without navigating away from the current page.**

---

## 🔑 What the Kendo Window Looks Like

```
┌─────────────────────────────────────────────────────────┐
│  Employee Details                          [─][□][×]    │  ← title bar
├─────────────────────────────────────────────────────────┤
│                                                         │
│   Name:        Alice Johnson                            │
│   Department:  IT                                       │
│   Salary:      $75,000                                  │
│   Hire Date:   06/15/2021                               │
│                                                         │
│                                [Close]                  │
└─────────────────────────────────────────────────────────┘
  ↑ draggable by title bar
  ↑ resizable by edges
  ↑ [─] minimize  [□] maximize  [×] close
  ↑ can block background with modal: true
```

---

## 🔑 Basic Setup — Static Content

```cshtml
{{!-- Container div — content lives here --}}
<div id="employeeWindow">
    <p>This content shows inside the window.</p>
</div>

<button id="openWindowBtn">View Details</button>
```

```javascript
$(function() {
    // Create the Kendo Window
    $("#employeeWindow").kendoWindow({
        title:    "Employee Details",
        width:    "600px",
        height:   "400px",
        visible:  false,   // start hidden — open manually
        modal:    false,   // true = blocks background
        actions:  ["Minimize", "Maximize", "Close"],
        draggable:  true,
        resizable:  true
    });

    // Open it when button is clicked
    $("#openWindowBtn").on("click", function() {
        var win = $("#employeeWindow").data("kendoWindow");
        win.open().center();   // open AND centre on screen
    });
});
```

---

## 🔑 Key Configuration Options

```javascript
$("#myWindow").kendoWindow({
    title:      "Window Title",
    width:      "600px",
    height:     "400px",
    minWidth:   300,
    minHeight:  200,
    visible:    false,         // hidden on init
    modal:      true,          // blocks background
    draggable:  true,          // drag by title bar
    resizable:  true,          // drag edges to resize
    scrollable: true,          // scrollbar if content overflows
    pinned:     false,         // true = stays fixed on page scroll
    animation: {
        open:  { effects: "fade:in",  duration: 200 },
        close: { effects: "fade:out", duration: 200 }
    },
    actions: ["Minimize", "Maximize", "Close"]
    // actions: ["Close"]   → only X button
    // actions: []          → no buttons at all
});
```

| Option        | What It Does      | Common Value                         |
| ------------- | ----------------- | ------------------------------------ |
| `title`     | Text in title bar | `"Employee Details"`               |
| `width`     | Window width      | `"600px"`                          |
| `height`    | Window height     | `"400px"`                          |
| `visible`   | Show on init      | `false`                            |
| `modal`     | Block background  | `true`for critical dialogs         |
| `draggable` | Drag by title bar | `true`                             |
| `resizable` | Resize by edges   | `true`                             |
| `actions`   | Title bar buttons | `["Minimize","Maximize","Close"]`  |
| `animation` | Open/close effect | `{ open: { effects: "fade:in" } }` |

---

## 🔑 JavaScript API

```javascript
var win = $("#employeeWindow").data("kendoWindow");

// ── Open and Close ─────────────────────────────────────────
win.open()              // show the window
win.close()             // hide the window
win.open().center()     // open AND centre on screen (most common)

// ── Position ───────────────────────────────────────────────
win.center()
win.setOptions({ position: { top: 100, left: 200 } })

// ── Size ───────────────────────────────────────────────────
win.setOptions({ width: 800, height: 500 })
win.maximize()          // fill the screen
win.minimize()          // collapse to title bar
win.restore()           // restore from min/max

// ── Title and Content ──────────────────────────────────────
win.title("New Title")                   // change title
win.content("<p>New HTML content</p>")   // replace body HTML
win.content()                            // get current HTML

// ── Destroy ────────────────────────────────────────────────
win.destroy()           // remove window widget completely
```

---

## 🔑 Loading AJAX Content — The Main Use Case

Load a Partial View from the server when the window opens:

```javascript
function openEmployeeDetails(empId) {
    var win = $("#employeeWindow").data("kendoWindow");

    // Load partial view via AJAX into the window body
    win.refresh({
        url: "/Employee/Details/" + empId
    });

    win.title("Employee Details");
    win.open().center();
}
```

```csharp
// Controller — returns a Partial View, NOT a full page
public IActionResult Details(int id)
{
    var emp = _db.Employees.Find(id);
    if (emp == null) return NotFound();

    return PartialView("_EmployeeDetails", emp);
    //     ↑ PartialView — no _Layout, just the HTML fragment
}
```

```cshtml
@* Views/Employee/_EmployeeDetails.cshtml *@
@model Employee

<div style="padding:20px;">
    <table class="table table-borderless">
        <tr><th>Name:</th>       <td>@Model.Name</td></tr>
        <tr><th>Department:</th> <td>@Model.Department</td></tr>
        <tr><th>Salary:</th>     <td>@Model.Salary.ToString("C0")</td></tr>
        <tr><th>Hire Date:</th>  <td>@Model.HireDate.ToString("MM/dd/yyyy")</td></tr>
    </table>
    <button class="k-button" onclick="$('#employeeWindow').data('kendoWindow').close()">
        Close
    </button>
</div>
```

---

## 🔑 Edit Form in a Window — Full Pattern

```javascript
function openEditWindow(empId) {
    var win = $("#employeeWindow").data("kendoWindow");

    win.title(empId ? "Edit Employee" : "Add New Employee");
    win.refresh({ url: empId ? "/Employee/EditForm/" + empId
                              : "/Employee/CreateForm" });
    win.setOptions({ width: 700, height: 550 });
    win.open().center();
}

// Call this from inside the loaded partial view after successful save
function onWindowFormSaved() {
    $("#employeeWindow").data("kendoWindow").close();
    $("#employeeGrid").data("kendoGrid").dataSource.read();
    showNotification("Saved successfully!", "success");
}
```

```csharp
public IActionResult EditForm(int? id)
{
    var emp = id.HasValue ? _db.Employees.Find(id) : new Employee();
    return PartialView("_EditForm", emp);
}

[HttpPost]
public JsonResult SaveForm(Employee employee)
{
    if (!ModelState.IsValid)
        return Json(new { success = false });

    if (employee.Id > 0) _db.Employees.Update(employee);
    else                  _db.Employees.Add(employee);

    _db.SaveChanges();
    return Json(new { success = true });
}
```

---

## 🔑 Window Events

```javascript
var win = $("#employeeWindow").data("kendoWindow");

// Fires when window opens
win.bind("open", function() {
    $(this.element).find("input:first").focus(); // auto-focus first field
});

// Fires when window closes
win.bind("close", function() {
    this.content(""); // clear content — prevents stale data flash on reopen
});

// Fires after window finishes opening (animation done)
win.bind("activate", function() {
    // Init Kendo widgets inside AJAX-loaded content
    kendo.init(this.element);
});

// Fires while user drags edges to resize
win.bind("resize", function() {
    console.log("New size:", this.element.outerWidth(), "x", this.element.outerHeight());
});
```

---

## ⚠️ Common Mistakes

| Mistake                                       | Symptom                                       | Fix                                                      |
| --------------------------------------------- | --------------------------------------------- | -------------------------------------------------------- |
| `open()`then `center()`on separate lines  | Window flashes then jumps                     | Chain:`win.open().center()`                            |
| `return View()`instead of `PartialView()` | Full page (with layout) loads inside window   | Always `PartialView()`for AJAX window content          |
| Not clearing content on close                 | Old employee data briefly shows when reopened | Bind `close`event and call `this.content("")`        |
| Kendo widgets in AJAX content don't work      | Raw `<input>`instead of DatePicker/DDL      | Call `kendo.init(win.element)`in the `activate`event |
| Window opens multiple times, stacks up        | Multiple windows stacking                     | Check first:`if (win.options.visible) return;`         |

---

## ❓ Interview Questions

**Q: What is the Kendo Window?**

> A floating panel that can display static HTML or AJAX-loaded partial views. Supports dragging, resizing, minimize/maximize/close, and modal mode to block background interaction.

**Q: How do you load a partial view into a Kendo Window?**

> Call `win.refresh({ url: "/Controller/Action/id" })` — the controller returns `PartialView(...)` (not a full `View()`), and the HTML fragment is inserted into the window body.

**Q: What does `modal: true` do?**

> It shows a dark semi-transparent overlay behind the window that blocks all user interaction with the page. The user must close the window before they can interact with anything underneath.

**Q: Why call `kendo.init(win.element)` in the activate event?**

> AJAX-loaded content contains raw HTML — Kendo's page-load initialization hasn't run on it. `kendo.init()` scans the container and initializes any Kendo widget markup (DatePicker, DropDownList etc.) found inside.
>
