
# 03 — Kendo DatePicker

---

## 🎯 One-Line Definition

> **The Kendo DatePicker replaces a plain date input with a styled calendar popup — it handles formatting, min/max date constraints, and integrates directly with your ASP.NET model's `DateTime` properties.**

---

## 🔑 Plain HTML vs Kendo DatePicker

```
Plain HTML <input type="date">:     Kendo DatePicker:
────────────────────────────────     ─────────────────────────────────────
[2021-06-15        ]                 [06/15/2021                     📅]

Browser-dependent UI                 Consistent UI across all browsers
Fixed format (yyyy-MM-dd)            Any format you choose
No min/max styling                   Min/max visually greys out dates
No week numbers                      Week numbers optional
No footer "Today" link               "Today" button at bottom
No programmatic control              Full JavaScript API
```

---

## 🔑 Basic Setup

```cshtml
{{!-- Minimal --}}
<kendo-datepicker name="HireDate" />


{{!-- With common options --}}
<kendo-datepicker name="HireDate"
                  value="@Model.HireDate"
                  format="MM/dd/yyyy"
                  placeholder="Select hire date..."
                  min="new DateTime(2000,1,1)"
                  max="DateTime.Today" />
```

---

## 🔑 All Key Attributes

```cshtml
<kendo-datepicker name="HireDate"
                  value="@Model.HireDate"
                  format="MM/dd/yyyy"
                  placeholder="Pick a date..."
                  min="new DateTime(2000,1,1)"
                  max="DateTime.Today.AddYears(1)"
                  start="CalendarView.Month"
                  depth="CalendarView.Month"
                  week-number="true"
                  footer="Today is: #=kendo.toString(value,'MM/dd/yyyy')#"
                  enabled="true"
                  readonly="false" />
```

| Attribute       | What It Does                     | Example                           |
| --------------- | -------------------------------- | --------------------------------- |
| `name`        | Field name — binds to model     | `name="HireDate"`               |
| `value`       | Pre-selected date                | `value="@Model.HireDate"`       |
| `format`      | How date displays in input       | `"MM/dd/yyyy"`,`"dd/MM/yyyy"` |
| `placeholder` | Hint text when empty             | `"Select a date"`               |
| `min`         | Earliest selectable date         | `new DateTime(2000,1,1)`        |
| `max`         | Latest selectable date           | `DateTime.Today`                |
| `start`       | Which view opens first           | `CalendarView.Month`            |
| `depth`       | Deepest view user can reach      | `CalendarView.Month`            |
| `week-number` | Show ISO week numbers            | `true`/`false`                |
| `enabled`     | Enable or disable                | `false`= greyed out             |
| `readonly`    | Read-only (shows value, no edit) | `true`                          |

---

## 🔑 Format Strings — How the Date Displays

The `format` controls what the user sees in the text box:

```cshtml
format="MM/dd/yyyy"        → 06/15/2021         (US style)
format="dd/MM/yyyy"        → 15/06/2021         (EU style)
format="dd MMM yyyy"       → 15 Jun 2021
format="MMMM dd, yyyy"     → June 15, 2021
format="yyyy-MM-dd"        → 2021-06-15         (ISO)
format="dd/MM/yyyy HH:mm"  → 15/06/2021 09:30   (with time)
format="ddd, dd MMM yyyy"  → Tue, 15 Jun 2021
```

```
Format tokens:
  d    → day number (1-31)          dd   → day zero-padded (01-31)
  ddd  → short day name (Mon)       dddd → full day name (Monday)
  M    → month number (1-12)        MM   → month zero-padded (01-12)
  MMM  → short month (Jun)          MMMM → full month (June)
  yy   → 2-digit year (21)          yyyy → 4-digit year (2021)
  H    → hour 24h (0-23)            HH   → hour 24h padded (00-23)
  h    → hour 12h (1-12)            hh   → hour 12h padded (01-12)
  m    → minutes (0-59)             mm   → minutes padded (00-59)
```

---

## 🔑 Calendar Views — Controlling Navigation Depth

`start` = which view opens when the calendar pops up.
`depth` = how far down the user can navigate (stops here).

