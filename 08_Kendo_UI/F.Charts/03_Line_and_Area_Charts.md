
# 03 — Line and Area Charts

---

## 🎯 One-Line Definition

> **Line charts connect data points with a line to show trends over time — Area charts do the same but fill the space under the line, making volume and magnitude more visually obvious.**

---

## 🔑 Line vs Area — The One Difference

```
LINE CHART:                           AREA CHART:
────────────────────────────────      ────────────────────────────────
$90k ┤           •                   $90k ┤           •
$80k ┤       •   |                   $80k ┤       •   |
$70k ┤   •  /    |  •                $70k ┤   •░░/░░░░|░░•
$60k ┤  / \/     \/                  $60k ┤  /░░\░░░░░\/░░░░
     └──┬──┬──┬──┬──                      └──┬──┬──┬──┬──
        Q1 Q2 Q3 Q4                           Q1 Q2 Q3 Q4

Shows the TREND clearly               Shows trend PLUS the volume
Clean, precise                         More visual impact

Best for:                             Best for:
  - Precise value reading              - Emphasising magnitude
  - Multiple series (less overlap)     - Single series (filled area)
  - Stock-like data                    - Showing volume over time
```

---

## 🔑 Complete Line Chart Setup

```cshtml
<kendo-chart name="revenueChart" height="350">

    <chart-title text="Monthly Revenue — 2024" />

    <chart-legend position="ChartLegendPosition.Bottom" />

    <series>
        <series-item type="ChartSeriesType.Line"
                     field="revenue"
                     category-field="month"
                     name="Revenue"
                     color="#3498db"
                     width="2">        {{!-- line thickness --}}

            {{!-- Dots at each data point --}}
            <markers visible="true"
                     size="6"
                     type="ChartMarkerType.Circle"
                     background="#3498db" />

            {{!-- Value labels at each point --}}
            <labels visible="true"
                    format="{0:C0}"
                    position="ChartSeriesLabelsPosition.Above"
                    background="transparent" />

            {{!-- Hover tooltip --}}
            <series-item-tooltip visible="true"
                                 template="#= category #: #= kendo.format('{0:C0}', value) #" />
        </series-item>
    </series>

    {{!-- X-axis: time categories --}}
    <category-axis>
        <category-axis-item>
            <labels rotation="-30" />
        </category-axis-item>
    </category-axis>

    {{!-- Y-axis: revenue values --}}
    <value-axis>
        <value-axis-item min="0">
            <labels format="{0:C0}" />
            <major-grid-lines color="#eeeeee" />
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetMonthlyRevenue", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

---

## 🔑 Complete Area Chart Setup

```cshtml
<kendo-chart name="headcountTrendChart" height="350">

    <chart-title text="Headcount Growth Over Time" />

    <series>
        <series-item type="ChartSeriesType.Area"     {{!-- ← Area fills below --}}
                     field="headcount"
                     category-field="quarter"
                     name="Total Employees"
                     color="#3498db"
                     opacity="0.4"        {{!-- fill transparency 0.0-1.0 --}}
                     line-style="ChartLineStyle.Smooth">  {{!-- smooth curve --}}

            <markers visible="true" size="5" />
            <series-item-tooltip visible="true"
                                 template="Q#= category #: #= value # employees" />
        </series-item>
    </series>

    <value-axis>
        <value-axis-item min="0">
            <labels format="{0} staff" />
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetHeadcountTrend", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

---

## 🔑 Controller — Time Series Data

```csharp
// Monthly revenue for the current year
public JsonResult GetMonthlyRevenue()
{
    var year = DateTime.Today.Year;

    // All 12 months — include months with 0 revenue
    var data = Enumerable.Range(1, 12).Select(month => {
        var revenue = _db.Orders
            .Where(o => o.OrderDate.Year == year &&
                        o.OrderDate.Month == month)
            .Sum(o => (decimal?)o.Total) ?? 0;

        return new {
            month   = new DateTime(year, month, 1).ToString("MMM"),
            revenue = Math.Round(revenue, 0)
        };
    }).ToList();

    return Json(data);
}
// Returns: [ {"month":"Jan","revenue":45000}, {"month":"Feb","revenue":52000}, ... ]


// Headcount trend by quarter
public JsonResult GetHeadcountTrend()
{
    var data = _db.Employees
        .Where(e => e.HireDate >= DateTime.Today.AddYears(-2))
        .GroupBy(e => new {
            Year    = e.HireDate.Year,
            Quarter = (e.HireDate.Month - 1) / 3 + 1
        })
        .OrderBy(g => g.Key.Year).ThenBy(g => g.Key.Quarter)
        .Select(g => new {
            quarter   = "Q" + g.Key.Quarter + " " + g.Key.Year,
            headcount = g.Count()
        })
        .ToList();

    return Json(data);
}
```

