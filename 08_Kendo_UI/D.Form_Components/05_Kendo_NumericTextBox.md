
# 05 — Kendo NumericTextBox

---

## 🎯 One-Line Definition

> **The Kendo NumericTextBox is a styled number input that enforces numeric-only entry, formats the displayed value (currency, percentage, decimals), and provides up/down spin buttons — all while giving you a clean JavaScript API to read and set the number.**

---

## 🔑 Plain Input vs Kendo NumericTextBox

```
Plain <input type="number">:         Kendo NumericTextBox:
──────────────────────────────────    ──────────────────────────────────
[75000                        ▲▼]    [$75,000.00                    ▲▼]
User types "abc" → field accepts it   User types "abc" → blocked, only numbers
No formatting                         Formatted as currency, %, decimal
No min/max visual feedback            Min/max enforced with visual cue
No step control                       Step size configurable (e.g. 1000)
```

---

## 🔑 Basic Setup

```cshtml
{{!-- Simple number input --}}
<kendo-numerictextbox name="Salary" />


{{!-- With common options --}}
<kendo-numerictextbox name="Salary"
                      value="@Model.Salary"
                      format="c2"
                      min="0"
                      max="999999"
                      step="500"
                      decimals="2"
                      placeholder="Enter salary..." />
```

---

## 🔑 Format Codes — How the Number Displays

The `format` attribute controls what the user sees:

```cshtml
{{!-- Currency --}}
<kendo-numerictextbox format="c0"  />   → $75,000
<kendo-numerictextbox format="c2"  />   → $75,000.00
<kendo-numerictextbox format="c"   />   → $75,000.00 (default currency decimals)

{{!-- Plain Numbers --}}
<kendo-numerictextbox format="n0"  />   → 75,000        (commas, no decimals)
<kendo-numerictextbox format="n2"  />   → 75,000.00     (commas, 2 decimals)
<kendo-numerictextbox format="n3"  />   → 75,000.000    (3 decimals)

{{!-- Percentage --}}
<kendo-numerictextbox format="p0"  />   → 75%
<kendo-numerictextbox format="p2"  />   → 75.50%
<kendo-numerictextbox format="p"   />   → 75% (default)

{{!-- Custom format --}}
<kendo-numerictextbox format="##,#.##" />  → 75,000.50  (custom pattern)
```

```
Format code breakdown:
  c = currency     ($, £, € based on browser locale)
  n = number       (plain number with thousands separator)
  p = percentage   (value 0.75 displays as 75%)
  0 = zero decimals
  2 = two decimals
```

---

## 🔑 All Key Attributes

```cshtml
<kendo-numerictextbox name="Salary"
                      value="@Model.Salary"
                      format="c0"
                      min="10000"
                      max="1000000"
                      step="1000"
                      decimals="0"
                      placeholder="Enter salary..."
                      spinners="true"
                      enabled="true"
                      readonly="false" />
```

| Attribute       | What It Does                       | Example                    |
| --------------- | ---------------------------------- | -------------------------- |
| `name`        | Field name — binds to model       | `name="Salary"`          |
| `value`       | Initial number value               | `value="@Model.Salary"`  |
| `format`      | How to display                     | `"c0"`,`"n2"`,`"p0"` |
| `min`         | Minimum allowed value              | `min="0"`                |
| `max`         | Maximum allowed value              | `max="999999"`           |
| `step`        | How much ▲▼ buttons change value | `step="1000"`            |
| `decimals`    | Max decimal places allowed         | `decimals="2"`           |
| `placeholder` | Hint text when empty               | `"Enter amount"`         |
| `spinners`    | Show ▲▼ arrow buttons            | `true`/`false`         |
| `enabled`     | Enable or disable                  | `false`= greyed out      |
| `readonly`    | Show value, no editing             | `true`                   |

---

## 🔑 `step` — The Spin Button Increment

```cshtml
{{!-- Salary in steps of 1000 --}}
<kendo-numerictextbox name="Salary"
                      value="50000"
                      step="1000"
                      format="c0" />
{{!-- Click ▲: 50000 → 51000 → 52000...
     Click ▼: 50000 → 49000 → 48000... --}}


{{!-- Percentage in steps of 0.01 (1%) --}}
<kendo-numerictextbox name="DiscountRate"
                      value="0.10"
                      step="0.01"
                      format="p0"
                      min="0"
                      max="1" />
{{!-- Stores: 0.10 (not 10)
     Displays: 10%
     Click ▲: 0.10 → 0.11 → 0.12...  Displays: 11%, 12%... --}}


{{!-- Quantity in steps of 1 --}}
<kendo-numerictextbox name="Quantity"
                      value="1"
                      step="1"
                      min="1"
                      max="100"
                      format="n0" />
```