```
Views from highest to lowest:
  century → decade → year → month

  CalendarView.Century  → shows years 2000-2099
  CalendarView.Decade   → shows years 2020-2029
  CalendarView.Year     → shows months Jan-Dec
  CalendarView.Month    → shows days 1-31 (most common)
```

```cshtml
{{!-- Normal date picker: opens at Month, user can navigate up --}}
<kendo-datepicker start="CalendarView.Month"
                  depth="CalendarView.Month" />

{{!-- Month picker: opens at Year (shows months), stops at Month --}}
{{!-- User picks a month, not a specific day --}}
<kendo-datepicker start="CalendarView.Year"
                  depth="CalendarView.Year"
                  format="MMMM yyyy" />

{{!-- Year picker: opens at Decade, stops at Year --}}
<kendo-datepicker start="CalendarView.Decade"
                  depth="CalendarView.Year"
                  format="yyyy" />
```

---

## 🔑 Min and Max Dates — Common Patterns

```cshtml
{{!-- Only allow past dates (e.g., birth date) --}}
<kendo-datepicker name="DateOfBirth"
                  max="DateTime.Today" />


{{!-- Only allow future dates (e.g., expiry date) --}}
<kendo-datepicker name="ExpiryDate"
                  min="DateTime.Today.AddDays(1)" />


{{!-- Date range: 2020 to 5 years from now --}}
<kendo-datepicker name="ContractDate"
                  min="new DateTime(2020, 1, 1)"
                  max="DateTime.Today.AddYears(5)" />


{{!-- Disable past dates (anything before today greys out) --}}
<kendo-datepicker name="AppointmentDate"
                  min="DateTime.Today" />
```

---

## 🔑 JavaScript API

```javascript
var dp = $("#HireDate").data("kendoDatePicker");

// ── Reading the value ──────────────────────────────────────────
dp.value()           // returns a JavaScript Date object (or null)
                     // e.g.: Tue Jun 15 2021 00:00:00

// Convert to string:
var date = dp.value();
if (date) {
    var formatted = kendo.toString(date, "MM/dd/yyyy");  // "06/15/2021"
    var isoStr    = kendo.toString(date, "yyyy-MM-dd");  // "2021-06-15"
}

// ── Setting the value ──────────────────────────────────────────
dp.value(new Date(2021, 5, 15));    // June 15, 2021 (month is 0-indexed!)
dp.value(new Date("2021-06-15"));   // same — ISO string constructor
dp.value(null);                      // clear the date
dp.value(new Date());                // set to today

// ── Enable / Disable ──────────────────────────────────────────
dp.enable(false)    // user can't change the date
dp.enable(true)

// ── Read-only (show value but no editing) ─────────────────────
dp.readonly(true)
dp.readonly(false)

// ── Change min/max dynamically ────────────────────────────────
dp.min(new Date(2023, 0, 1));   // set new minimum date
dp.max(new Date(2025, 11, 31)); // set new maximum date

// ── Open / Close programmatically ────────────────────────────
dp.open()   // show calendar
dp.close()  // hide calendar
```

---

## 🔑 Events

```javascript
var dp = $("#HireDate").data("kendoDatePicker");

// ── Change: fires when date is picked or typed ────────────────
dp.bind("change", function(e) {
    var date = this.value();  // JavaScript Date object or null

    if (date) {
        console.log("Date selected:", kendo.toString(date, "MM/dd/yyyy"));

        // Calculate days from today
        var today    = new Date();
        var diffMs   = date - today;
        var diffDays = Math.ceil(diffMs / (1000 * 60 * 60 * 24));
        console.log("Days from today:", diffDays);
    } else {
        console.log("Date cleared");
    }
});

// ── Open: fires when calendar opens ───────────────────────────
dp.bind("open", function() {
    console.log("Calendar opened");
});

// ── Close: fires when calendar closes ─────────────────────────
dp.bind("close", function() {
    // Good place to validate the date range
    var start = $("#StartDate").data("kendoDatePicker").value();
    var end   = $("#EndDate").data("kendoDatePicker").value();

    if (start && end && start > end) {
        showNotification("End date must be after start date", "error");
        this.value(null);  // clear invalid end date
    }
});
```

---

## 🔑 Date Range Pattern — Start and End Date

A very common real-world pattern: enforce that End Date is always after Start Date.

