
# 03 — Grid Editing Modes

---

## 🎯 One-Line Definition

> **Kendo Grid has three edit modes — Inline (row becomes a form), Popup (a modal window opens), and InCell (click any cell to edit it) — each suited for a different user experience.**

---

## 🔑 The Three Modes — Visual Comparison

```
MODE 1 — INLINE
────────────────────────────────────────────────────────────────
┌────┬──────────────────────────┬────────────┬─────────────────┐
│ #  │ Name                     │ Department │ Actions         │
├────┼──────────────────────────┼────────────┼─────────────────┤
│ 1  │ Alice                    │ IT         │ [Edit] [Delete] │
├────┼──────────────────────────┼────────────┼─────────────────┤
│ 2  │ [____Bob_____________]   │ [HR_____▼] │ [Update][Cancel]│  ← editing row 2
├────┼──────────────────────────┼────────────┼─────────────────┤
│ 3  │ Carol                    │ Finance    │ [Edit] [Delete] │
└────┴──────────────────────────┴────────────┴─────────────────┘
The row itself transforms into a form.
Other rows are still visible.


MODE 2 — POPUP
────────────────────────────────────────────────────────────────
┌────┬──────────────────────────┬────────────┬─────────────────┐
│ #  │ Name                     │ Department │ Actions         │
├────┼──────────────────────────┼────────────┼─────────────────┤
│ 1  │ Alice                    │ IT         │ [Edit] [Delete] │
│ 2  │ Bob                      │ HR         │ [Edit] [Delete] │
│ 3  │ Carol                    │ Finance    │ [Edit] [Delete] │
└────┴──────────────────────────┴────────────┴─────────────────┘
         ┌─────────────────────────────────────┐
         │  Edit Employee                  [X] │
         │  Name:       [___Bob___________]    │
         │  Department: [HR______________▼]    │
         │  Salary:     [___55000_________]    │
         │  Hire Date:  [___01/15/2020_____]   │
         │                  [Update]  [Cancel] │
         └─────────────────────────────────────┘
A modal popup opens. Grid is still visible behind it.


MODE 3 — INCELL
────────────────────────────────────────────────────────────────
┌────┬──────────────────────────┬────────────┬──────────┐
│ #  │ Name                     │ Department │ Salary   │
├────┼──────────────────────────┼────────────┼──────────┤
│ 1  │ Alice                    │ IT         │ $75,000  │
├────┼──────────────────────────┼────────────┼──────────┤
│ 2  │ Bob                      │ [HR_____▼] │ [55000_] │  ← clicked these cells
├────┼──────────────────────────┼────────────┼──────────┤
│ 3  │ Carol                    │ Finance    │ $60,000  │
└────┴──────────────────────────┴────────────┴──────────┘
Click any cell → it becomes an input.
No row transform, no popup.
Tab moves to next editable cell.
```

---

## 🔑 Mode 1 — Inline Editing

User clicks Edit → entire row transforms into a form. Update/Cancel buttons appear in the row.

```cshtml
<kendo-grid name="employeeGrid">

    <columns>
        <column field="Id"         title="#"    width="60"  />
        <column field="Name"       title="Name" width="200" />
        <column field="Department"              width="150" />
        <column field="Salary"     format="{0:C0}" width="120" />

        {{!-- Command column: Edit and Delete in each row --}}
        <column title="Actions" width="180">
            <commands>
                <column-command name="edit"    text="Edit"   />
                <column-command name="destroy" text="Delete" />
            </commands>
        </column>
    </columns>

    {{!-- Set edit mode to inline --}}
    <editable mode="inline" />

    {{!-- Toolbar: Add New button --}}
    <toolbar>
        <toolbar-button name="create" text="+ Add New" />
    </toolbar>

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
                    <field name="Id"         type="number" editable="false" />
                    <field name="Name"        type="string" />
                    <field name="Department"  type="string" />
                    <field name="Salary"      type="number" />
                </fields>
            </model>
        </schema>
    </datasource>

    <sortable   enabled="true" />
    <pageable   button-count="5" refresh="true" />
    <filterable enabled="true" />

</kendo-grid>
```

**When to use Inline:**

```
✅ Simple flat data with few columns
✅ User needs to see the context (other rows) while editing
✅ Quick field-by-field edits
❌ Many fields (row becomes very wide / cramped)
❌ Complex data with nested dropdowns and date pickers
```

