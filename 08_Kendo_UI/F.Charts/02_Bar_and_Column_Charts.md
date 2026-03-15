
# 02 — Bar and Column Charts

---

## 🎯 One-Line Definition

> **Bar and Column charts display categories side by side using rectangular bars — Column is vertical (height = value), Bar is horizontal (width = value) — both are best for comparing values across categories.**

---

## 🔑 Column vs Bar — The One Difference

```
COLUMN CHART (vertical):              BAR CHART (horizontal):
──────────────────────────────        ──────────────────────────────
$90k ┤                                IT      ████████████████
$80k ┤        ████                   HR      ████████
$70k ┤ ████   ████   ████            Finance ████████████
$60k ┤ ████   ████   ████   ████    Sales   ████████
     └──┬──────┬──────┬──────┬──           0   20k  40k  60k  80k
        IT     HR    Fin   Sales

Best for:                             Best for:
  - Few categories (< 7)               - Many categories (7+)
  - Category names are short           - Category names are long
  - Showing change over time           - Rankings and comparisons
```

They are the same type — just rotated. `ChartSeriesType.Column` = vertical, `ChartSeriesType.Bar` = horizontal.

---

## 🔑 Complete Column Chart Setup

```cshtml
<kendo-chart name="salaryChart" height="350">

    <chart-title text="Average Salary by Department" />

    <chart-legend position="ChartLegendPosition.Bottom" />

    {{!-- Series: one per data set to plot --}}
    <series>
        <series-item type="ChartSeriesType.Column"
                     field="avgSalary"
                     category-field="department"
                     name="Avg Salary"
                     color="#3498db">

            {{!-- Labels on top of each bar --}}
            <labels visible="true"
                    format="{0:C0}"
                    background="transparent"
                    color="#2c3e50" />

            {{!-- Tooltip when hovering --}}
            <series-item-tooltip visible="true"
                                 template="#= category #: #= kendo.format('{0:C0}', value) #" />
        </series-item>
    </series>

    {{!-- X-axis: categories (departments) --}}
    <category-axis>
        <category-axis-item>
            <labels rotation="-30" />   {{!-- tilt labels if they're long --}}
        </category-axis-item>
    </category-axis>

    {{!-- Y-axis: values (salary amounts) --}}
    <value-axis>
        <value-axis-item>
            <labels format="{0:C0}" />   {{!-- format axis labels as currency --}}
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetSalaryData", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

---

## 🔑 Complete Bar Chart Setup (Horizontal)

```cshtml
{{!-- Only difference: type="ChartSeriesType.Bar" --}}
<kendo-chart name="headcountChart" height="300">

    <chart-title text="Headcount by Department" />

    <series>
        <series-item type="ChartSeriesType.Bar"     {{!-- ← horizontal --}}
                     field="count"
                     category-field="department"
                     name="Employees"
                     color="#2ecc71">
            <labels visible="true" format="{0} staff" />
            <series-item-tooltip visible="true"
                                 template="#= category #: #= value # employees" />
        </series-item>
    </series>

    <value-axis>
        <value-axis-item min="0">
            <labels format="{0}" />
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetHeadcountData", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

---

## 🔑 Multiple Series — Grouped Bars

Show more than one metric side by side for each category:

```cshtml
<kendo-chart name="comparisonChart" height="400">

    <chart-title text="Salary vs Budget by Department" />

    <chart-legend position="ChartLegendPosition.Bottom" />

    <series>
        {{!-- Series 1: Actual Salary --}}
        <series-item type="ChartSeriesType.Column"
                     field="actualSalary"
                     category-field="department"
                     name="Actual Salary"
                     color="#3498db">
        </series-item>

        {{!-- Series 2: Budget --}}
        <series-item type="ChartSeriesType.Column"
                     field="budgetSalary"
                     category-field="department"
                     name="Budget"
                     color="#e74c3c">
        </series-item>
    </series>

    <value-axis>
        <value-axis-item>
            <labels format="{0:C0}" />
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetComparisonData", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

```csharp
// Controller — returns both values per category
public JsonResult GetComparisonData()
{
    var data = _db.Departments.Select(d => new {
        department   = d.Name,
        actualSalary = _db.Employees
                          .Where(e => e.DepartmentId == d.Id)
                          .Average(e => (decimal?)e.Salary) ?? 0,
        budgetSalary = d.SalaryBudget
    }).ToList();

    return Json(data);
}
// Returns: [ {"department":"IT","actualSalary":75000,"budgetSalary":80000}, ... ]
```

```
Result — grouped bars per department:
         ████ ░░░░
$80k ┤   ████ ░░░░       ████ ░░░░
$70k ┤   ████ ░░░░  ████ ████ ░░░░
$60k ┤   ████ ░░░░  ████ ████ ░░░░  ████ ░░░░
         IT          HR          Finance
  ■ Actual Salary   ░ Budget
```

---

## 🔑 Stacked Bars

Show how parts make up a whole:

```cshtml
<series>
    <series-item type="ChartSeriesType.Column"
                 field="juniorCount"
                 category-field="department"
                 name="Junior"
                 color="#3498db"
                 stack="true">    {{!-- stack="true" stacks this series --}}
    </series-item>
    <series-item type="ChartSeriesType.Column"
                 field="seniorCount"
                 category-field="department"
                 name="Senior"
                 color="#2ecc71"
                 stack="true">
    </series-item>
    <series-item type="ChartSeriesType.Column"
                 field="leadCount"
                 category-field="department"
                 name="Lead"
                 color="#e67e22"
                 stack="true">
    </series-item>
