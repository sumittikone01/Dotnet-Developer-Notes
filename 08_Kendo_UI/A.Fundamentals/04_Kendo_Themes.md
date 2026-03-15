
# 04 — Kendo Themes

---

## 🎯 One-Line Definition

> **A Kendo theme is one CSS file that controls the look of every single Kendo widget in your app — colours, borders, fonts, button styles — swap the file, the whole app re-skins.**

---

## 🎨 What a Theme Controls

Without a theme CSS file, Kendo widgets are completely unstyled — raw HTML with no visual design.

```
No theme:                          With Default theme:
──────────────────────────────     ──────────────────────────────
┌─────────────────────────┐        ┌─────────────────────────────┐
│ id name   dept          │        │ ▼ ID  Name     Department   │
│ 1  alice  it            │        ├─────┼────────┼─────────────┤
│ 2  bob    hr            │        │  1  │ Alice  │ IT          │
│ [edit][del]             │        │  2  │ Bob    │ HR          │
│ < 1 2 3 >               │        │ [✎ Edit] [🗑 Delete]       │
└─────────────────────────┘        │ ◄ 1  2  3  ►               │
  (raw, no styling)                └─────────────────────────────┘
                                     (styled, professional)
```

A theme CSS file styles:

```
Every widget in one CSS file:
─────────────────────────────────────────────────────────────
Grid          → table borders, header background, row hover,
                alternating row colour, button styles
DatePicker    → calendar popup, selected date highlight,
                navigation arrows
DropDownList  → dropdown arrow, item hover, selected item
TextBox       → border, focus ring, placeholder colour
Button        → background, hover state, active state
Window        → title bar, border, shadow, close button
Chart         → axis colours, series colours, tooltip
Notification  → success green, error red, warning yellow
...every other widget...
```

---

## 🎨 Built-in Themes — What Kendo Ships With

| Theme Name             | Character                      | Best For                          |
| ---------------------- | ------------------------------ | --------------------------------- |
| **Default**      | Clean blue-grey professional   | Most business apps                |
| **Default Dark** | Same but dark background       | Dark mode preference              |
| **Bootstrap**    | Matches Bootstrap 5 components | Apps already using Bootstrap      |
| **Material**     | Google Material Design         | Modern clean look                 |
| **Fluent**       | Microsoft Office / Teams style | Enterprise Microsoft-aligned apps |
| **Nova**         | Sleek modern, dark-friendly    | Modern dashboards                 |
| **Ocean Blue**   | Blue-toned professional        | Finance/data-heavy apps           |
| **Classic**      | Old Kendo look                 | Legacy apps staying consistent    |

---

## 🔗 How to Apply a Theme — Just One Line

The entire theme is one `<link>` tag in your `<head>`:

```html
<!-- Default theme -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css" />

<!-- Bootstrap theme -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/bootstrap/bootstrap-main.css" />

<!-- Material theme -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/material/material-main.css" />

<!-- Fluent theme -->
<link rel="stylesheet"
      href="https://kendo.cdn.telerik.com/themes/6.3.0/fluent/fluent-main.css" />
```

**To switch themes: change the `href` of that one CSS link. Nothing else changes.**

---

## 🔍 Understanding the Theme CSS URL

```
https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css
                                      ↑       ↑       ↑
                                  Version  Theme   File name
                                   6.3.0   name

Breaking it down:
  /themes/6.3.0/     → Kendo Themes package version
  /default/          → theme folder name
  /default-main.css  → the main theme file
```

---

## 🌙 Light and Dark Variants

Most themes ship with both a light and dark version:

```html
<!-- Light version (default) -->
<link rel="stylesheet"
      href=".../themes/6.3.0/default/default-main.css" />

<!-- Dark version -->
<link rel="stylesheet"
      href=".../themes/6.3.0/default/default-dark.css" />

<!-- Ocean Blue Light -->
<link rel="stylesheet"
      href=".../themes/6.3.0/default/default-ocean-blue.css" />

<!-- Ocean Blue Dark -->
<link rel="stylesheet"
      href=".../themes/6.3.0/default/default-ocean-blue-dark.css" />
```

---

## 🎨 Customizing a Theme — SASS Variables

If the built-in themes don't quite match your brand, Kendo themes are built with **SASS variables** that you can override.

Every visual property is a variable:

