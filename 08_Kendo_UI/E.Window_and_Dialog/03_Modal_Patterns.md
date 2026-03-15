# 03 — Modal Patterns

---

## 🎯 One-Line Definition

> **Modal patterns are the practical, reusable ways you combine Window and Dialog in real projects — open for view, open for edit, confirm before delete, multi-step form, and single shared window for the whole page.**

---

## 🔑 The 5 Core Modal Patterns

```
┌──────────────────────────────────────────────────────────────┐
│  Pattern 1: View Details                                     │
│  Click row → Window opens with readonly employee info        │
├──────────────────────────────────────────────────────────────┤
│  Pattern 2: Edit in Window                                   │
│  Click Edit → Window opens with editable form                │
├──────────────────────────────────────────────────────────────┤
│  Pattern 3: Confirm Before Delete                            │
│  Click Delete → Dialog asks "Are you sure?"                  │
├──────────────────────────────────────────────────────────────┤
│  Pattern 4: Single Shared Window                             │
│  One window, reused for both View and Edit                   │
├──────────────────────────────────────────────────────────────┤
│  Pattern 5: Multi-Step Form                                  │
│  Step 1 → Step 2 → Step 3 inside one Window                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 Pattern 1 — View Details Window

User clicks a View button → Window loads the employee's full details:

```cshtml
{{!-- Grid column with View button --}}
<column title="Actions" width="200">
    <column-template>
        <button class="k-button k-button-sm viewBtn" data-id="#= Id #">
            👁 View
        </button>
        <button class="k-button k-button-sm k-button-solid-primary editBtn"
                data-id="#= Id #">
            ✏ Edit
        </button>
    </column-template>
</column>

