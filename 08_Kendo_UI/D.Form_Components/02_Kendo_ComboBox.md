
# 02 — Kendo ComboBox

---

## 🎯 One-Line Definition

> **The Kendo ComboBox is a DropDownList where the user can also type a custom value not in the list — they get suggestions as they type, but they're not forced to pick from them.**

---

## 🔑 DropDownList vs ComboBox — One Key Difference

```
DropDownList:                        ComboBox:
────────────────────────────────     ────────────────────────────────
User MUST pick from the list         User CAN pick from list OR type
                                     their own custom value

[IT                          ▼]      [Typing: "Mark              ▼]
                                      ┌───────────────────────────┐
If user types "XYZ" and               │ Marketing                 │  ← suggestion
the list has no match →               │ Management                │  ← suggestion
selection is cleared                  └───────────────────────────┘
                                      User can also submit "Marketing Dept"
                                      even if it's not in the list
```

---

## 🔑 Basic Setup — Local Data

```cshtml
<kendo-combobox name="Department"
                option-label="-- Select or type --"
                filter="FilterType.Contains">
    <combobox-items>
        <item text="IT"      value="IT"      />
        <item text="HR"      value="HR"      />
        <item text="Finance" value="Finance" />
        <item text="Sales"   value="Sales"   />
    </combobox-items>
</kendo-combobox>
```

---

## 🔑 Remote Data — Load from Server

```cshtml
<kendo-combobox name="EmployeeName"
                data-text-field="name"
                data-value-field="id"
                option-label="Search or type a name..."
                filter="FilterType.Contains"
                min-length="2"
                delay="300">
    {{!-- min-length: don't search until 2 chars typed    --}}
    {{!-- delay: wait 300ms after keystroke before AJAX   --}}
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("SearchEmployees", "Employee")" />
        </transport>
    </datasource>
</kendo-combobox>
```

```csharp
// Kendo sends the typed text as "filter[filters][0][value]"
// but with suggest mode it sends it differently —
// simplest approach: receive the search text explicitly

public JsonResult SearchEmployees(string text)
// Kendo sends "text" = whatever user typed
{
    var results = _db.Employees
        .Where(e => e.Name.Contains(text ?? ""))
        .Select(e => new { id = e.Id, name = e.Name })
        .Take(10)
        .ToList();
    return Json(results);
}
```

---

## 🔑 Key Attributes

| Attribute            | What It Does                 | Example                    |
| -------------------- | ---------------------------- | -------------------------- |
| `name`             | Field name for model binding | `name="Department"`      |
| `data-text-field`  | JSON property to display     | `data-text-field="name"` |
| `data-value-field` | JSON property as value       | `data-value-field="id"`  |
| `option-label`     | Placeholder / empty hint     | `"-- Select or type --"` |
| `filter`           | How suggestions filter       | `FilterType.Contains`    |
| `min-length`       | Min chars before AJAX fires  | `min-length="2"`         |
| `delay`            | Ms to wait after keystroke   | `delay="300"`            |
| `value`            | Pre-selected value           | `value="@Model.Dept"`    |
| `suggest`          | Auto-complete in input       | `suggest="true"`         |
| `auto-bind`        | Load options on page load    | `auto-bind="false"`      |

---

## 🔑 Filter Modes

```cshtml
{{!-- Contains: "an" matches "Finance", "Management" --}}
<kendo-combobox filter="FilterType.Contains" />

{{!-- StartsWith: "fin" matches "Finance" but not "Defining" --}}
<kendo-combobox filter="FilterType.StartsWith" />

{{!-- None: no filtering — show all options regardless of what's typed --}}
<kendo-combobox filter="FilterType.None" />
```

---

## 🔑 `suggest` — Auto-Complete the Input

```cshtml
{{!-- suggest="true": as user types "IT", the input auto-completes --}}
<kendo-combobox name="Department"
                suggest="true"
                filter="FilterType.StartsWith">
    <combobox-items>
        <item text="IT"      value="IT"      />
        <item text="HR"      value="HR"      />
        <item text="Finance" value="Finance" />
    </combobox-items>
</kendo-combobox>

{{!-- User types "F" → input auto-completes to "Finance"
     User types "H" → input auto-completes to "HR" --}}
```

---

## 🔑 JavaScript API

```javascript
var combo = $("#EmployeeName").data("kendoComboBox");

// ── Reading values ─────────────────────────────────────────────
combo.value()   // the value (data-value-field) — or the typed text if custom
combo.text()    // the displayed text in the input

// ── Setting values ─────────────────────────────────────────────
combo.value(5)           // select item with value 5
combo.value("IT")        // select by value string
combo.value("")          // clear to empty state
combo.text("New Custom") // set free-text (custom value not in list)

// ── Working with data ─────────────────────────────────────────
combo.dataItem()          // get the selected dataItem object (null if custom)
combo.dataSource.read()   // reload options from server

// ── Enable / Disable ──────────────────────────────────────────
combo.enable(false)
combo.enable(true)

// ── Open / Close ──────────────────────────────────────────────
combo.open()
combo.close()
```