```scss
// Override Kendo theme variables in your own .scss file
// Import the theme first, then override

$kendo-color-primary:          #c0392b;   // your brand red (buttons, highlights)
$kendo-color-primary-hover:    #e74c3c;   // hover state of primary colour
$kendo-color-base:             #ffffff;   // widget background
$kendo-color-border:           #dddddd;   // borders around widgets
$kendo-font-family:            "Segoe UI", sans-serif;
$kendo-font-size:              14px;
$kendo-border-radius:          4px;       // corner roundness

@import "~@progress/kendo-theme-default/dist/all.scss";
```

> For most business projects you won't need SASS customization — pick the closest built-in theme and move on. Customization is for when your app needs to match a strict brand guideline.

---

## 📋 Theme in _Layout.cshtml — Full Correct Placement

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"]</title>

    <!-- ✅ Theme CSS goes in <head> — NEVER in body -->
    <link rel="stylesheet"
          href="https://kendo.cdn.telerik.com/themes/6.3.0/default/default-main.css" />

    <!-- Your own CSS comes AFTER the theme -->
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    @RenderBody()

    <!-- Scripts at the bottom of body -->
    <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.all.min.js"></script>
    <script src="https://kendo.cdn.telerik.com/2023.3.1114/js/kendo.aspnetmvc.min.js"></script>
    @RenderSection("Scripts", required: false)
</body>
</html>
```

---

## 🔧 Overriding Specific Widget Styles

Sometimes you want to tweak one widget without building a custom theme. Use your own CSS and target Kendo's CSS classes — but load your CSS **after** the theme:

```css
/* In your site.css — loaded after the theme CSS */

/* Make all Grid headers dark blue */
.k-grid-header {
    background-color: #2c3e50 !important;
    color: white !important;
}

/* Make alternating Grid rows light grey */
.k-grid tbody tr:nth-child(even) {
    background-color: #f8f9fa;
}

/* Make the primary buttons your brand colour */
.k-button-solid-primary {
    background-color: #e74c3c;
    border-color: #c0392b;
}

/* Increase Grid row height */
.k-grid td {
    padding: 12px 8px;
}
```

```
Rule: your overrides must load AFTER the Kendo theme CSS
      so your rules take priority over the theme rules.

_Layout.cshtml order:
1. <link> kendo theme CSS     ← theme sets defaults
2. <link> your site.css       ← your rules override theme
```

---

## ⚠️ Common Mistakes

| Mistake                                             | Symptom                                                  | Fix                                                 |
| --------------------------------------------------- | -------------------------------------------------------- | --------------------------------------------------- |
| Theme CSS not included                              | Widgets render with no styling — raw HTML look          | Add the theme `<link>`in `<head>`               |
| Theme CSS in `<body>`instead of `<head>`        | Flash of unstyled widgets when page loads                | Move `<link>`into `<head>`                      |
| Your `site.css`loads before Kendo theme           | Your overrides get overwritten by the theme              | Load your CSS after the Kendo theme `<link>`      |
| CSS and JS version mismatch                         | Widgets styled incorrectly — off colours, wrong borders | Use the matching theme+JS version from Telerik docs |
| Using `.k-`class overrides without `!important` | Theme styles win, your override doesn't apply            | Add `!important`to your override rules            |

---

## ❓ Interview Questions

**Q: What is a Kendo theme?**

> A single CSS file that controls the visual appearance of every Kendo widget — colours, borders, fonts, button states. Swapping the CSS file changes the look of the entire application.

**Q: How do you switch from the Default theme to the Bootstrap theme?**

> Change the `href` of the theme `<link>` tag from the default CSS path to the Bootstrap CSS path. No JavaScript or C# changes are needed.

**Q: Where must the theme CSS link be placed?**

> In the `<head>` tag, before your own CSS files. Placing it in the body causes a flash of unstyled widgets as the page loads.

**Q: How do you override the style of just one widget without building a custom theme?**

> Add your own CSS rules targeting Kendo's `.k-` classes in your `site.css`, loaded after the Kendo theme. Use `!important` if needed to ensure your rules take priority over the theme.

**Q: What are SASS variables used for in Kendo theming?**

> They let you customise the entire theme — primary colour, fonts, border radius, backgrounds — by overriding Kendo's predefined SASS variables before compiling. Used when the app needs to match a specific brand identity.
>