---

## 🔑 Mode 2 — Popup Editing

User clicks Edit → a modal popup window opens with a full form. Most common mode for forms with many fields.

```cshtml
<kendo-grid name="employeeGrid">

    <columns>
        <column field="Id"         title="#"    width="60"  />
        <column field="Name"       title="Name" width="200" />
        <column field="Department"              width="150" />
        <column field="Salary"     format="{0:C0}" width="120" />
        <column field="HireDate"   format="{0:MM/dd/yyyy}" width="130" />
        <column field="IsActive"   title="Active" width="80" />

        <column title="Actions" width="160">
            <commands>
                <column-command name="edit"    />
                <column-command name="destroy" />
            </commands>
        </column>
    </columns>

    {{!-- Set edit mode to popup --}}
    <editable mode="popup" />

    <toolbar>
        <toolbar-button name="create" text="+ Add New" />
    </toolbar>

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
                    <field name="Name"        type="string"  />
                    <field name="Department"  type="string"  />
                    <field name="Salary"      type="number"  />
                    <field name="HireDate"    type="date"    />
                    <field name="IsActive"    type="boolean" />
                </fields>
            </model>
        </schema>
    </datasource>

    <pageable   button-count="5" refresh="true" />
    <sortable   enabled="true" />
    <filterable enabled="true" />

</kendo-grid>
```

**When to use Popup:**

```
✅ Many fields to edit
✅ Complex fields (date pickers, dropdowns, file upload)
✅ Fields that need more space than a row allows
✅ Want to keep the grid visible while editing
✅ Most professional look for business applications
❌ Very simple data with 2-3 fields only (overkill)
```

---

## 🔑 Mode 3 — InCell Editing

User clicks any cell → that cell becomes an input. No buttons, no popups. Tab moves to next editable cell.

```cshtml
<kendo-grid name="employeeGrid">

    <columns>
        <column field="Id"         title="#"    width="60"  />
        <column field="Name"       title="Name" width="200" />
        <column field="Department"              width="150" />
        <column field="Salary"     format="{0:C0}" width="120" />
    </columns>

    {{!-- Set edit mode to incell --}}
    <editable mode="incell" />

    {{!-- InCell mode: Save Changes and Cancel Changes go in toolbar --}}
    <toolbar>
        <toolbar-button name="create"        text="+ Add New"      />
        <toolbar-button name="save"          text="Save All"        />
        <toolbar-button name="cancel"        text="Cancel Changes"  />
    </toolbar>

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
                    <field name="Id"         type="number" editable="false" />
                    <field name="Name"        type="string" />
                    <field name="Department"  type="string" />
                    <field name="Salary"      type="number" />
                </fields>
            </model>
        </schema>
    </datasource>

    <pageable button-count="5" refresh="true" />

</kendo-grid>
```

**When to use InCell:**

```
✅ Spreadsheet-like experience
✅ User needs to edit many cells quickly
✅ Data entry grids (budgets, schedules, quantities)
✅ Tab through cells workflow
❌ Complex validation per record
❌ Multi-step forms (popup is better)
❌ Mixed editability (some cells editable, some not — confusing)
```

---

## 🔑 Editable vs Non-Editable Columns

In all modes, control which columns can be edited:

```cshtml
<schema>
    <model id="Id">
        <fields>
            <field name="Id"         type="number"  editable="false" />  {{!-- never editable --}}
            <field name="Name"        type="string"  editable="true"  />  {{!-- editable --}}
            <field name="Department"  type="string"  editable="true"  />  {{!-- editable --}}
            <field name="CreatedAt"   type="date"    editable="false" />  {{!-- display only --}}
        </fields>
    </model>
</schema>
```

---

## 🔑 Custom Popup Editor Template

The default popup form is generated automatically from field definitions. Replace it with your own:

