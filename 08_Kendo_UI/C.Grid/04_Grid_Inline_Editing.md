# 04 — Grid Inline Editing

---

## 🎯 One-Line Definition

> **Inline editing transforms the clicked row directly into a form — inputs appear inside the cells, and Update/Cancel buttons replace the Edit/Delete buttons, all without opening any popup.**

---

## 🔑 What Inline Editing Looks Like

```
BEFORE clicking Edit:
┌────┬───────────────┬────────────┬──────────┬─────────────────┐
│ #  │ Name          │ Department │ Salary   │ Actions         │
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 1  │ Alice         │ IT         │ $75,000  │ [Edit] [Delete] │
│ 2  │ Bob           │ HR         │ $55,000  │ [Edit] [Delete] │
│ 3  │ Carol         │ Finance    │ $60,000  │ [Edit] [Delete] │
└────┴───────────────┴────────────┴──────────┴─────────────────┘

AFTER clicking Edit on row 2:
┌────┬───────────────┬────────────┬──────────┬─────────────────┐
│ #  │ Name          │ Department │ Salary   │ Actions         │
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 1  │ Alice         │ IT         │ $75,000  │ [Edit] [Delete] │
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 2  │ [___Bob_____] │ [HR______▼]│ [55000__]│[Update][Cancel] │ ← row is now a form
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 3  │ Carol         │ Finance    │ $60,000  │ [Edit] [Delete] │
└────┴───────────────┴────────────┴──────────┴─────────────────┘

ADDING a new row (after clicking "+ Add New" in toolbar):
┌────┬───────────────┬────────────┬──────────┬─────────────────┐
│ +  │ [____________]│ [_________]│ [_______]│[Update][Cancel] │ ← new empty row at top
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 1  │ Alice         │ IT         │ $75,000  │ [Edit] [Delete] │
│ 2  │ Bob           │ HR         │ $55,000  │ [Edit] [Delete] │
└────┴───────────────┴────────────┴──────────┴─────────────────┘
```

---

## 🔑 Complete Setup — Tag Helper

```cshtml
@* Views/Employee/Index.cshtml *@

<kendo-grid name="employeeGrid" height="500">

    <columns>
        <column field="Id"
                title="#"
                width="60" />

        <column field="Name"
                title="Full Name"
                width="200" />

        <column field="Department"
                title="Department"
                width="150" />

        <column field="Salary"
                title="Salary"
                format="{0:C0}"
                width="130" />

        <column field="HireDate"
                title="Hire Date"
                format="{0:MM/dd/yyyy}"
                width="130" />

        {{!-- Command column — required for inline editing --}}
        <column title="Actions" width="180">
            <commands>
                <column-command name="edit"    text="Edit"   />
                <column-command name="destroy" text="Delete" />
            </commands>
        </column>
    </columns>

    {{!-- Toolbar with Add New --}}
    <toolbar>
        <toolbar-button name="create" text="+ Add New Employee" />
    </toolbar>

    {{!-- THIS is what makes it inline --}}
    <editable mode="inline" />

    {{!-- DataSource with all 4 CRUD URLs --}}
    <datasource type="DataSourceTagHelperType.Ajax" page-size="10">
        <transport>
            <read    url="@Url.Action("Read",    "Employee")" type="POST" />
            <create  url="@Url.Action("Create",  "Employee")" type="POST" />
            <update  url="@Url.Action("Update",  "Employee")" type="POST" />
            <destroy url="@Url.Action("Destroy", "Employee")" type="POST" />
        </transport>
        <schema>
            <model id="Id">
                <fields>
                    <field name="Id"         type="number"  editable="false" />
                    <field name="Name"        type="string"                   />
                    <field name="Department"  type="string"                   />
                    <field name="Salary"      type="number"                   />
                    <field name="HireDate"    type="date"                     />
                </fields>
            </model>
        </schema>
    </datasource>

    <sortable   enabled="true" />
    <filterable enabled="true" />
    <pageable   button-count="5" refresh="true" page-sizes="true" />

</kendo-grid>
```

---

## 🔑 Complete Controller

```csharp
using Kendo.Mvc.Extensions;
using Kendo.Mvc.UI;

public class EmployeeController : Controller
{
    private readonly AppDbContext _db;
    public EmployeeController(AppDbContext db) => _db = db;

    // Renders the page (no data — just the View)
    public IActionResult Index() => View();

    // ── READ ─────────────────────────────────────────────────
    [HttpPost]
    public JsonResult Read([DataSourceRequest] DataSourceRequest request)
        => Json(_db.Employees.ToDataSourceResult(request));

    // ── CREATE ───────────────────────────────────────────────
    [HttpPost]
    public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                             Employee employee)
    {
        if (ModelState.IsValid)
        {
            _db.Employees.Add(employee);
            _db.SaveChanges();
            // employee.Id is now populated by the database
        }
        return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
    }

    // ── UPDATE ───────────────────────────────────────────────
    [HttpPost]
    public JsonResult Update([DataSourceRequest] DataSourceRequest request,
                             Employee employee)
    {
        if (ModelState.IsValid)
        {
            _db.Employees.Update(employee);
            _db.SaveChanges();
        }
        return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
    }

    // ── DESTROY ──────────────────────────────────────────────
    [HttpPost]
    public JsonResult Destroy([DataSourceRequest] DataSourceRequest request,
                              Employee employee)
    {
        var existing = _db.Employees.Find(employee.Id);
        if (existing != null)
        {
            _db.Employees.Remove(existing);
            _db.SaveChanges();
        }
        return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
    }
}
```

