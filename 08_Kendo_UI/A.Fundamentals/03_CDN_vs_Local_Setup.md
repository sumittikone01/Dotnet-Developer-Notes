
# 03 — CDN vs Local Setup

---

## 🎯 One-Line Definition

> **CDN loads Kendo files from Telerik's internet servers. Local setup downloads those same files into your project. Same result — different source.**

---

## 🤔 What Are We Even Choosing Between?

Kendo needs 4 files to work — a CSS theme file and 3 JavaScript files.

```
Files Kendo needs in every page:
──────────────────────────────────────────────────────────────
1. kendo theme CSS         → styles all widgets
2. jQuery JS               → Kendo depends on this
3. kendo.all.min.js        → all widget logic
4. kendo.aspnetmvc.min.js  → ASP.NET MVC integration
```

The question is: **where do those files come from?**

```
Option A — CDN:                    Option B — Local:
────────────────────────────────   ────────────────────────────────
Files live on Telerik's servers    Files live in YOUR project
Browser downloads them from        Browser downloads them from
kendo.cdn.telerik.com              your own server (wwwroot/lib/)
every time your page loads         when your page loads
```

---

## 🌐 Option A — CDN Setup

CDN = Content Delivery Network. Telerik hosts the files on fast global servers and you just link to them.

### What the links look like

```html
<!-- In _Layout.cshtml -->
<head>
    <!-- 1. Kendo CSS Theme from CDN -->
    <link rel="stylesheet"
          href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css" />
</head>

<body>
    @RenderBody()

    <!-- 2. jQuery from CDN (must be first) -->
    <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>

    <!-- 3. Kendo All JS from CDN -->
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.all.min.js"></script>

    <!-- 4. Kendo ASP.NET MVC JS from CDN -->
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.aspnetmvc.min.js"></script>
</body>
```

### ✅ Pros of CDN

```
✅  Zero setup — just paste the links, works immediately
✅  No files to store in your project
✅  Telerik's CDN is fast — global edge servers
✅  Browser caching — if user visited another Kendo site,
    the files are already cached locally, page loads faster
✅  Always available when you need to demo or prototype quickly
```

### ❌ Cons of CDN

```
❌  Requires internet connection — works only when online
❌  No internet on client's server = site breaks
❌  Telerik CDN going down = your app goes down
❌  Version changes must be managed carefully
❌  Firewall / network restrictions can block CDN
❌  Not suitable for intranet/offline applications
```

---

## 💾 Option B — Local Setup

Download Kendo files once, put them in your project, serve them from your own server.

### Step 1 — Install via NuGet

```bash
# In Package Manager Console
Install-Package Telerik.UI.for.AspNet.Core

# Or via CLI
dotnet add package Telerik.UI.for.AspNet.Core
```

After installing, Kendo files land in your project under `wwwroot/lib/`.

### Step 2 — What files are now in your project

```
wwwroot/
└── lib/
    └── kendo/
        ├── css/
        │   ├── default/
        │   │   └── default-main.css      ← theme CSS
        │   └── ...other themes...
        └── js/
            ├── jquery.min.js             ← jQuery
            ├── kendo.all.min.js          ← all Kendo widgets
            └── kendo.aspnetmvc.min.js    ← MVC integration
```

### Step 3 — Link to local files in _Layout.cshtml

```html
<head>
    <!-- CSS from your own project -->
    <link rel="stylesheet" href="~/lib/kendo/css/default/default-main.css" />
</head>

<body>
    @RenderBody()

    <!-- JS from your own project (~ = wwwroot) -->
    <script src="~/lib/kendo/js/jquery.min.js"></script>
    <script src="~/lib/kendo/js/kendo.all.min.js"></script>
    <script src="~/lib/kendo/js/kendo.aspnetmvc.min.js"></script>
</body>
```

### ✅ Pros of Local

```
✅  Works offline — no internet dependency
✅  Works on intranets and isolated servers
✅  You control the version — upgrades are deliberate
✅  Not affected by CDN outages
✅  Faster for internal/LAN deployments
✅  Passes strict security audits (no external requests)
```

### ❌ Cons of Local

```
❌  Files add size to your project / repository
❌  You manage updates yourself (download new version, replace files)
❌  No automatic browser caching benefit across other sites
```

---

## ⚖️ CDN vs Local — Decision Table

| Situation                                | Use                    |
| ---------------------------------------- | ---------------------- |
| Building an intranet app (no internet)   | **Local**        |
| Client's server has no internet access   | **Local**        |
| Security policy blocks external requests | **Local**        |
| Quick prototype or demo                  | **CDN**          |
| Public-facing internet app               | **Either**       |
| First learning / getting started         | **CDN**(simpler) |
| Production enterprise app                | **Local**(safer) |

