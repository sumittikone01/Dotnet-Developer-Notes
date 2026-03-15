
# 01 — Kendo DropDownList

---

## 🎯 One-Line Definition

> **The Kendo DropDownList is a styled replacement for the HTML `<select>` — it supports remote data loading via AJAX, searching, cascading, and binds directly to your model properties.**

---

## 🔑 Plain HTML Select vs Kendo DropDownList

```
Plain HTML <select>:              Kendo DropDownList:
──────────────────────────────    ──────────────────────────────────
┌──────────────────────────┐      ┌──────────────────────────────┐
│ IT                      ▼│      │ IT                          ▼│
└──────────────────────────┘      └──────────────────────────────┘
                                    ┌────────────────────────────┐
No search                           │ 🔍 Search...              │  ← searchable
No AJAX                             │────────────────────────────│
No styling                          │ ✓ IT                       │  ← selected marked
No cascade                          │   HR                       │
No templating                       │   Finance                  │
                                    │   Sales                    │
                                    └────────────────────────────┘
```

---

## 🔑 Basic Setup — Local Data (Static Options)

```cshtml
{{!-- Option 1: Inline items --}}
<kendo-dropdownlist name="Department"
                    option-label="-- Select Department --">
    <dropdownlist-items>
        <item text="IT"      value="IT"      />
        <item text="HR"      value="HR"      />
        <item text="Finance" value="Finance" />
        <item text="Sales"   value="Sales"   />
    </dropdownlist-items>
</kendo-dropdownlist>


{{!-- Option 2: Bind from ViewBag (most common in MVC) --}}
<kendo-dropdownlist name="DepartmentId"
                    bind-to="@((IEnumerable<SelectListItem>)ViewBag.Departments)"
                    option-label="-- Select Department --">
</kendo-dropdownlist>
```

```csharp
// Controller — populate ViewBag
public IActionResult Create()
{
    ViewBag.Departments = new SelectList(
        _db.Departments, "Id", "Name"
    );
    return View();
}
```

---

## 🔑 Remote Data — Load Options via AJAX

```cshtml
{{!-- Load from server when the page opens --}}
<kendo-dropdownlist name="DepartmentId"
                    data-text-field="name"
                    data-value-field="id"
                    option-label="-- Select Department --"
                    filter="FilterType.Contains">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetDepartments", "Department")" />
        </transport>
    </datasource>
</kendo-dropdownlist>
```

```csharp
// Controller returns simple JSON array
public JsonResult GetDepartments()
{
    var depts = _db.Departments
        .Select(d => new { id = d.Id, name = d.Name })
        .OrderBy(d => d.name)
        .ToList();
    return Json(depts);
}
// Returns: [ {"id":1,"name":"Finance"}, {"id":2,"name":"HR"}, ... ]
```

```
data-text-field = which JSON property to show as the display text
data-value-field = which JSON property to use as the actual value
```

---

## 🔑 Key Attributes

```cshtml
<kendo-dropdownlist name="DepartmentId"
                    data-text-field="name"
                    data-value-field="id"
                    option-label="-- Select --"
                    filter="FilterType.Contains"
                    value="@Model.DepartmentId"
                    enabled="true"
                    auto-bind="true"
                    min-length="0"
                    animation="false">
</kendo-dropdownlist>
```

| Attribute            | What It Does                    | Example                    |
| -------------------- | ------------------------------- | -------------------------- |
| `name`             | Field name — binds to model    | `name="DepartmentId"`    |
| `data-text-field`  | JSON property to display        | `data-text-field="name"` |
| `data-value-field` | JSON property as value          | `data-value-field="id"`  |
| `option-label`     | Placeholder text (empty option) | `"-- Select --"`         |
| `filter`           | Enable type-to-search           | `FilterType.Contains`    |
| `value`            | Pre-selected value              | `value="@Model.DeptId"`  |
| `enabled`          | Enable or disable               | `enabled="false"`        |
| `auto-bind`        | Load data immediately on init   | `auto-bind="true"`       |

---

## 🔑 Cascading DropDownLists

The most common real-world pattern: selecting a Country loads Cities for that Country.

```
User selects Country ──► AJAX fires ──► Cities load ──► City dropdown populated
[United States      ▼]               [ New York     ▼]
```

```cshtml
{{!-- PARENT: Country --}}
<kendo-dropdownlist name="CountryId"
                    data-text-field="name"
                    data-value-field="id"
                    option-label="Select Country...">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetCountries", "Location")" />
        </transport>
    </datasource>
</kendo-dropdownlist>


{{!-- CHILD: City — cascade-from links it to parent --}}
<kendo-dropdownlist name="CityId"
                    data-text-field="name"
                    data-value-field="id"
                    option-label="Select City..."
                    cascade-from="CountryId">
    {{!-- cascade-from="CountryId" means:
         When CountryId changes → reload this datasource
         Automatically sends CountryId value as a parameter --}}
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetCities", "Location")" />
        </transport>
    </datasource>
</kendo-dropdownlist>
```

```csharp
// Receives the parent value automatically
public JsonResult GetCities(int countryId)
// ↑ Kendo sends the selected CountryId automatically
{
    var cities = _db.Cities
        .Where(c => c.CountryId == countryId)
        .Select(c => new { id = c.Id, name = c.Name })
        .ToList();
    return Json(cities);
}
```

```
Three-level cascade (Country → State → City):
  cascade-from="CountryId"  on State DDL
  cascade-from="StateId"    on City DDL
```

---

