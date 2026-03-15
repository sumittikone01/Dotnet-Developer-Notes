
# 01 — Kendo Charts Overview

---

## 🎯 One-Line Definition

> **Kendo Charts turn your controller's JSON data into interactive, animated visualizations — bar, line, pie, area, and more — all connected to your ASP.NET backend via AJAX DataSource.**

---

## 🔑 What Kendo Charts Give You

```
Raw data in your database:              Interactive chart on the page:
────────────────────────────────        ──────────────────────────────────────
Department  | AvgSalary | Count         $90k ┤          ████
─────────────────────────────           $80k ┤    ████  ████  ████
IT          |   75,000  |  45           $70k ┤    ████  ████  ████  ████
HR          |   55,000  |  20     ──►   $60k ┤    ████  ████  ████  ████
Finance     |   65,000  |  30           $50k ┤    ████  ████  ████  ████
Sales       |   48,000  |  25                └────┬─────┬─────┬─────┬───
                                                  IT    HR   Fin  Sales

                                        Hover → tooltip shows exact value
                                        Click legend → hide/show a series
                                        Export → download as PNG/SVG
```

---

## 🔑 Chart Types Available

```
┌───────────────────────────────────────────────────────────────┐
│  COMPARISON                                                   │
│    Bar         → horizontal bars, compare categories         │
│    Column      → vertical bars (same as bar, rotated)        │
│    Bullet      → progress vs target                          │
│                                                               │
│  TREND OVER TIME                                              │
│    Line        → connected points, trends over time          │
│    Area        → filled line, emphasizes volume              │
│    StockChart  → financial OHLC candlestick                  │
│                                                               │
│  PROPORTION                                                   │
│    Pie         → parts of a whole (use < 7 slices)           │
│    Donut       → pie with a hole in the middle               │
│    Funnel      → conversion stages                           │
│                                                               │
│  DISTRIBUTION                                                 │
│    Scatter     → two variables, find correlation             │
│    Bubble      → three variables                             │
│    BoxPlot     → statistical distribution                    │
│                                                               │
│  SPECIALIZED                                                  │
│    Radar       → spider/web chart, multi-axis comparison     │
│    Sparkline   → tiny inline chart (no axes, no title)       │
└───────────────────────────────────────────────────────────────┘
```

---

## 🔑 Anatomy of a Kendo Chart

```
┌──────────────────────────────────────────────────────────────┐
│  Chart Title                                                 │  ← chartTitle
│                                                              │
│  $90k ┤                                                      │
│  $80k ┤    ████                                              │  ← valueAxis
│  $70k ┤    ████  ████                                        │
│  $60k ┤    ████  ████  ████                                  │
│        └────┬─────┬─────┬────                                │
│             IT    HR   Fin                                   │  ← categoryAxis
│                                                              │
│   ■ Avg Salary    ■ Headcount                                │  ← legend
│                        © Export  🔍 Zoom                    │  ← navigator
└──────────────────────────────────────────────────────────────┘

Each part maps to a Tag Helper section:
  chartTitle   → <chart-title text="..." />
  series       → <series><series-item .../></series>
  categoryAxis → <category-axis>
  valueAxis    → <value-axis>
  legend       → <chart-legend>
  tooltip      → tooltips on hover
  DataSource   → <chart-data-source>
```

---

## 🔑 Minimum Working Chart — Tag Helper

```cshtml
<kendo-chart name="salaryChart">

    <chart-title text="Average Salary by Department" />

    <series>
        <series-item type="ChartSeriesType.Column"
                     field="avgSalary"
                     category-field="department"
                     name="Avg Salary">
        </series-item>
    </series>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetSalaryData", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

```csharp
// Controller — returns JSON array
public JsonResult GetSalaryData()
{
    var data = _db.Employees
        .GroupBy(e => e.Department)
        .Select(g => new {
            department = g.Key,
            avgSalary  = Math.Round(g.Average(e => e.Salary), 0),
            count      = g.Count()
        })
        .OrderBy(x => x.department)
        .ToList();

    return Json(data);
}
// Returns: [ {"department":"Finance","avgSalary":65000,"count":30}, ... ]
```

---

## 🔑 DataSource — How Charts Get Their Data

Same DataSource concept as the Grid, but charts mostly use read-only:

```cshtml
{{!-- Remote AJAX (most common) --}}
<chart-data-source type="DataSourceTagHelperType.Ajax">
    <transport>
        <read url="@Url.Action("GetData", "Chart")" />
    </transport>
</chart-data-source>


{{!-- Local inline data (small static data) --}}
<chart-data-source type="DataSourceTagHelperType.Custom">
    <schema>
        <model>
            <fields>
                <field name="month"  type="string" />
                <field name="sales"  type="number" />
            </fields>
        </model>
    </schema>
