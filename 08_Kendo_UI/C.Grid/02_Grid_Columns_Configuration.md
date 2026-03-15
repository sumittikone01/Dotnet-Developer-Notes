
# 02 — Grid Columns Configuration

---

## 🎯 One-Line Definition

> **Columns define what data the Grid shows, how it looks, how wide it is, how it formats values, and what commands appear — each `<column>` is one vertical slice of the table.**

---

## 🔑 Basic Column — The 4 Most Used Attributes

```cshtml
<column field="Name"
        title="Employee Name"
        width="200"
        format="{0:C0}" />

  field  → which property of the data object to show ("Name" maps to emp.Name)
  title  → the column header text (if omitted, uses field name)
  width  → column width in pixels
  format → how to format the value (dates, currency, decimals)
```

---

## 🔑 All Columns Together — Real Example

```cshtml
<columns>

    {{!-- Simple text column --}}
    <column field="Id"
            title="#"
            width="60" />

    {{!-- Text with custom title --}}
    <column field="Name"
            title="Full Name"
            width="200" />

    {{!-- Currency format --}}
    <column field="Salary"
            title="Salary"
            format="{0:C0}"
            width="130" />

    {{!-- Date format --}}
    <column field="HireDate"
            title="Hire Date"
            format="{0:MM/dd/yyyy}"
            width="130" />

    {{!-- Boolean — shows as checkbox --}}
    <column field="IsActive"
            title="Active"
            width="80" />

    {{!-- Command column — Edit and Delete buttons --}}
    <column title="Actions" width="160">
        <commands>
            <column-command text="Edit"   name="edit"    />
            <column-command text="Delete" name="destroy" />
        </commands>
    </column>

</columns>
```

---

## 🔑 Format Strings — How to Display Values

The `format` attribute uses `{0:formatCode}` syntax:

### Numbers

```cshtml
format="{0:N0}"    {{!-- 1,234          whole number with comma --}}
format="{0:N2}"    {{!-- 1,234.56       2 decimal places --}}
format="{0:C0}"    {{!-- $1,234         currency, no cents --}}
format="{0:C2}"    {{!-- $1,234.56      currency with cents --}}
format="{0:P0}"    {{!-- 75%            percentage --}}
format="{0:P2}"    {{!-- 75.50%         percentage with decimals --}}
```

### Dates

```cshtml
format="{0:MM/dd/yyyy}"      {{!-- 06/15/2021 --}}
format="{0:dd MMM yyyy}"     {{!-- 15 Jun 2021 --}}
format="{0:yyyy-MM-dd}"      {{!-- 2021-06-15 --}}
format="{0:d}"               {{!-- 6/15/2021 (short date) --}}
format="{0:D}"               {{!-- Monday, June 15, 2021 (long date) --}}
format="{0:dd/MM/yyyy HH:mm}"{{!-- 15/06/2021 09:30 (with time) --}}
```

---

## 🔑 Column Width — Fixed vs Flexible

```cshtml
{{!-- Fixed pixel width --}}
<column field="Id"         width="60"  />
<column field="Department" width="150" />

{{!-- No width set → column takes remaining space (flexible) --}}
{{!-- Leave ONE column without width to fill available space --}}
<column field="Name" title="Employee Name" />  {{!-- fills remaining --}}

{{!-- Percentage width --}}
<column field="Name" width="30%" />

{{!-- Best practice: set width on most columns, leave one flexible --}}
<columns>
    <column field="Id"         width="60"  />
    <column field="Name"       {{!-- no width → flexible --}} />
    <column field="Department" width="150" />
    <column field="Salary"     width="130" format="{0:C0}" />
    <column title="Actions"    width="160">
        <commands>
            <column-command name="edit"    />
            <column-command name="destroy" />
        </commands>
    </column>
</columns>
```

---

## 🔑 Column Templates — Custom HTML in Cells

When format strings aren't enough, use a template to render custom HTML:

```cshtml
{{!-- Colour the salary based on value --}}
<column field="Salary" title="Salary" width="140">
    <column-template>
        <span style="color: #= Salary > 80000 ? 'green' : 'black' #;
                     font-weight: #= Salary > 80000 ? 'bold' : 'normal' #">
            #= kendo.format('{0:C0}', Salary) #
        </span>
    </column-template>
</column>

{{!-- Status badge --}}
<column field="IsActive" title="Status" width="100">
    <column-template>
        # if (IsActive) { #
            <span class="badge badge-success">Active</span>
        # } else { #
            <span class="badge badge-danger">Inactive</span>
        # } #
    </column-template>
</column>

{{!-- Profile image --}}
<column field="PhotoUrl" title="Photo" width="80">
    <column-template>
        <img src="#= PhotoUrl #"
             alt="#= Name #"
             style="width:40px;height:40px;border-radius:50%;object-fit:cover" />
    </column-template>
</column>

{{!-- Custom action buttons --}}
<column title="Actions" width="200">
    <column-template>
        <button class="k-button k-button-sm viewBtn"
                data-id="#= Id #">View</button>
        <button class="k-button k-button-sm k-button-solid-error deleteBtn"
                data-id="#= Id #">Delete</button>
    </column-template>
</column>
```