> **In most company/workplace projects: use Local.** You don't want your app depending on an external server you don't control.

---

## ⚠️ Version Matching — The Most Common Mistake

Whether CDN or Local, the  **CSS version and JS version must match exactly** .

```
❌ MISMATCH — broken widgets, visual glitches:
────────────────────────────────────────────────────────────
CSS:  kendo.cdn.telerik.com/themes/6.3.0/...    ← version 6.3.0
JS:   kendo.cdn.telerik.com/2023.1.117/js/...   ← version 2023.1

These are different release versions — styles won't match the widgets


✅ MATCH — everything works:
────────────────────────────────────────────────────────────
CSS:  kendo.cdn.telerik.com/themes/6.3.0/...    ← 6.3.0
JS:   kendo.cdn.telerik.com/2023.3.1114/js/...  ← 2023.3.1114

These are the correct pair for this release
(Telerik's docs always show which theme version pairs with which JS version)
```

---

## ⚙️ Program.cs — Required Either Way

Regardless of CDN or Local, this goes in `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddJsonOptions(options => {
        options.JsonSerializerOptions.PropertyNamingPolicy =
            System.Text.Json.JsonNamingPolicy.CamelCase;
    });

// Required — registers Kendo server-side helpers
builder.Services.AddKendo();

var app = builder.Build();
app.UseStaticFiles();   // required to serve files from wwwroot
```

## ⚙️ _ViewImports.cshtml — Required Either Way

```cshtml
@using YourProjectName
@using YourProjectName.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@* This line enables all Kendo Tag Helpers *@
@addTagHelper *, Telerik.UI.for.AspNet.Core
```

---

## 📋 Complete _Layout.cshtml — Both Versions Side by Side

```html
<!-- ── CDN VERSION ──────────────────────────────────────────── -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet"
          href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css" />
</head>
<body>
    @RenderBody()
    <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.all.min.js"></script>
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.aspnetmvc.min.js"></script>
    @RenderSection("Scripts", required: false)
</body>
</html>


<!-- ── LOCAL VERSION ────────────────────────────────────────── -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"]</title>
    <link rel="stylesheet" href="~/lib/kendo/css/default/default-main.css" />
</head>
<body>
    @RenderBody()
    <script src="~/lib/kendo/js/jquery.min.js"></script>
    <script src="~/lib/kendo/js/kendo.all.min.js"></script>
    <script src="~/lib/kendo/js/kendo.aspnetmvc.min.js"></script>
    @RenderSection("Scripts", required: false)
</body>
</html>
```

---

## ⚠️ Common Mistakes and Fixes

| Mistake                                      | Symptom                                                   | Fix                                                 |
| -------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------- |
| jQuery after Kendo JS                        | `$ is not defined`error in console, nothing renders     | Move jQuery script tag above Kendo                  |
| Version mismatch between CSS and JS          | Widgets render but look broken, wrong borders/colors      | Use the matching version pair from Telerik docs     |
| Missing `@addTagHelper`in `_ViewImports` | Kendo tag helpers not recognised, treated as plain HTML   | Add `@addTagHelper *, Telerik.UI.for.AspNet.Core` |
| Missing `app.UseStaticFiles()`             | Local files return 404                                    | Add `app.UseStaticFiles()`in Program.cs           |
| Missing `builder.Services.AddKendo()`      | `ToDataSourceResult()`not available, Tag Helpers broken | Add `AddKendo()`in Program.cs                     |
| CSS in body, not in head                     | Flash of unstyled widgets on load                         | Keep CSS link in `<head>`always                   |

---

## ❓ Interview Questions

**Q: What is the difference between CDN and local setup for Kendo?**

> CDN loads Kendo files from Telerik's internet servers — no files in your project, works immediately, requires internet. Local setup downloads the files into your project's `wwwroot` — works offline, no external dependency, better for enterprise/intranet apps.

**Q: Which approach is better for a company intranet application?**

> Local — the server may not have internet access, and you don't want the app to break because an external CDN is unreachable.

**Q: What order must the script tags appear in?**

> jQuery first, then `kendo.all.min.js`, then `kendo.aspnetmvc.min.js`. Kendo depends on jQuery — wrong order causes "$ is not defined" errors.

**Q: What happens if CSS and JS versions don't match?**

> Widgets render but look visually broken — wrong colours, borders out of place, icons missing — because the CSS classes don't match the HTML the JS is generating.
>
