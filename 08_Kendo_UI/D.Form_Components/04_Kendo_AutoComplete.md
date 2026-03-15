
# 04 — Kendo AutoComplete

---

## 🎯 One-Line Definition

> **AutoComplete shows a suggestion dropdown as the user types — it fetches matching results from the server in real time and lets the user pick one, but always stores the text, never an ID.**

---

## 🔑 AutoComplete vs ComboBox vs DropDownList

```
┌────────────────────────────────────────────────────────────────┐
│  DropDownList  → Must pick from list. Stores ID or text.       │
│  ComboBox      → Pick from list OR type custom. Stores ID/text.│
│  AutoComplete  → Suggestions while typing. Stores TEXT ONLY.   │
└────────────────────────────────────────────────────────────────┘

AutoComplete in action:
User types "al" →

[al                              🔍]
┌───────────────────────────────────┐
│ Alice Johnson                     │  ← suggestion
│ Albert Smith                      │  ← suggestion
│ Alexandra Lee                     │  ← suggestion
└───────────────────────────────────┘

User clicks "Alice Johnson" → input shows "Alice Johnson"
Stored value = "Alice Johnson"  (just the text — no hidden ID)
```

---

## 🔑 When to Use AutoComplete

```
USE AutoComplete when:
  ✅ User searches by name/keyword — you store the text
  ✅ Search box that gives hints while typing
  ✅ Tags, cities, product names — text values
  ✅ The result IS the value (city name, country name)

DON'T USE AutoComplete when:
  ❌ You need to store an ID behind the display text
     (Use ComboBox or DropDownList instead)
  ❌ User must pick — they can't type free text
     (Use DropDownList instead)
```

---

## 🔑 Basic Setup — Local Data

```cshtml
{{!-- Simple list of strings --}}
<kendo-autocomplete name="City"
                    placeholder="Type a city..."
                    filter="FilterType.Contains">
    <autocomplete-items>
        <item text="New York" />
        <item text="Los Angeles" />
        <item text="Chicago" />
        <item text="Houston" />
        <item text="Phoenix" />
    </autocomplete-items>
</kendo-autocomplete>
```

---

## 🔑 Remote Data — Load Suggestions from Server

This is the most common and most useful use case:

```cshtml
<kendo-autocomplete name="EmployeeSearch"
                    data-text-field="name"
                    placeholder="Type employee name..."
                    filter="FilterType.Contains"
                    min-length="2"
                    delay="300">
    {{!-- min-length: don't search until 2 chars typed   --}}
    {{!-- delay: wait 300ms after keystroke before AJAX  --}}
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("SearchEmployees", "Employee")" />
        </transport>
    </datasource>
</kendo-autocomplete>
```

```csharp
// Kendo sends the typed text in a filter parameter
// But the simplest approach is to receive it as "text"
public JsonResult SearchEmployees(string text)
{
    if (string.IsNullOrEmpty(text)) return Json(new List<object>());

    var results = _db.Employees
        .Where(e => e.Name.Contains(text))
        .Select(e => new { name = e.Name, id = e.Id, dept = e.Department })
        .Take(10)                // limit suggestions shown
        .OrderBy(e => e.name)
        .ToList();

    return Json(results);
}
// Returns: [ {"name":"Alice","id":1,"dept":"IT"}, ... ]
```

---

## 🔑 Key Attributes

```cshtml
<kendo-autocomplete name="EmployeeSearch"
                    data-text-field="name"
                    value="@Model.EmployeeName"
                    placeholder="Start typing..."
                    filter="FilterType.Contains"
                    min-length="2"
                    delay="300"
                    suggest="true"
                    highlight-first="true"
                    separator=", "
                    enabled="true" />
```

| Attribute           | What It Does                                 | Typical Value             |
| ------------------- | -------------------------------------------- | ------------------------- |
| `name`            | Field name — binds to model                 | `name="EmployeeSearch"` |
| `data-text-field` | Which JSON property to display as suggestion | `"name"`                |
| `value`           | Pre-filled text value                        | `"@Model.EmployeeName"` |
| `placeholder`     | Hint when empty                              | `"Type to search..."`   |
| `filter`          | How text matching works                      | `FilterType.Contains`   |
| `min-length`      | Min chars before AJAX fires                  | `2`                     |
| `delay`           | Ms to wait after keystroke                   | `300`                   |
| `suggest`         | Auto-complete typed text                     | `true`/`false`        |
| `highlight-first` | Auto-highlight first suggestion              | `true`/`false`        |
| `separator`       | Allow multiple values separated              | `", "`                  |

---

## 🔑 Filter Modes

