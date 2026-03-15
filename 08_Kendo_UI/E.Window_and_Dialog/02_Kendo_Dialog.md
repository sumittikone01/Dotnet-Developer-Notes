
# 02 — Kendo Dialog

---

## 🎯 One-Line Definition

> **The Kendo Dialog is a simplified modal popup designed for focused interactions — confirm/cancel prompts, alert messages, and short forms — always blocking the background and always centered on screen.**

---

## 🔑 Dialog vs Window — Know the Difference

```
┌──────────────────────────────────────────────────────────────┐
│  KENDO WINDOW                    KENDO DIALOG                │
│  ─────────────────               ────────────────────────    │
│  Draggable by title bar          NOT draggable               │
│  Resizable by edges              NOT resizable               │
│  Can be non-modal                ALWAYS modal                │
│  AJAX content loading            Simple content / form       │
│  Minimize / Maximize             No minimize / maximize      │
│  Full featured panel             Focused interaction only    │
│                                                              │
│  Use for: detail views,          Use for: confirm/cancel,    │
│  edit forms, dashboards          alerts, short prompts       │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔑 Built-in Global Dialogs — kendo.alert / confirm / prompt

Kendo provides three drop-in replacements for the browser's ugly built-in dialogs:

### `kendo.alert()` — Show a message

```javascript
// Replace the ugly browser alert()
// kendo.alert() returns a Promise

kendo.alert("Employee saved successfully!")
    .then(function() {
        // User clicked OK
        console.log("User acknowledged");
    });
```

### `kendo.confirm()` — Ask yes or no

```javascript
// Replace the ugly browser confirm()

kendo.confirm("Are you sure you want to delete this employee?")
    .then(function() {
        // ✅ User clicked OK / Yes
        deleteEmployee(empId);
        showNotification("Employee deleted", "success");
    })
    .fail(function() {
        // ❌ User clicked Cancel / No
        console.log("Delete cancelled");
    });
```

### `kendo.prompt()` — Ask for input

```javascript
// Replace the ugly browser prompt()

kendo.prompt("Enter a reason for the salary change:", "")
    .then(function(reason) {
        // User entered text and clicked OK
        console.log("Reason:", reason);
        saveSalaryChange(empId, newSalary, reason);
    })
    .fail(function() {
        // User clicked Cancel
        console.log("Cancelled");
    });
```

---

## 🔑 Custom Kendo Dialog Widget

For dialogs beyond alert/confirm/prompt, create a full `kendoDialog`:

```cshtml
{{!-- The dialog container --}}
<div id="deleteConfirmDialog"></div>
```

```javascript
$(function() {
    $("#deleteConfirmDialog").kendoDialog({
        title:   "Confirm Delete",
        width:   "450px",
        visible: false,    // start hidden
        modal:   true,     // always true for dialogs

        // Dialog body content
        content: "<p>Are you sure you want to delete this employee?</p>" +
                 "<p><strong>This action cannot be undone.</strong></p>",

        // Action buttons at the bottom
        actions: [
            {
                text:    "Delete",
                primary: true,        // highlighted/primary style
                action:  function(e) {
                    // Called when Delete is clicked
                    confirmDelete();
                    // Return false to KEEP dialog open
                    // Return true (or nothing) to CLOSE dialog
                }
            },
            {
                text:   "Cancel",
                action: function(e) {
                    console.log("Cancelled");
                    // Dialog closes automatically
                }
            }
        ],

        // Events
        open:  function() { console.log("Dialog opened"); },
        close: function() { console.log("Dialog closed"); }
    });
});

var currentEmpId;

function openDeleteDialog(empId) {
    currentEmpId = empId;
    var dialog = $("#deleteConfirmDialog").data("kendoDialog");
    dialog.open();
}

function confirmDelete() {
    $.post("/Employee/Delete", { id: currentEmpId }, function(r) {
        if (r.success) {
            $("#employeeGrid").data("kendoGrid").dataSource.read();
            showNotification("Employee deleted", "success");
        }
    });
}
```

---

## 🔑 Dialog with Dynamic Content

Change the dialog's content and title before opening:

```javascript
function openDeleteDialog(empId, empName) {
    var dialog = $("#deleteConfirmDialog").data("kendoDialog");

    // Update content dynamically
    dialog.content(
        "<p>Are you sure you want to delete <strong>" + empName + "</strong>?</p>" +
        "<p class='text-danger'>Employee ID: " + empId + " — This cannot be undone.</p>"
    );

    dialog.title("Delete: " + empName);

    // Store ID for the confirm action
    dialog.element.data("empId", empId);

    dialog.open();
}
```

---

## 🔑 Dialog with a Form

For short forms that need a focused interaction (not a full Window):

```cshtml
<div id="reasonDialog">
    <div style="padding:10px">
        <label>Reason for salary change: *</label>
        <textarea id="changeReason"
                  class="k-textarea"
                  rows="3"
                  style="width:100%;margin-top:8px"
                  placeholder="Enter reason...">
        </textarea>
        <span id="reasonError" class="text-danger" style="display:none">
            Reason is required
        </span>
    </div>