**Template syntax:**

| Syntax                          | Output              | Use For                           |
| ------------------------------- | ------------------- | --------------------------------- |
| `#= Value #`                  | Rendered as HTML    | HTML, numbers, formatted values   |
| `#: Value #`                  | HTML-encoded (safe) | User-entered text (prevents XSS)  |
| `# if (...) { # ... # } #`    | Conditional block   | Show/hide based on value          |
| `kendo.format('{0:C0}', val)` | Format number       | Currency, percent inside template |

---

## 🔑 Locked (Frozen) Columns

Columns that stay visible while the user scrolls horizontally:

```cshtml
<kendo-grid name="employeeGrid" height="400">
    <columns>
        {{!-- These columns are frozen — always visible --}}
        <column field="Id"   title="#"    width="60"  locked="true" />
        <column field="Name" title="Name" width="200" locked="true" />

        {{!-- These scroll horizontally --}}
        <column field="Department" width="150" />
        <column field="Salary"     width="130" format="{0:C0}" />
        <column field="HireDate"   width="130" format="{0:MM/dd/yyyy}" />
        <column field="Email"      width="250" />
        <column field="Phone"      width="150" />
        <column field="Notes"      width="400" />
    </columns>
    <scrollable enabled="true" />
</kendo-grid>
```

---

## 🔑 Hidden Columns

Include a column in the data model but hide it from view:

```cshtml
{{!-- Hidden column — data is available but not displayed --}}
<column field="InternalCode" hidden="true" />
<column field="DatabaseId"   hidden="true" />

{{!-- Use case: hidden ID column for delete/edit operations --}}
<column field="Id" hidden="true" />
<column title="Actions" width="160">
    <column-template>
        <button onclick="deleteRow(#= Id #)">Delete</button>
    </column-template>
</column>
```

---

## 🔑 Column-Level Filterable and Sortable

Disable sorting or filtering on specific columns:

```cshtml
<columns>
    {{!-- Sortable and filterable (default) --}}
    <column field="Name" />

    {{!-- Not sortable --}}
    <column field="Notes" sortable="false" />

    {{!-- Not filterable --}}
    <column field="InternalId" filterable="false" />

    {{!-- Neither --}}
    <column field="Photo" sortable="false" filterable="false" />

    {{!-- Command columns — never sortable/filterable --}}
    <column title="Actions" width="160">
        <commands>
            <column-command name="edit" />
            <column-command name="destroy" />
        </commands>
    </column>
</columns>
```

---

## 🔑 Column Header Template — Custom Header HTML

```cshtml
{{!-- Custom header with icon --}}
<column field="Salary" width="130" format="{0:C0}">
    <header-template>
        <span>💰 Salary</span>
    </header-template>
</column>

{{!-- Header with tooltip --}}
<column field="KPI" width="100">
    <header-template>
        KPI Score
        <span class="k-icon k-i-information"
              title="Key Performance Indicator — scored 0 to 100">
        </span>
    </header-template>
</column>
```

---

## 🔑 schema.model fields — Matching Columns to Types

For editing to work correctly, field types in the schema must match your data:

```cshtml
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
            </fields>
        </model>
    </schema>
</datasource>

{{!-- Field types affect how the edit form renders:
     string  → text input
     number  → numeric input
     date    → date picker
     boolean → checkbox --}}
```

---

## 📊 Column Options Quick Reference

| Option         | Type    | What It Does                            |
| -------------- | ------- | --------------------------------------- |
| `field`      | string  | Data property to display                |
| `title`      | string  | Header text                             |
| `width`      | px or % | Column width                            |
| `format`     | string  | Value format `{0:C0}`                 |
| `hidden`     | bool    | Hide but include in model               |
| `locked`     | bool    | Freeze column (needs scrollable)        |
| `sortable`   | bool    | Allow/block sorting this column         |
| `filterable` | bool    | Show/hide filter for this column        |
| `editable`   | bool    | Allow/block editing this column         |
| `encoded`    | bool    | HTML-encode cell content (default true) |

---

## ❓ Interview Questions

**Q: What is the difference between `<column-template>` and `format`?**

> `format` is for simple value formatting using format codes like `{0:C0}`. `<column-template>` lets you write arbitrary HTML with JavaScript logic — conditional colours, badges, images, custom buttons — when format strings aren't enough.

**Q: What does `#= Value #` vs `#: Value #` mean in a column template?**

> `#= Value #` outputs the value directly and renders HTML tags. `#: Value #` HTML-encodes the value — safe for user-entered content to prevent XSS attacks. Use `#:` for any value a user typed.

**Q: How do you freeze a column so it stays visible while scrolling?**

> Add `locked="true"` to the column and `<scrollable enabled="true" />` to the grid. The locked columns stay fixed while the unlocked columns scroll horizontally.

**Q: Why must field types in `schema.model fields` match the actual data?**

> Kendo uses the type to determine which editor to show in the edit form. A `date` field gets a DatePicker, `number` gets a NumericTextBox, `boolean` gets a checkbox. Wrong types cause the wrong editor to appear or values to be mishandled.
>