## 🔑 JavaScript API — Controlling the DropDownList

```javascript
// Get reference
var ddl = $("#DepartmentId").data("kendoDropDownList");


// ── Reading values ─────────────────────────────────────────────
ddl.value()        // get selected value:  "IT"  or  2  (the valueField)
ddl.text()         // get displayed text:  "Information Technology"

// ── Setting values ─────────────────────────────────────────────
ddl.value("HR")    // select by value
ddl.value(3)       // select by numeric value
ddl.text("HR")     // select by text (if value matches)

// ── Reload options from server ─────────────────────────────────
ddl.dataSource.read()

// ── Completely replace the options ────────────────────────────
ddl.setDataSource(["Option A", "Option B", "Option C"]);
// OR with objects:
ddl.setDataSource([
    { id: 1, name: "IT"      },
    { id: 2, name: "Finance" }
]);

// ── Enable / Disable ──────────────────────────────────────────
ddl.enable(false)   // disable (greys out, user can't change)
ddl.enable(true)    // enable

// ── Open / Close ──────────────────────────────────────────────
ddl.open()    // programmatically open the dropdown
ddl.close()   // programmatically close it

// ── Clear selection ───────────────────────────────────────────
ddl.value("")  // select the option-label (empty state)
```

---

## 🔑 Events

```javascript
var ddl = $("#DepartmentId").data("kendoDropDownList");

// ── Fires when selection changes ──────────────────────────────
ddl.bind("change", function(e) {
    var selected = this.value();   // the value
    var text     = this.text();    // the display text

    console.log("Selected:", selected, "—", text);

    // Load related data when selection changes
    loadEmployeesForDept(selected);
    loadCitiesForCountry(selected);
});

// ── Fires when dropdown opens ─────────────────────────────────
ddl.bind("open", function() {
    console.log("Dropdown opened");
});

// ── Fires when dropdown closes ────────────────────────────────
ddl.bind("close", function() {
    console.log("Dropdown closed");
});

// ── Fires when data loads ─────────────────────────────────────
ddl.bind("dataBound", function() {
    console.log("Options loaded:", this.dataSource.data().length);
});
```

---

## 🔑 Custom Item Template — Styled Dropdown Options

```cshtml
{{!-- Template for each item in the dropdown list --}}
<kendo-dropdownlist name="EmployeeId"
                    data-text-field="name"
                    data-value-field="id"
                    template="<div style='display:flex;align-items:center;gap:8px;'>
                                  <img src='#= photoUrl #'
                                       style='width:28px;height:28px;border-radius:50%'/>
                                  <div>
                                      <div style='font-weight:bold'>#= name #</div>
                                      <div style='font-size:11px;color:#888'>#= department #</div>
                                  </div>
                               </div>"
                    value-template="<span>#= name #</span>">
    {{!-- value-template = what shows in the INPUT after selection --}}
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetEmployees", "Employee")" />
        </transport>
    </datasource>
</kendo-dropdownlist>
```

---

## 🔑 DropDownList in Kendo Grid Edit Form

Inside a Grid popup or inline editor, use DropDownList for a column:

```cshtml
{{!-- Column with DropDownList editor --}}
<column field="Department" title="Department" width="150" />

{{!-- Define the editor in JavaScript --}}
@section Scripts {
<script>
function departmentEditor(container, options) {
    $('<input name="' + options.field + '"/>')
        .appendTo(container)
        .kendoDropDownList({
            optionLabel: "-- Select --",
            dataTextField:  "name",
            dataValueField: "name",
            dataSource: {
                transport: {
                    read: { url: "@Url.Action("GetDepartments", "Department")" }
                }
            }
        });
}
</script>
}
```

---

## ⚠️ Common Mistakes

| Mistake                                             | Symptom                                  | Fix                                                                       |
| --------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------- |
| `data-text-field`/`data-value-field`names wrong | Dropdown shows `[object Object]`       | Names must exactly match JSON property names (case-sensitive)             |
| No `option-label`set                              | No empty option — user forced to pick   | Add `option-label="-- Select --"`                                       |
| `value=""`on page load but option not found       | Nothing selected, or wrong item selected | Ensure the value exists in the loaded data                                |
| `cascade-from`ID doesn't match parent `name`    | Child never loads                        | `cascade-from`must exactly match parent DDL's `name`attribute         |
| `auto-bind="false"`but no manual read             | Dropdown is empty                        | Either keep `auto-bind="true"`or call `ddl.dataSource.read()`manually |

---

## ❓ Interview Questions

**Q: What is the difference between `data-text-field` and `data-value-field`?**

> `data-text-field` is the JSON property displayed to the user in the dropdown. `data-value-field` is the JSON property used as the actual selected value (usually an ID). For example: `{ id: 1, name: "IT" }` — `data-text-field="name"` shows "IT", `data-value-field="id"` submits `1`.

**Q: How does cascading dropdown work in Kendo?**

> `cascade-from="parentName"` links a child DDL to a parent. When the parent selection changes, Kendo automatically re-reads the child's DataSource — passing the parent's current value as a parameter to the server endpoint.

**Q: How do you get the selected value and text in JavaScript?**

> `ddl.value()` returns the selected value (the `data-value-field` property). `ddl.text()` returns the displayed text (the `data-text-field` property).

**Q: How do you reload dropdown options after the page is already loaded?**

> Call `ddl.dataSource.read()` to re-fetch from the server, or `ddl.setDataSource([...])` to replace the options with a new array entirely.
>
