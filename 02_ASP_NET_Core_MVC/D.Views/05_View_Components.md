
# 05 — View Components

---

## 🎯 One-Line Definition

> **A View Component is a self-contained, reusable widget that has its own C# logic class and Razor view — it fetches its OWN data from the database independently of the parent view, making it more powerful than a partial view for complex, reusable UI blocks.**

---

## 🔷 Partial View vs View Component — The Core Difference

```
PARTIAL VIEW:                           VIEW COMPONENT:
──────────────────────────────────      ──────────────────────────────────
Just a Razor template (.cshtml)         C# class  +  Razor template

Data comes FROM the parent view         Fetches its OWN data from DB/BAL
(parent passes the model down)          (independent of parent view)

No logic of its own                     Has InvokeAsync() method with logic

Simple HTML reuse                       Self-contained widget

Like a function call                    Like a mini-controller + view
```

```
WHEN TO USE WHICH:
──────────────────────────────────────────────────────────────
Parent view already has the data
and just wants to display it differently?   → Partial View

Widget needs to go to the DB itself
to get its own data
(notification count, sidebar menu, stats)?  → View Component
```

---

## 🔷 Real-World Examples of View Components

```
Things that appear on MANY pages but need their OWN data:
──────────────────────────────────────────────────────────────
🔔 Notification Bell       → needs count of unread notifications from DB
📋 Department Sidebar      → needs department list from DB
📊 Dashboard Stats Widget  → needs employee counts by department from DB
🧭 Navigation Menu         → needs menu items based on user's role from DB
🕐 Recent Activity Feed    → needs last 5 activities from DB
🏷️  Tag Cloud              → needs tags and frequencies from DB

All of these:
  ✅ Appear on multiple pages
  ✅ Need their own DB call
  ✅ Should not pollute every controller with "load this widget's data"
  ✅ Perfect for View Components
```

---

## 🔷 Structure — File Layout

```
Views/
└── Shared/
    └── Components/                         ← folder name is fixed
        └── DepartmentStats/                ← folder = component name
            └── Default.cshtml              ← view file (Default = default view name)

Components/ (or anywhere in project)
└── DepartmentStatsViewComponent.cs         ← the C# class
```

---

## 🔷 Building a View Component — Step by Step

### Step 1: The C# Class

```csharp
// Components/DepartmentStatsViewComponent.cs

using Microsoft.AspNetCore.Mvc;

// Class name MUST end with "ViewComponent"
public class DepartmentStatsViewComponent : ViewComponent
//                                          ↑ inherit from ViewComponent
{
    private readonly EmployeeBAL _bal;

    // Constructor injection — DI works exactly like controllers
    public DepartmentStatsViewComponent(EmployeeBAL bal)
    {
        _bal = bal;
    }

    // InvokeAsync — the entry point
    // Called when the view component is rendered
    // Returns: IViewComponentResult (like IActionResult for controllers)
    public async Task<IViewComponentResult> InvokeAsync(string filter = "all")
    {
        // This component fetches its OWN data — independent of parent view
        var stats = await _bal.GetDepartmentStatsAsync(filter);

        // Pass data to the Razor template:
        return View(stats);
        // Looks for: Views/Shared/Components/DepartmentStats/Default.cshtml
    }
}
```

### Step 2: The Razor Template

```html
<!-- Views/Shared/Components/DepartmentStats/Default.cshtml -->
@model List<DepartmentStat>

<div class="stats-widget">
    <h4>Department Statistics</h4>
    <ul>
        @foreach (var stat in Model)
        {
            <li>
                <strong>@stat.DepartmentName</strong>:
                @stat.EmployeeCount employees,
                Avg Salary: @stat.AvgSalary.ToString("C")
            </li>
        }
    </ul>
</div>
```

### Step 3: Use It in Any View

```html
<!-- In any .cshtml view — invoke the component -->

<!-- Tag Helper syntax (recommended): -->
<vc:department-stats></vc:department-stats>
<!-- ↑ DepartmentStatsViewComponent → <vc:department-stats> (kebab-case) -->

<!-- With a parameter: -->
<vc:department-stats filter="active"></vc:department-stats>

<!-- Programmatic syntax: -->
@await Component.InvokeAsync("DepartmentStats")
@await Component.InvokeAsync("DepartmentStats", new { filter = "active" })
```

---

## 🔷 Complete Real Example — Notification Bell

```
Scenario: Every page shows a notification bell icon with unread count.
The count comes from the database.
Without View Component: every single controller action would need to
  load notification count and add it to ViewBag — nightmare.
With View Component: self-contained, zero changes to other controllers.
```

```csharp
// Components/NotificationBellViewComponent.cs

public class NotificationBellViewComponent : ViewComponent
{
    private readonly NotificationBAL _notifBal;

    public NotificationBellViewComponent(NotificationBAL notifBal)
    {
        _notifBal = notifBal;
    }

    public async Task<IViewComponentResult> InvokeAsync()
    {
        // Get current user's ID from HttpContext (User claims)
        var userId = HttpContext.User
            .FindFirstValue(ClaimTypes.NameIdentifier);

        if (userId == null)
            return View(0);   // not logged in — show 0

        int unreadCount = await _notifBal.GetUnreadCountAsync(int.Parse(userId));

        return View(unreadCount);
        // Passes int to the Razor template
    }
}
```

