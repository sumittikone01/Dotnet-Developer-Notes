# 05 — Grid Popup Editing

---

## 🎯 One-Line Definition

> **Popup editing opens a modal window with a full form when the user clicks Edit — the grid stays visible behind it, there's unlimited space for fields, and the form closes after saving or cancelling.**

---

## 🔑 What Popup Editing Looks Like

```
Grid stays visible, popup appears on top:

┌────┬───────────────┬────────────┬──────────┬─────────────────┐
│ #  │ Name          │ Department │ Salary   │ Actions         │
├────┼───────────────┼────────────┼──────────┼─────────────────┤
│ 1  │ Alice         │ IT         │ $75,000  │ [Edit] [Delete] │
│ 2  │ Bob           │ HR         │ $55,000  │ [Edit] [Delete] │  ← clicked Edit
│ 3  │ Carol         │ Finance    │ $60,000  │ [Edit] [Delete] │
└────┴───────────────┴────────────┴──────────┴─────────────────┘

         ┌──────────────────────────────────────────┐
         │  Edit Employee                       [×] │
         ├──────────────────────────────────────────┤
         │                                          │
         │  Full Name *                             │
         │  [___Bob_______________________________] │
         │                                          │
         │  Department *                            │
         │  [HR____________________________________▼]│
         │                                          │
         │  Salary                                  │
         │  [___$55,000___________________________] │
         │                                          │
         │  Hire Date                               │
         │  [___01/15/2020_____________________📅]  │
         │                                          │
         │  ☑ Active                                │
         │                                          │
         │              [Update]  [Cancel]          │
         └──────────────────────────────────────────┘
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
                width="220" />
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
        <column field="IsActive"
                title="Active"
                width="80" />
        <column field="Email"
                title="Email"
                width="220" />

        {{!-- Command column --}}
        <column title="Actions" width="160">
            <commands>
                <column-command name="edit"    text="Edit"   />
                <column-command name="destroy" text="Delete" />
            </commands>
        </column>
    </columns>

    {{!-- Toolbar --}}
    <toolbar>
        <toolbar-button name="create" text="+ Add New Employee" />
        <toolbar-button name="excel"  text="Export to Excel"    />
        <toolbar-button name="search"                           />
    </toolbar>

    {{!-- THIS is what makes it popup --}}
    <editable mode="popup" />

    {{!-- DataSource --}}
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
                    <field name="IsActive"    type="boolean"                  />
                    <field name="Email"       type="string"                   />
                </fields>
            </model>
        </schema>
    </datasource>

    <sortable   enabled="true" />
    <filterable enabled="true" />
    <pageable   button-count="5" refresh="true" page-sizes="true" />
    <excel      file-name="employees.xlsx" all-pages="true" />

</kendo-grid>
```

---

## 🔑 Complete Controller

```csharp
public class EmployeeController : Controller
{
    private readonly AppDbContext _db;
    public EmployeeController(AppDbContext db) => _db = db;

    public IActionResult Index() => View();

    [HttpPost]
    public JsonResult Read([DataSourceRequest] DataSourceRequest request)
        => Json(_db.Employees.ToDataSourceResult(request));

    [HttpPost]
    public JsonResult Create([DataSourceRequest] DataSourceRequest request,
                             Employee employee)
    {
        if (ModelState.IsValid)
        {
            // Business validation
            if (_db.Employees.Any(e => e.Email == employee.Email))
                ModelState.AddModelError("Email", "Email already in use");
            else
            {
                employee.HireDate = DateTime.Today;
                _db.Employees.Add(employee);
                _db.SaveChanges();
            }
        }
        return Json(new[] { employee }.ToDataSourceResult(request, ModelState));
    }

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

## 🔑 Custom Popup Template — Full Control Over the Form

The default popup auto-generates a form from field definitions.
Replace it with your own HTML for full control:

```cshtml
{{!-- Define popup form template --}}
<script type="text/x-kendo-template" id="employeePopupTemplate">
    <div style="padding: 20px; min-width: 400px;">

        <div class="form-group mb-3">
            <label class="form-label fw-bold">Full Name *</label>
            <input name="Name"
                   data-bind="value: Name"
                   class="k-textbox w-100"
                   required
                   data-required-msg="Name is required" />
            <span data-for="Name" class="k-invalid-msg text-danger"></span>
        </div>

        <div class="form-group mb-3">
            <label class="form-label fw-bold">Department *</label>
            <select name="Department"
                    data-bind="value: Department"
                    data-role="dropdownlist"
                    data-option-label="-- Select Department --"
                    style="width:100%"
                    required>
                <option>IT</option>
                <option>HR</option>
                <option>Finance</option>
                <option>Sales</option>
                <option>Operations</option>
            </select>
            <span data-for="Department" class="k-invalid-msg text-danger"></span>
        </div>

        <div class="form-group mb-3">
            <label class="form-label fw-bold">Salary</label>
            <input name="Salary"
                   data-bind="value: Salary"
                   data-role="numerictextbox"
                   data-format="c0"
                   data-min="0"
                   data-max="1000000"
                   style="width:100%" />
            <span data-for="Salary" class="k-invalid-msg text-danger"></span>
        </div>

        <div class="form-group mb-3">
            <label class="form-label fw-bold">Hire Date</label>
            <input name="HireDate"
                   data-bind="value: HireDate"
                   data-role="datepicker"
                   data-format="MM/dd/yyyy"
                   style="width:100%" />
        </div>

        <div class="form-group mb-3">
            <label class="form-label fw-bold">Email *</label>
            <input name="Email"
                   data-bind="value: Email"
                   class="k-textbox w-100"
                   type="email"
                   required
                   data-email-msg="Enter a valid email" />
            <span data-for="Email" class="k-invalid-msg text-danger"></span>
        </div>

        <div class="form-group mb-3">
            <label class="form-label">
                <input name="IsActive"
                       data-bind="checked: IsActive"
                       type="checkbox" />
                  Active Employee
            </label>
        </div>

    </div>