---

## 🔑 Multiple Line Series — Compare Trends

Show multiple metrics on the same time axis:

```cshtml
<kendo-chart name="trendCompareChart" height="400">

    <chart-title text="Revenue vs Target — Monthly" />
    <chart-legend position="ChartLegendPosition.Bottom" />

    <series>
        {{!-- Line 1: Actual Revenue --}}
        <series-item type="ChartSeriesType.Line"
                     field="actual"
                     category-field="month"
                     name="Actual Revenue"
                     color="#27ae60"
                     width="2">
            <markers visible="true" size="5" />
        </series-item>

        {{!-- Line 2: Target Revenue --}}
        <series-item type="ChartSeriesType.Line"
                     field="target"
                     category-field="month"
                     name="Target"
                     color="#e74c3c"
                     width="2"
                     dash-type="ChartDashType.Dash">  {{!-- dashed line for target --}}
            <markers visible="false" />
        </series-item>

        {{!-- Line 3: Last Year --}}
        <series-item type="ChartSeriesType.Line"
                     field="lastYear"
                     category-field="month"
                     name="Last Year"
                     color="#95a5a6"
                     width="1"
                     dash-type="ChartDashType.Dot">
        </series-item>
    </series>

    <value-axis>
        <value-axis-item min="0">
            <labels format="{0:C0}" />
        </value-axis-item>
    </value-axis>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetRevenueTrend", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

```csharp
public JsonResult GetRevenueTrend()
{
    var data = Enumerable.Range(1, 12).Select(m => new {
        month    = new DateTime(2024, m, 1).ToString("MMM"),
        actual   = GetMonthRevenue(2024, m),
        target   = GetMonthTarget(m),
        lastYear = GetMonthRevenue(2023, m)
    }).ToList();

    return Json(data);
}
```

```
Result:
$90k ┤         ___•---•           ← Actual (green solid)
$80k ┤  •---•-/ - - - -           ← Target (red dashed)
$70k ┤ ......•..............      ← Last Year (grey dotted)
     └──┬──┬──┬──┬──┬──┬──
        Jan Feb Mar Apr May Jun
```

---

## 🔑 Stacked Area Chart

Show how individual parts contribute to a total over time:

```cshtml
<series>
    <series-item type="ChartSeriesType.Area"
                 field="itRevenue"
                 category-field="month"
                 name="IT"
                 color="#3498db"
                 opacity="0.7"
                 stack="true">
    </series-item>
    <series-item type="ChartSeriesType.Area"
                 field="hrRevenue"
                 category-field="month"
                 name="HR"
                 color="#e74c3c"
                 opacity="0.7"
                 stack="true">
    </series-item>
    <series-item type="ChartSeriesType.Area"
                 field="salesRevenue"
                 category-field="month"
                 name="Sales"
                 color="#2ecc71"
                 opacity="0.7"
                 stack="true">
    </series-item>
</series>
```

```
Stacked result — each band = one department's revenue:
$200k ┤ ████████████ ← Sales (top)
$150k ┤ ░░░░░░░░░░░░ ← HR
$100k ┤ ████████████ ← IT (bottom)
       Jan Feb Mar Apr
Total height = combined revenue
```

---

## 🔑 Line Style Options

```cshtml
{{!-- Normal: straight lines between points (default) --}}
<series-item line-style="ChartLineStyle.Normal" />

{{!-- Smooth: curved bezier lines --}}
<series-item line-style="ChartLineStyle.Smooth" />

{{!-- Step: horizontal then vertical (like a staircase) --}}
<series-item line-style="ChartLineStyle.Step" />
```

```
Normal:    •——•——•——•
Smooth:    •~~•~~•~~•  (curved)
Step:      •__┐__┐__•  (staircase)
```

---

## 🔑 Markers — Data Point Dots

```cshtml
{{!-- Visible circle markers --}}
<markers visible="true"
         size="6"
         type="ChartMarkerType.Circle"
         background="#3498db"
         border-color="white"
         border-width="2" />

{{!-- Square markers --}}
<markers visible="true"
         type="ChartMarkerType.Square"
         size="7" />