{{!-- Shared window container --}}
<div id="employeeWindow"></div>
```

```javascript
$(function() {

    // Create the window once
    $("#employeeWindow").kendoWindow({
        title:    "Employee Details",
        width:    "650px",
        height:   "auto",
        visible:  false,
        modal:    true,
        resizable: false,
        draggable: false,
        actions:  ["Close"]
    });

    // View button
    $(document).on("click", ".viewBtn", function() {
        var empId = $(this).data("id");
        var win   = $("#employeeWindow").data("kendoWindow");

        win.title("Employee Details");
        win.refresh({ url: "/Employee/Details/" + empId });
        win.open().center();
    });

    // Edit button
    $(document).on("click", ".editBtn", function() {
        var empId = $(this).data("id");
        var win   = $("#employeeWindow").data("kendoWindow");

        win.title("Edit Employee");
        win.setOptions({ width: "700px" });
        win.refresh({ url: "/Employee/EditForm/" + empId });
        win.open().center();
    });
});
```

---

## 🔑 Pattern 2 — Add New in Window (Toolbar Button)

```javascript
// Called when + Add New toolbar button in grid is overridden
// OR from a button outside the grid
function openAddNewWindow() {
    var win = $("#employeeWindow").data("kendoWindow");

    win.title("Add New Employee");
    win.setOptions({ width: "700px", height: "550px" });
    win.refresh({ url: "/Employee/CreateForm" });
    win.open().center();
}
```

```cshtml
{{!-- Override the Grid's default create action --}}
<toolbar>
    <toolbar-button template="<button class='k-button k-button-solid-primary'
                                       onclick='openAddNewWindow()'>
                                  + Add New Employee
                              </button>" />
</toolbar>

{{!-- No <editable> needed if using Window for create --}}
```

---

## 🔑 Pattern 3 — Confirm Before Delete (Three Styles)

### Style A — `kendo.confirm()` (simplest)

```javascript
$(document).on("click", ".deleteBtn", function() {
    var row  = $(this).closest("tr");
    var grid = $("#employeeGrid").data("kendoGrid");
    var item = grid.dataItem(row);

    kendo.confirm("Delete <strong>" + item.Name + "</strong>?")
        .then(function() {
            grid.dataSource.remove(item);
            grid.dataSource.sync();
        });
});
```

### Style B — Custom Dialog with rich content

```javascript
var empToDelete = null;

// Create dialog once
$("#deleteDialog").kendoDialog({
    title:   "Confirm Delete",
    width:   "420px",
    visible: false,
    modal:   true,
    actions: [
        {
            text:    "Yes, Delete",
            primary: true,
            action: function() {
                $.post("/Employee/Delete", { id: empToDelete.Id })
                    .done(function() {
                        $("#employeeGrid").data("kendoGrid").dataSource.read();
                        showNotification("Employee deleted", "success");
                    });
            }
        },
        { text: "Cancel" }
    ]
});

$(document).on("click", ".deleteBtn", function() {
    var row  = $(this).closest("tr");
    var grid = $("#employeeGrid").data("kendoGrid");
    empToDelete = grid.dataItem(row);

    var dialog = $("#deleteDialog").data("kendoDialog");
    dialog.content(
        "<div style='text-align:center;padding:10px'>" +
            "<p style='font-size:16px'>Delete <strong>" + empToDelete.Name + "</strong>?</p>" +
            "<p style='color:#888'>Department: " + empToDelete.Department + "</p>" +
            "<p style='color:#e74c3c'>⚠ This cannot be undone.</p>" +
        "</div>"
    );
    dialog.open();
});
```

### Style C — Grid's built-in `remove` event with custom confirm

```javascript
var grid = $("#employeeGrid").data("kendoGrid");

grid.bind("remove", function(e) {
    e.preventDefault();   // stop automatic delete

    var item = e.model;

    kendo.confirm("Delete <strong>" + item.Name + "</strong>?")
        .then(function() {
            e.sender.dataSource.remove(item);
            e.sender.dataSource.sync();
        });
});
```

---

## 🔑 Pattern 4 — Single Shared Window (View + Edit)

One window reused for multiple purposes — the cleanest architecture:

```javascript
$(function() {

    // ── Single window for the whole page ─────────────────────
    var $win = $("#sharedWindow");

    $win.kendoWindow({
        visible:  false,
        modal:    true,
        actions:  ["Close"]
    });

    function getWin() {
        return $win.data("kendoWindow");
    }

    // ── Open in VIEW mode ────────────────────────────────────
    $(document).on("click", ".viewBtn", function() {
        var empId = $(this).data("id");
        var win   = getWin();
        win.title("Employee Details");
        win.setOptions({ width: "650px", height: "auto", resizable: false });
        win.refresh({ url: "/Employee/Details/" + empId });
        win.open().center();
    });

    // ── Open in EDIT mode ────────────────────────────────────
    $(document).on("click", ".editBtn", function() {
        var empId = $(this).data("id");
        var win   = getWin();
        win.title("Edit Employee");
        win.setOptions({ width: "720px", height: "560px", resizable: true });
        win.refresh({ url: "/Employee/EditForm/" + empId });
        win.open().center();
    });

    // ── Open for ADD NEW ─────────────────────────────────────
    $(document).on("click", "#addNewBtn", function() {
        var win = getWin();
        win.title("Add New Employee");
        win.setOptions({ width: "720px", height: "560px", resizable: true });
        win.refresh({ url: "/Employee/CreateForm" });
        win.open().center();
    });

    // ── Global: called from inside loaded partial views ───────
    window.closeSharedWindow = function() {
        getWin().close();
    };

    window.onWindowSaveSuccess = function() {
        getWin().close();
        $("#employeeGrid").data("kendoGrid").dataSource.read();
        showNotification("Changes saved successfully!", "success");
    };
});
```

```cshtml
{{!-- In the partial view (_EditForm.cshtml) --}}
<div style="padding:20px;">
    {{!-- ... form fields ... --}}
    <button class="k-button k-button-solid-primary" onclick="submitEditForm()">
        Save
    </button>
    <button class="k-button" onclick="closeSharedWindow()">
        Cancel
    </button>
</div>

@section Scripts {
<script>
function submitEditForm() {
    var validator = $("#editForm").data("kendoValidator");
    if (!validator || !validator.validate()) return;

    $.post("/Employee/Save", $("#editForm").serialize())
        .done(function(r) {
            if (r.success) {
                window.onWindowSaveSuccess();
            } else {
                showNotification(r.message, "error");
            }
        });
}
</script>
}
```

---

## 🔑 Pattern 5 — Multi-Step Form in a Window

Guide the user through steps inside one Window:

```cshtml
<div id="multiStepWindow">
    {{!-- Step 1 --}}
    <div id="step1" class="step-panel">
        <h4>Step 1 of 3 — Basic Info</h4>
        <div class="mb-3">
            <label>Full Name *</label>
            <input id="step1Name" class="k-textbox w-100" />
        </div>
        <div class="mb-3">
            <label>Email *</label>
            <input id="step1Email" type="email" class="k-textbox w-100" />
        </div>
        <button class="k-button k-button-solid-primary" onclick="goToStep(2)">
            Next →
        </button>
    </div>

    {{!-- Step 2 --}}
    <div id="step2" class="step-panel" style="display:none">
        <h4>Step 2 of 3 — Work Details</h4>
        <div class="mb-3">
            <label>Department</label>
            <input id="step2Dept" />  {{!-- becomes Kendo DDL --}}
        </div>
        <div class="mb-3">
            <label>Salary</label>
            <input id="step2Salary" /> {{!-- becomes Kendo NumericTextBox --}}
        </div>
        <button class="k-button" onclick="goToStep(1)">← Back</button>
        <button class="k-button k-button-solid-primary" onclick="goToStep(3)">
            Next →
        </button>
    </div>

    {{!-- Step 3 --}}
    <div id="step3" class="step-panel" style="display:none">
        <h4>Step 3 of 3 — Review</h4>
        <div id="reviewSummary"></div>
        <button class="k-button" onclick="goToStep(2)">← Back</button>
        <button class="k-button k-button-solid-success" onclick="submitEmployee()">
            ✓ Submit
        </button>
    </div>
</div>
```

```javascript
$(function() {
    // Initialize the multi-step window
    $("#multiStepWindow").kendoWindow({
        title:   "Add New Employee — Step 1",
        width:   "600px",
        height:  "400px",
        visible: false,
        modal:   true,
        actions: ["Close"]
    });

    // Initialize Kendo widgets in Step 2
    $("#step2Dept").kendoDropDownList({
        dataSource: ["IT", "HR", "Finance", "Sales"],
        optionLabel: "-- Select --"
    });
    $("#step2Salary").kendoNumericTextBox({ format: "c0", min: 0 });

    var currentStep = 1;

    window.goToStep = function(step) {
        // Validate current step before moving forward
        if (step > currentStep && !validateStep(currentStep)) return;

        $(".step-panel").hide();
        $("#step" + step).show();

        var win = $("#multiStepWindow").data("kendoWindow");
        win.title("Add New Employee — Step " + step + " of 3");

        if (step === 3) buildReviewSummary();

        currentStep = step;
    };

    function validateStep(step) {
        if (step === 1) {
            if (!$("#step1Name").val().trim()) {
                showNotification("Name is required", "error");
                return false;
            }
        }
        return true;
    }

    function buildReviewSummary() {
        var html = "<table class='table'>" +
            "<tr><td><b>Name:</b></td><td>" + $("#step1Name").val() + "</td></tr>" +
            "<tr><td><b>Email:</b></td><td>" + $("#step1Email").val() + "</td></tr>" +
            "<tr><td><b>Dept:</b></td><td>" +
                $("#step2Dept").data("kendoDropDownList").value() + "</td></tr>" +
            "<tr><td><b>Salary:</b></td><td>" +
                kendo.format("{0:c0}", $("#step2Salary").data("kendoNumericTextBox").value()) +
            "</td></tr></table>";
        $("#reviewSummary").html(html);
    }

    window.submitEmployee = function() {
        var payload = {
            name:       $("#step1Name").val(),
            email:      $("#step1Email").val(),
            department: $("#step2Dept").data("kendoDropDownList").value(),
            salary:     $("#step2Salary").data("kendoNumericTextBox").value()
        };

        $.post("/Employee/Create", payload)
            .done(function(r) {
                if (r.success) {
                    $("#multiStepWindow").data("kendoWindow").close();
                    $("#employeeGrid").data("kendoGrid").dataSource.read();
                    showNotification("Employee added!", "success");
                    // Reset steps
                    goToStep(1);
                }
            });
    };
});
```

---

## 🔑 Closing a Window from Inside a Partial View

```javascript
// In a partial view loaded into a Window —
// closing the PARENT window from inside the loaded content:

// Option A: reference by ID (simple)
function closeWindow() {
    $("#employeeWindow").data("kendoWindow").close();
}

// Option B: use a global function set by the parent page
function onSaved() {
    if (window.onWindowSaveSuccess) {
        window.onWindowSaveSuccess();   // defined in the parent page
    }
}

// Option C: traverse to parent window element
function closeMyWindow() {
    // Find the closest .k-window-content and get its window
    $(document).find(".k-window").first()
        .data("kendoWindow").close();
}
```

---

## 📊 Pattern Decision Guide

| Situation                         | Use                                    |
| --------------------------------- | -------------------------------------- |
| Show employee details (read-only) | Window + refresh (PartialView)         |
| Edit employee with many fields    | Window + refresh (PartialView form)    |
| "Are you sure?" before delete     | `kendo.confirm()`                    |
| Styled confirm with rich content  | Custom `kendoDialog`                 |
| Alert user of a result            | `kendo.alert()`                      |
| Ask user for a reason/input       | `kendo.prompt()`or Dialog with input |
| Complex guided data entry         | Multi-step Window                      |
| Reuse same window for view + edit | Single shared Window pattern           |

---

## ⚠️ Common Mistakes

| Mistake                                                               | Symptom                                          | Fix                                                                             |
| --------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------- |
| Creating window inside event handler                                  | New window created on every click, stacks up     | Create window ONCE in `$(function() {})`, open/close it                       |
| Not resetting multi-step form on close                                | Step 3 shows on second open                      | Reset `goToStep(1)`in the `close`event                                      |
| Calling `window.xxx`from partial view but parent function undefined | JS error:`window.xxx is not a function`        | Check `if (window.xxx)`before calling, or always define it on the parent page |
| `PartialView`includes `@section Scripts`                          | Scripts in sections are ignored in partial views | Move scripts into `<script>`tags directly in the partial                      |

---

## ❓ Interview Questions

**Q: When should you use a single shared window vs separate windows for view and edit?**

> A single shared window is cleaner — one DOM element, one init, reused for different purposes by changing the title, size, and loaded URL. Use separate windows only when view and edit have very different sizes or behaviors that are hard to toggle.

**Q: How do you close a Window from inside a partial view loaded into it?**

> Either reference the window by its known ID directly: `$("#employeeWindow").data("kendoWindow").close()`, or define a global callback function on the parent page that the partial view can call.

**Q: What are the three styles for delete confirmation and when do you use each?**

> `kendo.confirm()` for the simplest case. A custom `kendoDialog` for richer content (employee name, warning text, styled buttons). The grid's `remove` event with `e.preventDefault()` when you need to integrate with the grid's own delete flow.

**Q: How do you validate between steps in a multi-step form?**

> Before calling `goToStep(nextStep)`, run validation on the current step's fields. Return early (don't advance) if validation fails. Only proceed to the next step when the current step's data is valid.
>