```cshtml
{{!-- Contains — "al" matches "Alice", "Vitaly", "Albert" --}}
<kendo-autocomplete filter="FilterType.Contains" />

{{!-- StartsWith — "al" only matches "Alice", "Albert" --}}
<kendo-autocomplete filter="FilterType.StartsWith" />

{{!-- None — show ALL items regardless of what user types --}}
<kendo-autocomplete filter="FilterType.None" />
```

---

## 🔑 `suggest` — Auto-Complete the Typed Text

```cshtml
<kendo-autocomplete name="City"
                    suggest="true"
                    filter="FilterType.StartsWith">
    <autocomplete-items>
        <item text="New York" />
        <item text="New Jersey" />
        <item text="Newark" />
    </autocomplete-items>
</kendo-autocomplete>

{{!--
User types "New" → input auto-completes to "New York" (first match)
User keeps typing "New J" → auto-completes to "New Jersey"
The suggested completion is highlighted/selected text

suggest works best with StartsWith filter
--}}
```

---

## 🔑 `separator` — Multiple Values in One Field

Let users type and pick multiple values separated by a character:

```cshtml
{{!-- Tags input: user picks multiple skills --}}
<kendo-autocomplete name="Skills"
                    data-text-field="name"
                    placeholder="Add skills (type and pick)..."
                    filter="FilterType.Contains"
                    separator=", ">
    {{!-- separator=", " means each picked item is appended --}}
    {{!-- Result in input: "C#, SQL, JavaScript"           --}}
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetSkills", "Employee")" />
        </transport>
    </datasource>
</kendo-autocomplete>

{{!-- Stored value: "C#, SQL, JavaScript" (plain string) --}}
```

---

## 🔑 JavaScript API

```javascript
var ac = $("#EmployeeSearch").data("kendoAutoComplete");

// ── Reading the value ──────────────────────────────────────────
ac.value()      // the current text in the input
                // e.g.: "Alice Johnson"
                // Note: ALWAYS text — AutoComplete has no hidden ID

// ── Setting the value ──────────────────────────────────────────
ac.value("Alice Johnson")   // fill the input with text
ac.value("")                // clear the input

// ── Programmatic search ────────────────────────────────────────
ac.search("Ali")            // trigger a search for "Ali"

// ── Close the suggestion dropdown ─────────────────────────────
ac.close()

// ── Reload suggestions ─────────────────────────────────────────
ac.dataSource.read()

// ── Enable / Disable ──────────────────────────────────────────
ac.enable(false)   // greys out, user can't type
ac.enable(true)

// ── Read-only ─────────────────────────────────────────────────
ac.readonly(true)
ac.readonly(false)
```

---

## 🔑 Events

```javascript
var ac = $("#EmployeeSearch").data("kendoAutoComplete");

// ── Change: fires when user picks or clears ───────────────────
ac.bind("change", function() {
    var typedText = this.value();
    console.log("Value now:", typedText);
    filterGridByName(typedText);
});

// ── Select: fires when user PICKS an item from the list ───────
// (more precise than change — only fires on actual selection)
ac.bind("select", function(e) {
    // e.item = the <li> element selected
    // e.dataItem = the data object from the datasource
    var dataItem = e.dataItem;

    if (dataItem) {
        console.log("Picked:", dataItem.name, "ID:", dataItem.id);
        // You CAN access the ID even though it's not stored as value
        loadEmployeeById(dataItem.id);
    }
});

// ── Filtering: fires when user types ──────────────────────────
ac.bind("filtering", function(e) {
    // e.filter.value = text user typed
    console.log("Searching for:", e.filter.value);
});

// ── DataBound: fires when suggestions load ────────────────────
ac.bind("dataBound", function() {
    var count = this.dataSource.data().length;
    if (count === 0) {
        console.log("No matches found");
    }
});
```

---

## 🔑 Accessing the Hidden ID — The Real Pattern

AutoComplete stores only text — but you can grab the ID from the `select` event
and store it yourself in a hidden field:

```cshtml
{{!-- Hidden field to store the selected ID --}}
<input type="hidden" id="SelectedEmployeeId" name="EmployeeId" />

{{!-- AutoComplete for display --}}
<kendo-autocomplete name="EmployeeSearch"
                    data-text-field="name"
                    placeholder="Search employee..."
                    filter="FilterType.Contains"
                    min-length="2"
                    delay="300">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("SearchEmployees", "Employee")" />
        </transport>
    </datasource>
</kendo-autocomplete>
```

```javascript
var ac = $("#EmployeeSearch").data("kendoAutoComplete");

// When user picks from the list → store their ID
ac.bind("select", function(e) {
    if (e.dataItem) {
        $("#SelectedEmployeeId").val(e.dataItem.id);
        console.log("Employee ID stored:", e.dataItem.id);
    }
});

// When user clears/changes text → clear the stored ID
ac.bind("change", function() {
    if (!this.value()) {
        $("#SelectedEmployeeId").val("");
    }
});
```

---