---

## 🔑 Events

```javascript
var combo = $("#Department").data("kendoComboBox");

// ── Change: fires when value changes (pick or clear) ─────────
combo.bind("change", function() {
    var val      = this.value();
    var text     = this.text();
    var dataItem = this.dataItem();  // null if user typed custom value

    if (dataItem) {
        console.log("Picked from list:", dataItem.id, dataItem.name);
    } else {
        console.log("Custom value typed:", text);
    }
});

// ── Filtering: fires when user types ──────────────────────────
combo.bind("filtering", function(e) {
    // e.filter.value = what the user typed
    console.log("User typed:", e.filter.value);
});

// ── Open/Close ────────────────────────────────────────────────
combo.bind("open",  function() { console.log("opened"); });
combo.bind("close", function() { console.log("closed"); });

// ── DataBound: fires when options loaded ──────────────────────
combo.bind("dataBound", function() {
    console.log("Options loaded:", this.dataSource.total());
});
```

---

## 🔑 Checking Whether User Picked from List or Typed Custom

```javascript
var combo = $("#Department").data("kendoComboBox");

combo.bind("change", function() {
    var dataItem = this.dataItem();  // null = user typed something custom

    if (dataItem) {
        // User picked an existing item
        var id   = dataItem.id;
        var name = dataItem.name;
        console.log("Existing item selected — ID:", id);
    } else {
        // User typed a custom value not in the list
        var customText = this.text();
        console.log("Custom value entered:", customText);

        // Maybe offer to add it to the DB
        offerToCreateNewDepartment(customText);
    }
});
```

---

## 🔑 ComboBox for Tags / Multiple Selection

> For selecting multiple items, use `<kendo-multiselect>` instead.
> ComboBox is single-selection only.

```cshtml
{{!-- MultiSelect for multiple choices --}}
<kendo-multiselect name="Skills"
                   data-text-field="name"
                   data-value-field="id"
                   placeholder="Select skills...">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetSkills", "Employee")" />
        </transport>
    </datasource>
</kendo-multiselect>
```

---

## 📊 DropDownList vs ComboBox vs AutoComplete

|                          | DropDownList        | ComboBox                 | AutoComplete           |
| ------------------------ | ------------------- | ------------------------ | ---------------------- |
| User must pick from list | ✅ Yes              | ❌ No — can type custom | ❌ No — just suggests |
| Stores ID or text        | Either (valueField) | Either (valueField)      | Stores text only       |
| Searchable               | Optional            | ✅ Always                | ✅ Always              |
| Suggestions while typing | ❌                  | ✅                       | ✅                     |
| Custom free-text allowed | ❌                  | ✅                       | ✅                     |
| Best for                 | Strict picklist     | Flexible picklist        | Search / lookup        |

---

## ⚠️ Common Mistakes

| Mistake                                     | Symptom                                                 | Fix                                                                      |
| ------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------------ |
| Using ComboBox where DropDownList is needed | User types garbage values that fail server validation   | Use DropDownList when values MUST be from the list                       |
| `min-length`too low with remote data      | AJAX fires on every single keystroke, too many requests | Set `min-length="2"`or higher +`delay="300"`                         |
| Not handling `dataItem() === null`case    | Server receives unexpected free text                    | Check `dataItem()`in change handler — handle custom values explicitly |
| Forgetting `delay`                        | AJAX fires while user is still typing, slow/flashing    | Add `delay="300"`to debounce                                           |

---

## ❓ Interview Questions

**Q: What is the difference between ComboBox and DropDownList?**

> DropDownList forces the user to select from the list — typing filters but the value must match an item. ComboBox allows free-text — the user can type anything, and the server may receive a value not in the list.

**Q: How do you detect whether the user selected from the list or typed a custom value?**

> Call `combo.dataItem()` in the change handler. If it returns a data object, the user picked from the list. If it returns `null`, the user typed a custom value — read it with `combo.text()`.

**Q: What do `min-length` and `delay` do?**

> `min-length` prevents AJAX requests until the user has typed at least N characters. `delay` adds a millisecond pause after the last keystroke before firing the request — prevents a request on every single keystroke.

**Q: When should you use ComboBox vs MultiSelect?**

> ComboBox is for single-value selection with optional free-text entry. MultiSelect is for selecting multiple items from a list. If the user needs to choose more than one option, use MultiSelect.
>