```cshtml
{{!-- Define a custom popup template --}}
<script type="text/x-kendo-template" id="editPopupTemplate">
    <div class="k-edit-form-container">
        <div class="form-row">
            <label>Full Name</label>
            <input name="Name" data-bind="value:Name" class="k-textbox" required />
            <span data-for="Name" class="k-invalid-msg"></span>
        </div>
        <div class="form-row">
            <label>Department</label>
            <input name="Department" data-bind="value:Department"
                   data-role="dropdownlist"
                   data-source='["IT","HR","Finance","Sales"]' />
        </div>
        <div class="form-row">
            <label>Salary</label>
            <input name="Salary" data-bind="value:Salary"
                   data-role="numerictextbox"
                   data-format="c0"
                   data-min="0" />
            <span data-for="Salary" class="k-invalid-msg"></span>
        </div>
        <div class="form-row">
            <label>Active</label>
            <input name="IsActive" data-bind="checked:IsActive"
                   type="checkbox" />
        </div>
    </div>
</script>

{{!-- Reference the template in the grid --}}
<editable mode="popup"
          template-id="editPopupTemplate" />
```

---

## 🔑 The Controller — Same for All Three Modes

All three editing modes use the same four controller actions. The mode only changes the front-end UX:

```csharp
[HttpPost]
public JsonResult Read([DataSourceRequest] DataSourceRequest request)
    => Json(_db.Employees.ToDataSourceResult(request));

[HttpPost]
public JsonResult Create([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    if (ModelState.IsValid) { _db.Employees.Add(emp); _db.SaveChanges(); }
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
}

[HttpPost]
public JsonResult Update([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    if (ModelState.IsValid) { _db.Employees.Update(emp); _db.SaveChanges(); }
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
}

[HttpPost]
public JsonResult Destroy([DataSourceRequest] DataSourceRequest request, Employee emp)
{
    var item = _db.Employees.Find(emp.Id);
    if (item != null) { _db.Employees.Remove(item); _db.SaveChanges(); }
    return Json(new[] { emp }.ToDataSourceResult(request, ModelState));
}
```

---

## 📊 Three Modes — Decision Table

|                                 | Inline                  | Popup                      | InCell                 |
| ------------------------------- | ----------------------- | -------------------------- | ---------------------- |
| **Edit triggers**         | Click Edit button       | Click Edit button          | Click any cell         |
| **Form location**         | Inside the row          | Modal window               | Inside the cell        |
| **Save/Cancel**           | Per-row buttons         | Popup buttons              | Toolbar buttons        |
| **Other rows visible**    | ✅ Yes                  | ✅ Yes                     | ✅ Yes                 |
| **Best for**              | Few fields, quick edits | Many fields, complex forms | Spreadsheet data entry |
| **Validation display**    | Inline under fields     | Inside popup               | Tooltip on cell        |
| **Multiple rows at once** | ❌ One at a time        | ❌ One at a time           | ✅ Yes                 |
| `<editable mode=`             | `"inline"`            | `"popup"`                | `"incell"`           |

---

## ⚠️ Common Mistakes

| Mistake                                      | Symptom                            | Fix                                                 |
| -------------------------------------------- | ---------------------------------- | --------------------------------------------------- |
| No command column for inline/popup           | No Edit/Delete buttons appear      | Add `<column title="Actions"><commands>...`       |
| InCell with no Save toolbar button           | Changes made but no way to save    | Add `<toolbar-button name="save" />`              |
| `editable="false"`on Id but not in schema  | Id gets sent as editable field     | Set `editable="false"`in schema model field       |
| Missing Create/Update/Destroy transport URLs | Editing opens but save gives error | All 4 URLs (read, create, update, destroy) required |

---

## ❓ Interview Questions

**Q: What are the three Kendo Grid edit modes?**

> Inline — the clicked row transforms into a form with Update/Cancel buttons. Popup — a modal window opens with the edit form. InCell — clicking any cell turns it into an input directly. Mode is set with `<editable mode="inline/popup/incell" />`.

**Q: Which edit mode is best for forms with many fields?**

> Popup. A popup window gives unlimited vertical space for many fields, date pickers, and dropdowns — a row would be too cramped and an incell approach would require many clicks.

**Q: How does InCell mode save changes?**

> InCell doesn't have per-row save buttons. Instead, "Save Changes" and "Cancel Changes" buttons go in the grid toolbar and apply to all pending changes at once. This makes it suitable for spreadsheet-style bulk editing.

**Q: Does the controller need to change based on which edit mode is used?**

> No. All three modes use the exact same four controller actions (Read, Create, Update, Destroy). The editing mode only changes the front-end user experience — the server-side code is identical.
>