</series>
```

```
Stacked result:
      ░░░░░  ← Lead
      ██████  ← Senior
      ██████  ← Junior
       IT
Total height = IT total headcount
```

---

## 🔑 Category Axis Configuration

```cshtml
<category-axis>
    <category-axis-item>

        {{!-- Rotate labels if they overlap --}}
        <labels rotation="-45" font="11px Arial" />

        {{!-- Grid lines between categories --}}
        <major-grid-lines visible="false" />

        {{!-- Axis line --}}
        <axis-crossing-value>0</axis-crossing-value>

    </category-axis-item>
</category-axis>
```

---

## 🔑 Value Axis Configuration

```cshtml
<value-axis>
    <value-axis-item min="0"
                     max="100000">    {{!-- optional: fix the range --}}
        <labels format="{0:C0}" />    {{!-- format each axis number --}}
        <major-grid-lines visible="true"
                          color="#eeeeee" />  {{!-- horizontal grid lines --}}
        <title text="Annual Salary" rotation="-90" />
    </value-axis-item>
</value-axis>
```

---

## 🔑 Tooltip Configuration

```cshtml
{{!-- Global tooltip for all series (on chart level) --}}
<tooltip visible="true"
         shared="true"
         template="<b>#= category #</b><br/>
                   #= series.name #: #= kendo.format('{0:C0}', value) #" />

{{!-- Per-series tooltip (overrides global) --}}
<series-item-tooltip visible="true"
                     template="#= category # ➜ #= kendo.format('{0:C0}', value) #"
                     background="#2c3e50"
                     color="white"
                     border-color="#2c3e50" />
```

---

## 🔑 Controller — Providing Data for Multiple Series

```csharp
// Returns one array — each object has all series values
public JsonResult GetDeptStats()
{
    var data = _db.Employees
        .GroupBy(e => e.Department)
        .Select(g => new {
            department     = g.Key,
            avgSalary      = Math.Round(g.Average(e => e.Salary), 0),
            headcount      = g.Count(),
            activeCount    = g.Count(e => e.IsActive),
            maxSalary      = g.Max(e => e.Salary)
        })
        .OrderByDescending(x => x.avgSalary)
        .ToList();

    return Json(data);
}
```

```json
[
  { "department": "IT",      "avgSalary": 75000, "headcount": 45, "activeCount": 42 },
  { "department": "Finance", "avgSalary": 65000, "headcount": 30, "activeCount": 28 },
  { "department": "HR",      "avgSalary": 55000, "headcount": 20, "activeCount": 20 },
  { "department": "Sales",   "avgSalary": 48000, "headcount": 25, "activeCount": 22 }
]
```

---

## 🔑 Click Event — Drill Down from Chart

When the user clicks a bar, load more detail:

```javascript
var chart = $("#salaryChart").data("kendoChart");

chart.bind("seriesClick", function(e) {
    // e.category = the clicked category ("IT")
    // e.value    = the numeric value (75000)
    // e.series.name = series name ("Avg Salary")
    // e.dataItem = the full data object

    console.log("Clicked:", e.category, "→", e.value);

    var dept = e.category;

    // Drill down: filter the grid to show only this department
    var grid = $("#employeeGrid").data("kendoGrid");
    grid.dataSource.filter({
        field:    "Department",
        operator: "eq",
        value:    dept
    });

    // OR: open a Window with department details
    openDeptWindow(dept, e.value);
});
```

---

## 📊 Quick Reference — Column vs Bar Decision

| Situation                          | Use                              |
| ---------------------------------- | -------------------------------- |
| ≤ 6 categories, short names       | Column (vertical)                |
| 7+ categories OR long names        | Bar (horizontal)                 |
| Showing change over time by period | Column                           |
| Ranking items                      | Bar                              |
| Comparing parts of a whole         | Stacked Column/Bar               |
| Comparing two metrics per category | Grouped Column (multiple series) |

---

## ⚠️ Common Mistakes

| Mistake                                  | Symptom                                          | Fix                                                                 |
| ---------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------- |
| `field`name case mismatch              | Chart renders empty                              | `field="avgSalary"`must match JSON `"avgSalary"`exactly         |
| No `category-field`                    | All bars stack on same X position                | Always set `category-field="department"`                          |
| Too many categories                      | X-axis labels overlap                            | Use Bar (horizontal) or rotate labels:`<labels rotation="-45" />` |
| Stacked series but `stack`only on some | Only some bars stack                             | Set `stack="true"`on ALL series you want stacked                  |
| `value-axis`min not set                | Axis starts at non-zero, exaggerates differences | Set `min="0"`unless intentional                                   |

---

## ❓ Interview Questions

**Q: What is the difference between a Bar chart and a Column chart in Kendo?**

> They are the same chart type, just rotated. Column uses `ChartSeriesType.Column` and draws vertical bars (height = value). Bar uses `ChartSeriesType.Bar` and draws horizontal bars (width = value). Use Column for fewer categories with short names, Bar for many categories or long names.

**Q: How do you show multiple data series on one chart?**

> Add multiple `<series-item>` elements inside `<series>`. Each has its own `field`, `name`, and `color`. They render as grouped bars by default, or stacked bars if `stack="true"` is set.

**Q: How do you handle a bar click to drill down into details?**

> Bind the chart's `seriesClick` event. The event object provides `e.category` (the clicked label), `e.value` (the number), and `e.dataItem` (the full data object). Use these to filter a grid, open a window, or load more data.

**Q: How do you format the value axis labels as currency?**

> Add `<labels format="{0:C0}" />` inside `<value-axis-item>`. The `{0}` is the placeholder and `C0` is the currency format with no cents.
>