## 🔑 Custom Item Template — Styled Suggestions

```cshtml
{{!-- Rich suggestion items with photo + details --}}
<kendo-autocomplete name="EmployeeSearch"
                    data-text-field="name"
                    min-length="2"
                    delay="300"
                    template="
                        <div class='suggestion-item'>
                            <img src='#= photoUrl || \"/images/avatar.png\" #'
                                 style='width:32px;height:32px;border-radius:50%;
                                        object-fit:cover;vertical-align:middle;
                                        margin-right:8px;'/>
                            <strong>#: name #</strong>
                            <span style='color:#888;font-size:12px;margin-left:6px;'>
                                #= department #
                            </span>
                        </div>">
    <datasource type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("SearchEmployees", "Employee")" />
        </transport>
    </datasource>
</kendo-autocomplete>
```

---

## 🔑 AutoComplete as a Live Search / Filter for a Grid

The most common real-world use: search box above a grid that filters it as you type.

```cshtml
{{!-- Search box above the grid --}}
<div class="mb-3">
    <label>Search Employees:</label>
    <kendo-autocomplete name="GridSearch"
                        data-text-field="name"
                        placeholder="Type name to filter grid..."
                        filter="FilterType.Contains"
                        min-length="1"
                        delay="400"
                        style="width:300px">
        <datasource type="DataSourceTagHelperType.Ajax">
            <transport>
                <read url="@Url.Action("SearchEmployees", "Employee")" />
            </transport>
        </datasource>
    </kendo-autocomplete>
</div>

<kendo-grid name="employeeGrid">
    ...
</kendo-grid>
```

```javascript
$(function() {
    var ac   = $("#GridSearch").data("kendoAutoComplete");
    var grid = $("#employeeGrid").data("kendoGrid");

    // Filter grid when user picks a suggestion
    ac.bind("select", function(e) {
        if (e.dataItem) {
            grid.dataSource.filter({
                field:    "Name",
                operator: "contains",
                value:    e.dataItem.name
            });
        }
    });

    // Clear grid filter when search is cleared
    ac.bind("change", function() {
        if (!this.value()) {
            grid.dataSource.filter({});
        }
    });
});
```

---

## 📊 AutoComplete vs ComboBox vs DropDownList — Final Table

| Feature                  | AutoComplete                | ComboBox           | DropDownList       |
| ------------------------ | --------------------------- | ------------------ | ------------------ |
| Stores                   | Text only                   | Value (ID or text) | Value (ID or text) |
| Forces list pick         | ❌ No                       | ❌ No              | ✅ Yes             |
| Suggestions while typing | ✅ Always                   | ✅ Yes             | ❌ Optional filter |
| Hidden ID support        | ❌ (use hidden field trick) | ✅ Built-in        | ✅ Built-in        |
| Best for                 | Search, free text           | Flexible picklist  | Strict picklist    |

---

## ⚠️ Common Mistakes

| Mistake                                         | Symptom                                            | Fix                                                                |
| ----------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------ |
| Using AutoComplete when you need to store an ID | Submit sends text instead of ID                    | Use ComboBox (stores ID) or use hidden field trick                 |
| No `min-length`set                            | AJAX fires on every single keystroke               | Add `min-length="2"`                                             |
| No `delay`set                                 | Too many rapid AJAX requests while typing          | Add `delay="300"`                                                |
| Reading `value()`expecting an ID              | Gets text string, not an ID                        | Use the `select`event to capture `e.dataItem.id`               |
| Using `change`to capture the selected item ID | `change`fires for typing too,`dataItem`is null | Use `select`event for picked-from-list,`change`for text clears |

---

## ❓ Interview Questions

**Q: What does the Kendo AutoComplete store as its value?**

> It stores only text — the `data-text-field` property of the selected item. It has no hidden ID like DropDownList or ComboBox. If you need the ID, use the `select` event and store `e.dataItem.id` in a hidden field.

**Q: What is the difference between `change` and `select` events?**

> `select` fires only when the user picks an item from the suggestion list — `e.dataItem` contains the full data object. `change` fires whenever the input value changes — including typing, clearing, and selecting. Use `select` when you need the selected item's data; use `change` for general value change tracking.

**Q: What do `min-length` and `delay` do and why are both important?**

> `min-length` prevents AJAX until N characters are typed — avoids empty or useless requests. `delay` waits N milliseconds after the last keystroke before firing — prevents a request on every single key press. Together they make the AutoComplete efficient: `min-length="2" delay="300"` means "search only when 2+ chars typed and the user has paused for 300ms."

**Q: How do you use AutoComplete to filter a Kendo Grid?**

> Handle the `select` event and call `grid.dataSource.filter({ field: "Name", operator: "contains", value: e.dataItem.name })`. Handle `change` and call `grid.dataSource.filter({})` when the input is cleared.
>