</script>

{{!-- Reference the template and give the popup a title --}}
<editable mode="popup"
          template-id="employeePopupTemplate"
          window-title="Employee Details" />
```

---

## 🔑 Configuring the Popup Window Size

```javascript
// Set popup window size via Grid events
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");

    grid.bind("edit", function(e) {
        // e.container = the popup window element
        // e.model     = the data item being edited

        var popup = e.container.data("kendoWindow");

        // Set size
        popup.setOptions({
            width:  600,
            height: 500
        });

        // Different title for Add vs Edit
        if (e.model.isNew()) {
            popup.title("Add New Employee");
        } else {
            popup.title("Edit Employee — " + e.model.Name);
        }

        // Centre after resizing
        popup.center();
    });
});
```

---

## 🔑 Pre-Populating Fields When Opening Popup

Set default values for new records when the Add New popup opens:

```javascript
$(function() {
    var grid = $("#employeeGrid").data("kendoGrid");

    grid.bind("edit", function(e) {
        if (e.model.isNew()) {
            // Set defaults for new records only
            e.model.set("HireDate", new Date());        // today
            e.model.set("IsActive", true);              // active by default
            e.model.set("Department", "IT");            // default department
        }
    });
});
```

---

## 🔑 Accessing Popup Form Fields After Open

```javascript
grid.bind("edit", function(e) {
    // e.container = the popup <div>
    // Query inside the popup to find specific inputs

    var nameInput    = e.container.find("[name='Name']");
    var salaryInput  = e.container.find("[name='Salary']");
    var deptDdl      = e.container.find("[name='Department']").data("kendoDropDownList");

    // Focus the Name field automatically
    nameInput.focus();

    // Disable salary field if editing an existing record
    if (!e.model.isNew()) {
        salaryInput.data("kendoNumericTextBox").enable(false);
    }
});
```

---

## 🔑 Popup Save Confirmation

Intercept the save to show a confirm dialog:

```javascript
grid.bind("save", function(e) {
    // e.model     = the data being saved
    // e.container = the popup container

    // You can validate here and cancel if needed
    if (e.model.Salary > 500000) {
        e.preventDefault();   // stop the save

        kendo.confirm("Salary is unusually high ($" +
            kendo.format("{0:C0}", e.model.Salary) +
            "). Are you sure?")
            .then(function() {
                // User confirmed — save manually
                e.sender.dataSource.sync();
            });
    }
});
```

---

## 🔑 Popup vs Inline — When to Choose Which

```
Choose POPUP when:                    Choose INLINE when:
────────────────────────────────      ────────────────────────────────
Many fields to edit (5+)             Few fields (2–4)
Need DatePicker, file upload         Simple text/number inputs only
Complex validation messages          Quick edits with little validation
Fields need full width               User needs to see all rows
Professional enterprise form look    Compact quick-edit experience
Add and edit look the same           Save is per-row immediately
```

---

## ⚠️ Common Popup Editing Mistakes

| Mistake                                       | Symptom                              | Fix                                                                     |
| --------------------------------------------- | ------------------------------------ | ----------------------------------------------------------------------- |
| Missing `window-title`                      | Popup title shows "undefined"        | Add `window-title="Your Title"`to `<editable>`                      |
| Template `id`typo                           | Default form shown instead of custom | Match `template-id`exactly to `<script id="...">`                   |
| `data-bind`names wrong                      | Fields empty or not saved            | `data-bind="value: FieldName"`must match schema field name exactly    |
| Popup too small for content                   | Form overflows the window            | Set `width`and `height`in the `edit`event handler                 |
| Kendo widget in template not initialised      | Raw input instead of DatePicker/DDL  | Use `data-role="datepicker"`or `data-role="dropdownlist"`attributes |
| `e.model.set()`called with wrong field name | Field doesn't update                 | Field name is case-sensitive — matches schema exactly                  |

---

## ❓ Interview Questions

**Q: How do you use a custom form in the popup instead of the auto-generated one?**

> Create a `<script type="text/x-kendo-template">` block with your form HTML, then reference it with `template-id="yourTemplateId"` on the `<editable>` tag. Inside the template, use `data-bind="value: FieldName"` to bind each input to the model.

**Q: How do you change the popup window title and size?**

> Handle the grid's `edit` event. Inside it, `e.container.data("kendoWindow")` gives you the popup window widget. Call `.title("New Title")` and `.setOptions({ width: 600, height: 500 })` then `.center()`.

**Q: How do you tell if the popup is opening for a new record vs an existing one?**

> Check `e.model.isNew()` inside the `edit` event handler. Returns `true` for new records (Add New clicked), `false` for existing records (Edit clicked).

**Q: How do validation errors appear in popup mode?**

> The server returns `Errors` in the JSON response via `ToDataSourceResult(request, ModelState)`. Kendo reads this and shows error messages next to the corresponding fields inside the popup. Each field needs a `<span data-for="FieldName" class="k-invalid-msg">` for the message to appear.
>