```html
<!-- Views/Shared/Components/NotificationBell/Default.cshtml -->
@model int

<div class="notification-bell">
    <span class="bell-icon">🔔</span>
    @if (Model > 0)
    {
        <span class="badge badge-danger">@Model</span>
    }
</div>
```

```html
<!-- Views/Shared/_Layout.cshtml — used on every page -->
<nav>
    <span>Employee Portal</span>

    <!-- Notification bell appears on every page automatically -->
    <!-- It fetches its own data — no controller involved -->
    <vc:notification-bell></vc:notification-bell>
</nav>
```

---

## 🔷 View Component With Parameters

```csharp
// Component that accepts parameters to customize its behavior:

public class RecentActivityViewComponent : ViewComponent
{
    private readonly ActivityBAL _actBal;
    public RecentActivityViewComponent(ActivityBAL actBal)
        => _actBal = actBal;

    // InvokeAsync accepts parameters — passed from the view
    public async Task<IViewComponentResult> InvokeAsync(
        int    count  = 5,          // how many items to show
        string type   = "all",      // filter type
        bool   compact = false)     // compact or full display
    {
        var activities = await _actBal.GetRecentAsync(count, type);

        // Pass extra data to view via ViewData:
        ViewData["Compact"] = compact;

        return View(activities);
    }
}
```

```html
<!-- Call with parameters: -->
<vc:recent-activity count="10" type="login" compact="true"></vc:recent-activity>

<!-- Programmatic with anonymous object: -->
@await Component.InvokeAsync("RecentActivity",
    new { count = 10, type = "login", compact = true })
```

---

## 🔷 Multiple Views for One Component

```csharp
// A component can have multiple Razor templates — choose at runtime:

public async Task<IViewComponentResult> InvokeAsync(bool minimal = false)
{
    var stats = await _bal.GetStatsAsync();

    if (minimal)
        return View("Minimal", stats);
    // → Views/Shared/Components/DepartmentStats/Minimal.cshtml

    return View(stats);
    // → Views/Shared/Components/DepartmentStats/Default.cshtml
}
```

```
Views/Shared/Components/DepartmentStats/
├── Default.cshtml    ← return View(model)
└── Minimal.cshtml    ← return View("Minimal", model)
```

---

## 🔷 Registering for Tag Helper Syntax

```html
<!-- For <vc:component-name> to work, add this to _ViewImports.cshtml: -->
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<!-- Also add your assembly for custom components: -->
@addTagHelper *, EmployeeManagement
<!--           ↑ your project/assembly name -->
```

---

## 🔷 View Component vs Controller — Key Difference

```csharp
// Controller action — responds to a URL:
// GET /Employee/GetStats → EmployeeController.GetStats() → View or JSON

// View Component — embedded in a Razor view:
// <vc:department-stats /> → DepartmentStatsViewComponent.InvokeAsync()
//                        → renders HTML fragment inline

// View Component is NOT accessible via a URL directly
// It's always invoked FROM a view
```

---

## 🔷 View Component vs Partial View — Decision Summary

```
Question to ask yourself:
──────────────────────────────────────────────────────────────

"Where does the data come from?"

Data is ALREADY IN the parent view's model
  → Use Partial View
  → Parent passes model down: <partial name="_Card" model="emp" />
  → No extra DB call

Data comes from the DATABASE independently
  → Use View Component
  → Component fetches its own: <vc:department-stats />
  → Each page that uses it gets fresh data automatically
```

---

## ⭐ Interview Quick-Fire

| Question                                                                             | Answer                                                                                                                                     |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| What is a View Component?                                                            | A self-contained C# class + Razor template that fetches its own data — a reusable widget independent of the parent view                   |
| How is a View Component different from a Partial View?                               | Partial = HTML reuse, data from parent. View Component = has its own C# logic and its own data from DB                                     |
| What must a View Component class inherit from?                                       | `ViewComponent`                                                                                                                          |
| What is the entry method of a View Component?                                        | `InvokeAsync()`— returns `IViewComponentResult`                                                                                       |
| Where does the Razor template for a View Component live?                             | `Views/Shared/Components/{ComponentName}/Default.cshtml`                                                                                 |
| What is the Tag Helper syntax to invoke a View Component named `NotificationBell`? | `<vc:notification-bell></vc:notification-bell>`(class name in kebab-case)                                                                |
| Does DI work in View Components?                                                     | ✅ Yes — constructor injection works exactly like controllers                                                                             |
| What is a good use case for a View Component?                                        | Notification bell count, sidebar stats, navigation menu based on role — anything that needs its own DB call and appears on multiple pages |
| Can a View Component return different views?                                         | ✅ Yes —`return View("Minimal", model)`returns a named template instead of Default                                                      |