---

## 🔑 Customising the Inline Editor — DropDownList in a Cell

By default, Kendo uses a plain text input for string fields.
Replace it with a DropDownList using the `editor` setting:

```cshtml
{{!-- Column with custom inline editor --}}
<column field="Department" title="Department" width="150">
    <column-editor>
        @(Html.Kendo().DropDownListFor(m => m.Department)
            .DataSource(ds => ds
                .Read(r => r.Action("GetDepartments", "Employee"))
            )
            .DataTextField("Name")
            .DataValueField("Name")
            .OptionLabel("-- Select --")
        )
    </column-editor>
</column>
```

Or using JavaScript in a `<script>` block:

```javascript
// Define editor function for the Department column
function departmentEditor(container, options) {
    $('<input name="' + options.field + '"/>')
        .appendTo(container)
        .kendoDropDownList({
            dataSource: ["IT", "HR", "Finance", "Sales", "Operations"],
            optionLabel: "-- Select Department --"
        });
}
```

```cshtml
{{!-- Reference the editor function --}}
<column field="Department" editor="departmentEditor" width="150" />
```

---

## 🔑 Validation in Inline Mode

When the server returns validation errors via `ModelState`, Kendo displays them inline under each field in the row:

```csharp
// Controller — ModelState errors flow back automatically
[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                         Employee employee)
{
    // Custom server-side validation
    if (_db.Employees.Any(e => e.Email == employee.Email))
        ModelState.AddModelError("Email", "This email already exists");

    if (ModelState.IsValid)
    {
        _db.Employees.Add(employee);
        _db.SaveChanges();
    }

    // Passing ModelState is what sends errors back to the grid
    return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
}
```

```csharp
// Model — Data Annotation validation
public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, ErrorMessage = "Max 100 characters")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Department is required")]
    public string Department { get; set; }

    [Range(10000, 1000000, ErrorMessage = "Salary must be between 10,000 and 1,000,000")]
    public decimal Salary { get; set; }

    public DateTime HireDate { get; set; }
}
```

---

## 🔑 Programmatic Inline Edit from JavaScript

```javascript
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");

    // Trigger edit on the first row programmatically
    grid.editRow(grid.tbody.find("tr:first"));

    // Trigger edit by finding a specific row
    function editEmployeeById(empId) {
        var row = grid.tbody.find("tr[data-uid]").filter(function() {
            return grid.dataItem(this).Id === empId;
        });
        grid.editRow(row);
    }

    // Save the currently editing row
    grid.saveRow();

    // Cancel the current edit
    grid.cancelRow();
});
```

---

## 🔑 Inline Editing Flow — What Happens Step by Step

```
User clicks [Edit] on a row
    │
    ▼
Grid: row cells become inputs (text, number, date, dropdown)
      Edit/Delete buttons → Update/Cancel buttons
    │
    ▼
User modifies values
    │
User clicks [Update]
    │
    ▼
Grid: validates fields (client-side first)
      If invalid → shows error messages under fields, stops
      If valid   → sends POST to /Employee/Update
                   Body: { id:2, name:"Bob", department:"IT", salary:60000 }
    │
    ▼
Controller: receives Employee model
            validates ModelState
            saves to database
            returns: { Data:[{id:2,...}], Total:1, Errors:null }
    │
    ▼
Grid: success → row goes back to display mode with new values
      error   → shows server validation errors under fields
    │
User clicks [Cancel]
    │
    ▼
Grid: row goes back to display mode with original values
      No request sent to server
```

---

## ⚠️ Common Inline Editing Mistakes

| Mistake                              | Symptom                                    | Fix                                                               |
| ------------------------------------ | ------------------------------------------ | ----------------------------------------------------------------- |
| Missing command column               | No Edit/Delete buttons                     | Add `<column title="Actions"><commands>...</commands></column>` |
| `editable="false"` on Id missing   | Id field appears as editable input         | Set `editable="false"` in schema model for Id                   |
| No toolbar `create` button         | Can't add new rows                         | Add `<toolbar-button name="create" />`                          |
| Not passing `ModelState` in return | Validation errors not shown in row         | Use `.ToDataSourceResult(request, ModelState)`                  |
| Only `<read>` in transport         | Edit/delete fail silently                  | All 4 transport URLs (read, create, update, destroy) required     |
| Sort/filter breaks editing           | Row snaps out of edit when clicking header | Expected behaviour — save or cancel before sorting               |

---

## ❓ Interview Questions

**Q: How do you enable inline editing in a Kendo Grid?**

> Add `<editable mode="inline" />` and include a command column with `<column-command name="edit" />` and `<column-command name="destroy" />`. The toolbar should include `<toolbar-button name="create" />` for adding new rows.

**Q: How does Kendo display server-side validation errors in inline mode?**

> The controller passes `ModelState` to `ToDataSourceResult(request, ModelState)`. Kendo reads the `Errors` property in the JSON response and shows the error messages directly under the corresponding input fields in the editing row.

**Q: How do you replace the default text input with a dropdown for a specific column in inline mode?**

> Use the `editor` attribute on the column pointing to a JavaScript function, or use `<column-editor>` with an HTML Helper inside. The function receives a container and options, and you initialise a Kendo widget inside the container.

**Q: What happens when the user clicks Cancel during inline editing?**

> The row reverts to its original display values. No request is sent to the server — the cancellation is entirely client-side.
>