---

## 🔑 `decimals` vs `format` — Know the Difference

```
format   = how the number DISPLAYS to the user
decimals = how many decimal places are ALLOWED as input

They are independent — both can matter:

<kendo-numerictextbox format="c2" decimals="2" />
  format="c2"   → displays as $75,000.00
  decimals="2"  → allows up to 2 decimal places input

<kendo-numerictextbox format="n0" decimals="0" />
  format="n0"   → displays as 75,000 (no decimals shown)
  decimals="0"  → user cannot type decimal point at all

<kendo-numerictextbox format="c2" decimals="0" />
  format="c2"   → displays as $75,000.00
  decimals="0"  → BUT user can't type decimals (inconsistent!)
                  → keep format and decimals in sync
```

---

## 🔑 Percentage — Important: Value vs Display

```cshtml
{{!-- IMPORTANT: percentage values --}}
<kendo-numerictextbox name="DiscountRate"
                      format="p0"
                      min="0"
                      max="1"
                      step="0.05"
                      value="0.25" />

{{!--
  Displays: 25%          ← what the user sees
  Stores:   0.25         ← what gets submitted to server
  
  If you want to store/work with 0-100 range instead:
  Use format="n0" with min=0, max=100 and handle the /100 yourself
--}}
```

```csharp
// Controller receives the raw value (0.25, not 25)
public JsonResult Save(decimal discountRate)  // receives 0.25
{
    // If you displayed as p0, value = 0.25 = 25%
}
```

---

## 🔑 JavaScript API

```javascript
var num = $("#Salary").data("kendoNumericTextBox");

// ── Reading the value ──────────────────────────────────────────
num.value()         // returns the raw number: 75000  (NOT "$75,000")
                    // returns null if empty

// Always check for null:
var salary = num.value();
if (salary !== null) {
    console.log("Salary:", salary);  // 75000
}

// ── Setting the value ──────────────────────────────────────────
num.value(80000)    // set to 80000 (displays as $80,000 with c0 format)
num.value(null)     // clear the field
num.value(0)        // set to zero

// ── Enable / Disable ──────────────────────────────────────────
num.enable(false)   // greys out, user can't change
num.enable(true)

// ── Read-only ─────────────────────────────────────────────────
num.readonly(true)
num.readonly(false)

// ── Change min/max dynamically ────────────────────────────────
num.min(5000)       // update minimum
num.max(500000)     // update maximum

// ── Focus the field ───────────────────────────────────────────
num.focus()
```

---

## 🔑 Events

```javascript
var num = $("#Salary").data("kendoNumericTextBox");

// ── Change: fires when value changes (after user leaves field or spins) ─
num.bind("change", function() {
    var value = this.value();   // the raw number

    if (value === null) {
        console.log("Field is empty");
        return;
    }

    console.log("Salary changed to:", value);

    // Real-time tax calculation
    var tax        = value * 0.2;
    var netSalary  = value - tax;
    $("#TaxDisplay").text(kendo.format("{0:c0}", tax));
    $("#NetSalary").text(kendo.format("{0:c0}", netSalary));

    // Validate against budget
    if (value > 150000) {
        showNotification("Salary exceeds budget cap of $150,000", "warning");
    }
});

// ── Spin: fires when ▲ or ▼ button is clicked ─────────────────
num.bind("spin", function() {
    console.log("Spin to:", this.value());
});
```

---

## 🔑 Reading the Value — The Right Way

```javascript
// ── In jQuery AJAX — always read via Kendo API, not raw input ─
var salary = $("#Salary").data("kendoNumericTextBox").value();
// Returns: 75000  (the number)
// NOT "$75,000" which is what the raw input contains

// Wrong — reading raw HTML input:
var wrong = $("#Salary").val();
// Returns: "$75,000.00"  (formatted string — useless for server)

// Always use:
var payload = {
    name:   $("#Name").val(),
    salary: $("#Salary").data("kendoNumericTextBox").value()
    //      ↑ always use .data("kendoNumericTextBox").value()
};
$.post("/Employee/Save", payload, function(r) { ... });
```