</chart-data-source>
{{!-- Bind local data via JavaScript: chart.dataSource.data([...]) --}}
```

---

## 🔑 JavaScript API — Controlling Charts

```javascript
// Get reference
var chart = $("#salaryChart").data("kendoChart");

// ── Refresh data from server ──────────────────────────────────
chart.dataSource.read()

// ── Redraw the chart (re-render with current data) ────────────
chart.redraw()

// ── Resize (call when container size changes) ─────────────────
chart.resize()

// ── Export ────────────────────────────────────────────────────
chart.exportImage()   // returns a Promise with base64 PNG
chart.exportSVG()     // returns a Promise with SVG string
chart.exportPDF()     // triggers PDF download

// ── Save as image ─────────────────────────────────────────────
kendo.drawing.drawDOM($("#salaryChart"), { paperSize: "A4" })
    .then(function(group) {
        kendo.drawing.pdf.saveAs(group, "chart.pdf");
    });
```

---

## 🔑 Common Chart Configuration — Applies to All Types

```cshtml
<kendo-chart name="myChart" height="350">

    {{!-- Title --}}
    <chart-title text="Chart Title"
                 font="16px Arial"
                 color="#2c3e50" />

    {{!-- Legend --}}
    <chart-legend position="ChartLegendPosition.Bottom"
                  visible="true" />

    {{!-- Tooltip on hover --}}
    <tooltip visible="true"
             format="{0:C0}"
             template="#= series.name #: #= kendo.format('{0:C0}', value) #" />

    {{!-- Theme colours --}}
    {{!-- Kendo uses its theme colours by default. --}}
    {{!-- Override per-series with color="..." --}}

    <series>
        <series-item type="ChartSeriesType.Column"
                     field="value"
                     category-field="label"
                     color="#3498db"
                     name="Series Name">
        </series-item>
    </series>

    <chart-data-source type="DataSourceTagHelperType.Ajax">
        <transport>
            <read url="@Url.Action("GetData", "Chart")" />
        </transport>
    </chart-data-source>

</kendo-chart>
```

---

## 🔑 Refreshing a Chart — Auto-Refresh Pattern

```javascript
// Refresh chart data when something changes
function refreshChart() {
    $("#salaryChart").data("kendoChart").dataSource.read();
}

// Auto-refresh every 60 seconds
setInterval(function() {
    refreshChart();
}, 60000);

// Refresh chart when grid saves
var grid = $("#employeeGrid").data("kendoGrid");
grid.dataSource.bind("requestEnd", function(e) {
    if (e.type === "create" || e.type === "update" || e.type === "destroy") {
        refreshChart();   // keep chart in sync with grid changes
    }
});
```

---

## 🔑 Chart Series Types Quick Reference

```cshtml
{{!-- Column (vertical bars) --}}
<series-item type="ChartSeriesType.Column" />

{{!-- Bar (horizontal bars) --}}
<series-item type="ChartSeriesType.Bar" />

{{!-- Line --}}
<series-item type="ChartSeriesType.Line" />

{{!-- Area (filled line) --}}
<series-item type="ChartSeriesType.Area" />

{{!-- Pie --}}
<series-item type="ChartSeriesType.Pie" />

{{!-- Donut --}}
<series-item type="ChartSeriesType.Donut" />

{{!-- Scatter --}}
<series-item type="ChartSeriesType.Scatter" />
```

---

## ⚠️ Common Chart Mistakes

| Mistake                                   | Symptom                             | Fix                                                         |
| ----------------------------------------- | ----------------------------------- | ----------------------------------------------------------- |
| `field`name doesn't match JSON property | Chart renders empty (no data shown) | Match exactly:`field="avgSalary"`↔ JSON `"avgSalary"`  |
| `category-field`not set                 | All bars stack on one category      | Set `category-field="department"`                         |
| Chart not refreshing after grid save      | Chart shows stale data              | Call `chart.dataSource.read()`after grid's `requestEnd` |
| Chart invisible in hidden tab             | Renders at 0x0 size                 | Call `chart.resize()`when the tab becomes visible         |

---

## ❓ Interview Questions

**Q: How does a Kendo Chart get its data?**

> Via a DataSource with a transport read URL. The chart makes an AJAX call to a controller action that returns a JSON array. Each object in the array is one data point. The `field` attribute on the series maps to the JSON property name.

**Q: How do you refresh a chart when the underlying data changes?**

> Call `chart.dataSource.read()` to re-fetch data from the server. Pair this with the grid's `requestEnd` event so the chart stays in sync after saves.

**Q: What is the difference between `field` and `category-field` in a series?**

> `field` is the numeric value plotted on the value axis (e.g., salary amount). `category-field` is the label shown on the category axis (e.g., department name). Without `category-field`, all bars collapse into one category.
>
