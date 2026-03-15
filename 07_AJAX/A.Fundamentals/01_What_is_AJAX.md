
# 01 — What is AJAX?

---

## 🎯 One-Line Definition

> **AJAX lets your web page talk to the server in the background — without reloading the page.**

That's it. Everything else is detail.

---

## 🤔 Why Does AJAX Exist? — The Problem First

Imagine filling out a long registration form. You type everything and click  **Submit** .
The **entire page goes white** and reloads. You're back to a blank form.

That's the world before AJAX.

Every single user action that needed server data caused a  **full page reload** :

* Click a button → page reloads
* Submit a form → page reloads
* Click "next page" in a table → page reloads

This was **slow, jarring, and frustrating** for users.

---

## 💡 What AJAX Changed

With AJAX, the page  **stays alive** . Only the data that needs to change, changes.

```
WITHOUT AJAX                         WITH AJAX
─────────────────────────────────    ─────────────────────────────────
User clicks "Delete Employee"        User clicks "Delete Employee"
         │                                    │
         ▼                                    ▼
Whole page goes white              Small background request sent
Whole page reloads from scratch    Page stays exactly as it is
User loses scroll position         JavaScript removes that one row
User sees the same page again      Done. Fast. No flash. No reload.
(took 1-2 seconds)                 (took ~100ms)
```

---

## 📱 AJAX is Everywhere — You Use It Every Day

| App                       | Where AJAX Runs                                          |
| ------------------------- | -------------------------------------------------------- |
| **Google Search**   | Suggestions appear as you type                           |
| **Gmail**           | New emails load without refreshing inbox                 |
| **YouTube**         | Like button updates instantly                            |
| **Instagram**       | New posts load as you scroll down                        |
| **Your Kendo Grid** | Data loads, rows save, rows delete — all without reload |
| **Any login form**  | "Email already exists" check while you type              |

Every one of these works because **JavaScript quietly sends a request in the background** and updates only the part of the page that needs to change.

---

## 📖 What Does the Name Mean?

**A**synchronous **J**avaScript **A**nd **X**ML

| Letter      | Means        | Reality Today                               |
| ----------- | ------------ | ------------------------------------------- |
| **A** | Asynchronous | ✅ Still true — runs in background         |
| **J** | JavaScript   | ✅ Still true — JS makes the requests      |
| **A** | And          | —                                          |
| **X** | XML          | ❌ Nobody uses XML anymore — it's JSON now |

> The name is old (2005). The "X" should be "J" for JSON today. But the name stuck.

---

## 🔄 How AJAX Works — The Full Picture

```
  YOUR BROWSER                              YOUR SERVER
  ────────────                              ────────────

  [1] User clicks "Load Employees"

  [2] JavaScript creates a background request
      "GET /Employee/GetAll"
                    │
                    │  ──── travels over internet ────►
                    │
                    │                       [3] Controller runs
                    │                           queries database
                    │                           builds JSON data
                    │
                    │  ◄─── JSON response ─────────────
                    │
  [4] JavaScript receives JSON
      [ {id:1, name:"Alice"}, {id:2, name:"Bob"} ]

  [5] JavaScript updates the page
      Adds rows to the table

  PAGE NEVER RELOADED.
  User never saw a white flash.
  User can still click/type/scroll during steps 2-4.
```

---

## 🧰 AJAX is NOT a Language or Library

This is a common confusion. Clear it up now:

| What is AJAX?                  | What is it NOT?                       |
| ------------------------------ | ------------------------------------- |
| A**technique**           | Not a programming language            |
| Uses existing browser features | Not a framework you install           |
| Works in every browser         | Not specific to any technology        |
| Can be used with any server    | Not limited to ASP.NET or any backend |

Think of AJAX like "texting while driving" — it's a technique (don't actually do that), not a technology itself.

---

## 🛠️ Three Ways to Write AJAX in JavaScript

You'll see AJAX written in three ways. Know all three:

```
┌─────────────────────────────────────────────────────────┐
│  1. XMLHttpRequest (XHR)                                 │
│     The original. Verbose. Still in older codebases.    │
│     Good to understand. You won't write this often.     │
├─────────────────────────────────────────────────────────┤
│  2. Fetch API                                            │
│     Modern built-in. Clean. Promise-based.              │
│     Standard in new JavaScript projects.                │
├─────────────────────────────────────────────────────────┤
│  3. jQuery $.ajax / $.get / $.post      ← YOU USE THIS  │
│     Simplest syntax. Works with Kendo UI.               │
│     Most common in ASP.NET MVC projects.                │
└─────────────────────────────────────────────────────────┘
```

A quick look at all three doing the same thing — loading employee data:

```javascript
// ── Way 1: XMLHttpRequest (old, verbose) ──────────────────
var xhr = new XMLHttpRequest();
xhr.open("GET", "/Employee/GetAll");
xhr.onload = function() {
    var data = JSON.parse(xhr.responseText);
    console.log(data);
};
xhr.send();

// ── Way 2: Fetch API (modern) ─────────────────────────────
fetch("/Employee/GetAll")
    .then(response => response.json())
    .then(data => console.log(data));

// ── Way 3: jQuery $.get (simplest) ───────────────────────
$.get("/Employee/GetAll", function(data) {
    console.log(data);
});
```

All three do  **exactly the same thing** . jQuery is the shortest and what you'll use in MVC + Kendo projects.

---

## 🏗️ AJAX in Your ASP.NET MVC Project

```
  RAZOR VIEW (.cshtml)                    CONTROLLER (.cs)
  ────────────────────                    ────────────────

  Kendo Grid needs data
         │
         │ AJAX POST /Employee/Read ──────► [HttpPost]
         │                                  public JsonResult Read(...)
         │                                  {
         │                                      var data = _db.Employees...
         │                                      return Json(data);
         │                                  }
         │ ◄──── JSON: { Data:[...], Total:50 }
         │
  Grid shows rows in table
  No page reload happened
```

Every Kendo widget — Grid, Chart, DropDownList with remote data —  **uses AJAX to talk to your controller** . When you configure a DataSource with a `read` URL, Kendo is making an AJAX call for you automatically.

---

## ✅ Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│  Remember These 5 Things About AJAX                      │
│                                                          │
│  1. Background request — page stays alive               │
│  2. Uses JSON today (not XML, ignore the name)          │
│  3. Not a language — it's a technique using JS + HTTP   │
│  4. Kendo Grid uses AJAX for all its data operations    │
│  5. Three ways: XHR (old) → Fetch (modern) → jQuery ✓  │
└─────────────────────────────────────────────────────────┘
```

---

## ❓ Interview Questions

**Q: What is AJAX in one sentence?**

> AJAX is a technique that lets JavaScript send HTTP requests to the server in the background and update parts of the page without a full reload.

**Q: Does AJAX use XML?**

> The name says XML but JSON is used in almost all modern applications. The name is historical.

**Q: Is AJAX a framework or a language?**

> Neither. It's a technique that uses existing browser features — specifically the XMLHttpRequest or Fetch API built into every browser.

**Q: What happens on the server side when an AJAX request arrives?**

> The server receives a normal HTTP request — it doesn't know or care if it came from AJAX or a form. In ASP.NET MVC, a controller action handles it and returns JSON instead of a full HTML page.