{{!-- No markers (clean line) --}}
<markers visible="false" />
```

---

## 🔑 Date-Based Category Axis

When your X-axis is actual dates (not month name strings):

```cshtml
{{!-- Use a date category axis for proper date spacing --}}
<category-axis>
    <category-axis-item type="ChartCategoryAxisType.Date"
                        base-unit="months">
        <labels format="{0:MMM yyyy}"
                rotation="-45" />
        <major-grid-lines visible="false" />
    </category-axis-item>
</category-axis>
```

```csharp
// Controller returns actual DateTime values
public JsonResult GetDailyRevenue()
{
    var data = _db.Orders
        .GroupBy(o => o.OrderDate.Date)
        .Select(g => new {
            date    = g.Key,           // actual DateTime — not a string
            revenue = g.Sum(o => o.Total)
        })
        .OrderBy(x => x.date)
        .ToList();

    return Json(data);
}
```

---

## 🔑 Real-Time / Auto-Refresh Line Chart

Live chart that updates every N seconds:

```javascript
$(function() {
    var chart = $("#revenueChart").data("kendoChart");

    // Refresh every 30 seconds
    setInterval(function() {
        chart.dataSource.read();
    }, 30000);

    // Also refresh when user clicks a button
    $("#refreshBtn").on("click", function() {
        $(this).text("Refreshing...").prop("disabled", true);
        chart.dataSource.read().done(function() {
            $("#refreshBtn").text("Refresh").prop("disabled", false);
        });
    });

    // Show last-updated timestamp after each load
    chart.dataSource.bind("requestEnd", function() {
        var now = kendo.toString(new Date(), "MM/dd/yyyy hh:mm:ss tt");
        $("#lastUpdated").text("Last updated: " + now);
    });
});
```

---

## 🔑 Point Click — Navigate on Line Click

```javascript
chart.bind("seriesClick", function(e) {
    // e.category = the X-axis value clicked ("Jan", "Feb", etc.)
    // e.value    = the Y-axis value at that point
    // e.dataItem = the full data object for that point

    console.log("Point clicked:", e.category, "→", e.value);

    // Example: filter a grid to show only that month's orders
    var clickedMonth = e.dataItem.monthNumber;  // e.g. 3 for March
    var grid = $("#ordersGrid").data("kendoGrid");
    grid.dataSource.filter({
        field:    "OrderMonth",
        operator: "eq",
        value:    clickedMonth
    });
});
```

---

## 📊 Line vs Area Decision Guide

| Situation                      | Use                        |
| ------------------------------ | -------------------------- |
| Multiple series on same chart  | Line (less visual clutter) |
| Single trend to emphasise      | Area (filled, more impact) |
| Precise value comparison       | Line (cleaner)             |
| Show volume / magnitude        | Area                       |
| Stock / financial data         | Line                       |
| Stacked contributions to total | Stacked Area               |
| Show actual vs target          | Two Lines (solid + dashed) |

---

## ⚠️ Common Mistakes

| Mistake                                             | Symptom                                  | Fix                                                          |
| --------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| Too many line series (5+)                           | Chart looks like spaghetti               | Max 3-4 series per line chart                                |
| `opacity`not set on stacked Area                  | Areas overlap, lower series invisible    | Set `opacity="0.7"`on each Area series                     |
| No `min="0"`on value axis                         | Line appears flat for small differences  | Use `min="0"`unless showing negative values                |
| Missing `markers`causes confusion on short series | Hard to see individual data points       | Add `<markers visible="true" />`for series with < 8 points |
| Smooth curves on step data                          | Misleading curve between discrete values | Use `line-style="ChartLineStyle.Step"`for step data        |

---

## ❓ Interview Questions

**Q: What is the difference between a Line chart and an Area chart?**

> They are the same chart type — Line connects data points with a line. Area fills the space between the line and the X-axis. Area is more visually impactful for single series; Line is cleaner when multiple series overlap.

**Q: How do you show a dashed line for a target or forecast series?**

> Add `dash-type="ChartDashType.Dash"` (or `ChartDashType.Dot`) to the `<series-item>`. This visually distinguishes the target from actual values on the same chart.

**Q: What does `line-style="ChartLineStyle.Smooth"` do?**

> It draws bezier curves between data points instead of straight lines, giving a flowing, smooth appearance. Best for data that naturally changes gradually. Use `Normal` for precise financial data where straight lines accurately represent the values.

**Q: How do you use a stacked Area chart?**

> Add `stack="true"` to each `<series-item>`. Each area stacks on top of the previous one — the total height represents the combined value. Set `opacity` on each series so the filled areas remain visible underneath each other.
>