</div>
```

```javascript
$("#reasonDialog").kendoDialog({
    title:   "Salary Change Reason",
    width:   "450px",
    visible: false,
    modal:   true,
    actions: [
        {
            text:    "Confirm Change",
            primary: true,
            action:  function() {
                var reason = $("#changeReason").val().trim();

                if (!reason) {
                    $("#reasonError").show();
                    return false;  // return false = keep dialog OPEN
                }

                $("#reasonError").hide();
                applySalaryChange(reason);
                // return true (default) = close dialog
            }
        },
        { text: "Cancel" }
    ],
    close: function() {
        // Reset form when dialog closes
        $("#changeReason").val("");
        $("#reasonError").hide();
    }
});
```

---

## 🔑 Key Dialog Options

```javascript
$("#myDialog").kendoDialog({
    title:      "Dialog Title",
    width:      "450px",
    height:     "auto",         // auto = grows with content
    visible:    false,
    modal:      true,           // always true for dialogs
    closable:   true,           // show the X button
    content:    "<p>Content</p>",
    actions: [
        { text: "OK",     primary: true,  action: onOK     },
        { text: "Cancel",               action: onCancel }
    ]
});
```

| Option       | What It Does     | Notes                              |
| ------------ | ---------------- | ---------------------------------- |
| `title`    | Title bar text   | Can be changed dynamically         |
| `width`    | Dialog width     | `"450px"`is a good default       |
| `height`   | Dialog height    | `"auto"`grows with content       |
| `visible`  | Show on init     | `false`= open manually           |
| `modal`    | Block background | Always `true`for Dialog          |
| `closable` | Show X button    | `false`to force a button choice  |
| `content`  | Body HTML        | Can be changed with `.content()` |
| `actions`  | Bottom buttons   | Array of button configs            |

---

## 🔑 Dialog JavaScript API

```javascript
var dialog = $("#myDialog").data("kendoDialog");

// Open / Close
dialog.open()
dialog.close()

// Update content
dialog.content("<p>New content</p>")
dialog.content()  // get current content

// Update title
dialog.title("New Title")

// Destroy
dialog.destroy()
```

---

## 🔑 Dialog Events

```javascript
$("#myDialog").kendoDialog({
    // ...options...

    open: function() {
        // Focus first input when dialog opens
        this.element.find("input:first").focus();
    },

    close: function() {
        // Reset any form fields
        this.element.find("input, textarea").val("");
    },

    initOpen: function() {
        // Fires only the FIRST time the dialog opens
        console.log("Dialog initialised and opened for first time");
    }
});
```

---

## 🔑 `kendo.confirm()` for Grid Delete — The Best Pattern

Use `kendo.confirm()` before any destructive action in the grid:

```javascript
$(document).on("click", ".deleteBtn", function() {
    var row   = $(this).closest("tr");
    var grid  = $("#employeeGrid").data("kendoGrid");
    var item  = grid.dataItem(row);

    kendo.confirm(
        "Delete <strong>" + item.Name + "</strong>?" +
        "<br><small>Department: " + item.Department + "</small>"
    )
    .then(function() {
        // User confirmed — remove via DataSource
        grid.dataSource.remove(item);
        grid.dataSource.sync();
    });
    // .fail() is optional — nothing needed for cancel
});
```

---

## ⚠️ Common Mistakes

| Mistake                                         | Symptom                                     | Fix                                                                           |
| ----------------------------------------------- | ------------------------------------------- | ----------------------------------------------------------------------------- |
| Returning `false`without meaning to keep open | Dialog keeps open after clicking OK         | Return `false`only when you want to PREVENT close (e.g., validation failed) |
| Using Dialog for complex edit forms             | Dialog feels cramped                        | Use Window for complex multi-field forms                                      |
| `kendo.confirm()`without `.then()`          | Delete runs immediately before user answers | Always chain `.then()`for the confirmed action                              |
| `closable: false`with no cancel button        | User trapped — can't close dialog          | Always have at least one exit: cancel button OR X button                      |

---

## ❓ Interview Questions

**Q: What is the difference between Kendo Window and Kendo Dialog?**

> Window is a full-featured floating panel — draggable, resizable, can be non-modal, supports AJAX content. Dialog is simpler — always modal, always centered, not draggable, designed for focused interactions like confirm/cancel prompts.

**Q: What do `kendo.alert()`, `kendo.confirm()`, and `kendo.prompt()` replace?**

> They are Promise-based replacements for the browser's built-in `alert()`, `confirm()`, and `prompt()`. They use Kendo styling, support HTML content, and work with `.then()` / `.fail()` for the user's response.

**Q: How do you keep a Dialog open after clicking an action button?**

> Return `false` from the button's `action` function. Returning `true` or `undefined` closes the dialog. Return `false` when validation fails and you want to show an error and keep the form visible.

**Q: When should you use Dialog instead of Window?**

> Dialog is best for short, focused interactions: confirm before delete, alert messages, short prompts. Window is better for larger content like detail views, edit forms with many fields, or anything that benefits from being draggable and resizable.
>