```cshtml
<div>
    <label>Start Date</label>
    <kendo-datepicker name="StartDate"
                      value="@Model.StartDate"
                      format="MM/dd/yyyy" />
</div>

<div>
    <label>End Date</label>
    <kendo-datepicker name="EndDate"
                      value="@Model.EndDate"
                      format="MM/dd/yyyy" />
</div>
```

```javascript
$(function() {
    var startPicker = $("#StartDate").data("kendoDatePicker");
    var endPicker   = $("#EndDate").data("kendoDatePicker");

    // When start date changes → update end date's minimum
    startPicker.bind("change", function() {
        var start = this.value();
        if (start) {
            // End date can't be before start date
            endPicker.min(start);

            // If current end date is now invalid, clear it
            var currentEnd = endPicker.value();
            if (currentEnd && currentEnd < start) {
                endPicker.value(null);
            }
        }
    });

    // When end date changes → update start date's maximum
    endPicker.bind("change", function() {
        var end = this.value();
        if (end) {
            startPicker.max(end);
        }
    });
});
```

---

## 🔑 Sending Dates to the Server — What Actually Travels

```javascript
// What Kendo sends in a form submit or AJAX call:
// The date is serialized as a string in ISO format:
// "HireDate": "2021-06-15T00:00:00"
// OR the short format: "2021-06-15"

// In jQuery AJAX — get the value and format it:
var dp   = $("#HireDate").data("kendoDatePicker");
var date = dp.value();

var payload = {
    name:     $("#Name").val(),
    hireDate: date ? kendo.toString(date, "yyyy-MM-dd") : null
};

$.post("/Employee/Create", payload, function(r) { ... });
```

```csharp
// Controller receives it fine — ASP.NET parses ISO date strings
public JsonResult Create(string name, DateTime? hireDate)
{
    // hireDate = June 15, 2021 — properly deserialized
}

// OR via model binding
public JsonResult Create(Employee employee)
{
    // employee.HireDate = June 15, 2021
}
```

---

## 🔑 DatePicker in Grid Popup Edit Form

```html
{{!-- Inside a Kendo template for popup edit --}}
<div class="form-group">
    <label>Hire Date</label>
    <input name="HireDate"
           data-bind="value: HireDate"
           data-role="datepicker"
           data-format="MM/dd/yyyy"
           style="width:100%" />
</div>
```

---

## ⚠️ Common Mistakes

| Mistake                                                 | Symptom                                         | Fix                                                        |
| ------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| `new Date()`months are 0-indexed                      | `new Date(2021, 6, 15)`= July 15, not June 15 | Month is 0-based:`new Date(2021, 5, 15)`= June 15        |
| Comparing dates with `==`or `===`                   | Always false even for same date                 | Use `.getTime()`:`date1.getTime() === date2.getTime()` |
| Format mismatch between display and server              | Server receives wrong date or null              | Use `yyyy-MM-dd`for server, any format for display       |
| `value`is null after picking                          | Parsing failed — format mismatch               | Ensure `format`matches what user types manually          |
| `min`set to `DateTime.Today`but today is greyed out | `min`is inclusive by default                  | To exclude today:`min="DateTime.Today.AddDays(1)"`       |

---

## ❓ Interview Questions

**Q: How does the Kendo DatePicker differ from `<input type="date">`?**

> Kendo DatePicker is consistent across all browsers, supports any display format, has min/max constraints with visual greying, supports week numbers and "Today" button, and provides a full JavaScript API. The native HTML date input varies between browsers and offers limited customization.

**Q: What does `depth="CalendarView.Year"` do?**

> It stops the calendar navigation at the Year view — the user can only navigate to months, not individual days. Used to create a "month picker" where the user selects a month and year rather than a specific day.

**Q: How do you get the selected date as a JavaScript Date object?**

> `var date = $("#myDatePicker").data("kendoDatePicker").value()` — returns a JS Date object, or null if nothing is selected.

**Q: Why is `new Date(2021, 5, 15)` June 15 and not May 15?**

> JavaScript's Date constructor uses zero-based months — January = 0, February = 1... June = 5. This is a common gotcha. Use `new Date("2021-06-15")` with an ISO string to avoid confusion.
>