---

## 🔑 Real-World Examples

### Salary with budget warning

```cshtml
<div class="form-group">
    <label>Annual Salary</label>
    <kendo-numerictextbox name="Salary"
                          value="@Model.Salary"
                          format="c0"
                          min="20000"
                          max="500000"
                          step="1000"
                          decimals="0" />
    <small id="salaryHint" class="text-muted">
        Range: $20,000 — $500,000
    </small>
</div>
```

### Quantity with min 1

```cshtml
<div class="form-group">
    <label>Quantity</label>
    <kendo-numerictextbox name="Quantity"
                          value="1"
                          min="1"
                          max="9999"
                          step="1"
                          format="n0"
                          decimals="0"
                          spinners="true" />
</div>
```

### Tax rate as percentage

```cshtml
<div class="form-group">
    <label>Tax Rate</label>
    <kendo-numerictextbox name="TaxRate"
                          value="@Model.TaxRate"
                          format="p2"
                          min="0"
                          max="1"
                          step="0.01"
                          decimals="4" />
    <small class="text-muted">
        Enter as decimal: 0.20 = 20%
    </small>
</div>
```

### Discount percentage in 0-100 range

```cshtml
{{!-- When you prefer to work with 0-100 scale, not 0.0-1.0 --}}
<kendo-numerictextbox name="DiscountPercent"
                      value="@Model.DiscountPercent"
                      format="n0"
                      min="0"
                      max="100"
                      step="5"
                      decimals="0" />
<small class="text-muted">Enter 0 – 100 (e.g. 20 = 20% off)</small>
```

---

## 🔑 NumericTextBox in Kendo Grid Edit Form

```html
{{!-- Inside a Kendo popup template --}}
<div class="form-group mb-3">
    <label>Salary</label>
    <input name="Salary"
           data-bind="value: Salary"
           data-role="numerictextbox"
           data-format="c0"
           data-min="0"
           data-max="999999"
           data-step="1000"
           data-decimals="0"
           style="width:100%" />
    <span data-for="Salary" class="k-invalid-msg text-danger"></span>
</div>
```

---

## ⚠️ Common Mistakes

| Mistake                                                                                              | Symptom                                                   | Fix                                                            |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------------------------- |
| Reading `$("#Salary").val()`                           | Gets `"$75,000.00"`string, not a number | Always use `.data("kendoNumericTextBox").value()`       |                                                                |
| Displaying `p0`but storing `25`instead of `0.25`                                               | 25x too large in the database                             | `format="p"`works with 0.0–1.0 range. Store `0.25`for 25% |
| `format="c0"`but `decimals="2"`                                                                  | Confusing UX — displays no cents but allows typing cents | Keep format and decimals consistent                            |
| Not handling `null`from `.value()`                                                               | JS error when doing math on null                          | Always check `if (value !== null)`before calculations        |
| Setting very small `step`on integer field                                                          | Spin goes 1001, 1002, 1003... frustrating                 | Match `step`to meaningful increments for the field           |

---

## ❓ Interview Questions

**Q: What does the Kendo NumericTextBox `format` attribute do?**

> It controls how the number is displayed to the user — `c0` shows currency without cents, `n2` shows a number with 2 decimal places, `p0` shows a percentage. The actual value stored internally is always the raw number, never the formatted string.

**Q: How do you read the numeric value from a NumericTextBox in JavaScript?**

> Always use `$("#fieldId").data("kendoNumericTextBox").value()` — this returns the raw number (e.g. `75000`). Reading `$("#fieldId").val()` returns the formatted display string (e.g. `"$75,000"`) which cannot be used for calculation or submission.

**Q: What is the difference between `format` and `decimals`?**

> `format` controls the display — `c2` shows `$75,000.00`. `decimals` controls the maximum decimal places the user can input. They should be kept in sync — `format="c2"` with `decimals="2"` makes sense; `format="c2"` with `decimals="0"` would display cents but prevent typing them.

**Q: If you use `format="p0"`, what value does the server receive?**

> The raw decimal — `0.25` is stored and sent when the display shows `25%`. The percentage format divides by 100 for display and multiplies for storage. If you want to work in a 0–100 scale, use `format="n0"` and handle the conversion yourself.
>
