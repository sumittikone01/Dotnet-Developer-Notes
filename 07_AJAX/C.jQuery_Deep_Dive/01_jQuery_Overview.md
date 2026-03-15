
# 01 — jQuery Overview

---

## 🎯 One-Line Definition

> **jQuery is a JavaScript library that makes selecting HTML elements, changing them, handling events, and making AJAX calls much shorter and simpler to write.**

---

## 🤔 Why jQuery Exists — The Problem It Solved

Writing plain JavaScript in 2006 (when jQuery was created) was painful:

```javascript
// ── Vanilla JS (old, without jQuery) ─────────────────────────

// Select all elements with class "btn" and hide them
var buttons = document.getElementsByClassName("btn");
for (var i = 0; i < buttons.length; i++) {
    buttons[i].style.display = "none";
}

// Make an AJAX request
var xhr = new XMLHttpRequest();
xhr.open("GET", "/api/data");
xhr.onload = function() { ... };
xhr.send();


// ── Same thing with jQuery ────────────────────────────────────

// Select and hide
$(".btn").hide();

// AJAX request
$.get("/api/data", function(data) { ... });
```

One line instead of six. That's why jQuery became the most used JavaScript library in history.

---

## 🌍 jQuery in ASP.NET MVC Projects — Why You Must Know It

```
jQuery is everywhere in your stack:

  ✅ Kendo UI is BUILT ON jQuery — every widget depends on it
  ✅ ASP.NET MVC uses jQuery for unobtrusive validation
  ✅ $.ajax / $.get / $.post = how you do AJAX in MVC projects
  ✅ Every Kendo AJAX call goes through jQuery internally
  ✅ Kendo grid reference: $("#grid").data("kendoGrid") ← jQuery
```

You cannot work with Kendo UI without understanding jQuery.

---

## 🔑 The `$` Symbol — What It Actually Is

`$` is just a  **shortcut name for the `jQuery` function** . They are identical.

```javascript
// These do exactly the same thing:
jQuery("#myDiv").hide();
$("#myDiv").hide();      // $ is just an alias for jQuery

// $ is a function — you call it like a function
$("p")           // selects all <p> elements
$("#header")     // selects element with id="header"
$(".card")       // selects all elements with class="card"
$(document)      // wraps the document object
$(this)          // wraps the current element in an event handler
```

---

## 🔑 What jQuery Returns — The jQuery Object

When you call `$()`, you get back a **jQuery object** — a special wrapper that:

* Holds a collection of matched HTML elements
* Has jQuery methods attached (`.hide()`, `.css()`, `.on()`, etc.)

```javascript
var result = $("p");
// result is a jQuery object containing ALL <p> elements on the page

result.length    // how many elements matched
result.hide()    // hides ALL matched elements at once
result.css("color", "red")   // turns ALL of them red

// Contrast with vanilla JS:
document.querySelectorAll("p")  // returns a NodeList (not a jQuery object)
// NodeList has no .hide(), no .css() — you'd loop manually
```

---

## 🔑 Chaining — jQuery's Superpower

Every jQuery method returns the jQuery object — so you can chain methods:

```javascript
// Without chaining — three separate statements
$("#myDiv").addClass("highlight");
$("#myDiv").css("font-size", "18px");
$("#myDiv").show();

// With chaining — one clean statement
$("#myDiv")
    .addClass("highlight")
    .css("font-size", "18px")
    .show();

// Both do exactly the same thing
// Chaining is faster and more readable
```

---

## 🔑 Document Ready — When to Run Your Code

Your script must not run before the HTML is loaded. Wrap it in document ready:

```javascript
// ── Modern way (jQuery 3+) — recommended ─────────────────────
$(function() {
    // All your jQuery code goes here
    // Runs when DOM is fully loaded (not waiting for images)
    $("#saveBtn").on("click", saveEmployee);
    loadEmployees();
});

// ── Old way — same result, more typing ───────────────────────
$(document).ready(function() {
    // same thing
});

// ── Without document ready — WRONG ───────────────────────────
// This runs immediately — before HTML elements exist
$("#saveBtn").on("click", saveEmployee);
// Error: #saveBtn not found (HTML hasn't rendered yet)
```

---

## 🔑 Including jQuery

```html
<!-- Option 1: CDN (internet required) -->
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>

<!-- Option 2: Local file from wwwroot -->
<script src="~/lib/jquery/dist/jquery.min.js"></script>
```

```
Load order rule (same as Kendo):
─────────────────────────────────────────────
1. jQuery        ← always first
2. kendo.all.js  ← depends on jQuery
3. your code     ← depends on both
```

---

## 📊 jQuery vs Vanilla JS — Quick Comparison

| Task            | Vanilla JS                               | jQuery                    |
| --------------- | ---------------------------------------- | ------------------------- |
| Select by ID    | `document.getElementById("x")`         | `$("#x")`               |
| Select by class | `document.getElementsByClassName("x")` | `$(".x")`               |
| Select by tag   | `document.querySelectorAll("p")`       | `$("p")`                |
| Hide element    | `el.style.display = "none"`            | `$(el).hide()`          |
| Get input value | `document.getElementById("x").value`   | `$("#x").val()`         |
| Add CSS class   | `el.classList.add("x")`                | `$(el).addClass("x")`   |
| Click handler   | `el.addEventListener("click", fn)`     | `$(el).on("click", fn)` |
| AJAX GET        | 6+ lines of XHR                          | `$.get(url, callback)`  |

---

## 📦 What jQuery Gives You

```
jQuery has 5 main areas:

  1. DOM Selection    → $("selector") — find elements fast
  2. DOM Manipulation → .html(), .text(), .css(), .show(), .hide()
  3. Events           → .on(), .off(), .trigger()
  4. AJAX             → $.ajax(), $.get(), $.post()
  5. Utilities        → $.each(), $.trim(), $.isArray(), $.extend()
```

---

## ❓ Interview Questions

**Q: What is jQuery?**

> A JavaScript library that simplifies DOM selection, DOM manipulation, event handling, and AJAX requests. It wraps browser APIs in a consistent, short syntax.

**Q: What is `$` in jQuery?**

> `$` is just an alias for the `jQuery` function. `$("#id")` and `jQuery("#id")` do the exact same thing. It's short to type and became the standard way to use jQuery.

**Q: Why is jQuery important for Kendo UI?**

> Kendo UI is built on top of jQuery — every Kendo widget depends on jQuery being loaded first. All Kendo AJAX calls use jQuery's `$.ajax` internally, and you access widget instances using jQuery's `.data()` method.

**Q: What is document ready and why do you need it?**

> Document ready (`$(function() { })`) ensures your code runs only after the HTML is fully parsed. Without it, your code runs immediately and tries to find elements that don't exist yet.
>
