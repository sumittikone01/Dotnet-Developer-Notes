
# 02 — Razor Syntax

---

## 🎯 One-Line Definition

> **Razor syntax uses `@` to embed C# inside HTML — a single `@` renders a value, `@{ }` runs code blocks, and `@if` / `@foreach` / `@for` add control flow — keeping views readable with minimal noise.**

---

## 🔷 The `@` Symbol — The Only Thing You Need to Know

```
@ = "switch to C# right here"

<p>@Model.Name</p>         ← renders the value of Model.Name
@{ var x = 5; }            ← runs a C# code block
@if (Model.IsActive) { }   ← C# if statement
@foreach (var e in Model) { } ← C# foreach loop
@* this is a comment *@    ← Razor comment (not sent to browser)
@@                          ← literal @ character (escaped)
```

---

## 🔷 1 — Inline Expressions

Render a single C# value directly into HTML:

```cshtml
@* Simple property *@
<h2>@Model.Name</h2>
<p>@Model.Salary</p>

@* Method call *@
<p>@Model.Salary.ToString("C0")</p>
<p>@DateTime.Now.Year</p>
<p>@Model.HireDate.ToString("dd MMM yyyy")</p>

@* String operations *@
<p>@Model.Name.ToUpper()</p>
<p>@Model.Name.Trim()</p>

@* Ternary expression *@
<p>@(Model.IsActive ? "Active" : "Inactive")</p>
@*  ↑ complex expressions need () around them *@

@* String interpolation — use () *@
<p>@($"Welcome, {Model.Name}! Salary: {Model.Salary:C0}")</p>
```

---

## 🔷 2 — Code Blocks `@{ }`

Run multiple C# statements — output is NOT automatically rendered:

```cshtml
@{
    ViewData["Title"] = "Employee List";
    var count     = Model.Count;
    var highPaid  = Model.Where(e => e.Salary > 70000).ToList();
    var greeting  = DateTime.Now.Hour < 12 ? "Good morning" : "Good afternoon";
    string cssClass = count > 0 ? "text-success" : "text-danger";
}

<h1>@ViewData["Title"]</h1>
<p class="@cssClass">@count employees found</p>
<h2>@greeting!</h2>
```

---

## 🔷 3 — If / Else If / Else

```cshtml
@if (Model.Salary > 80000)
{
    <span class="badge bg-success">High Earner</span>
}
else if (Model.Salary > 50000)
{
    <span class="badge bg-warning">Mid Range</span>
}
else
{
    <span class="badge bg-secondary">Entry Level</span>
}

@* Inline ternary — for simple cases *@
<span class="badge @(Model.IsActive ? "bg-success" : "bg-danger")">
    @(Model.IsActive ? "Active" : "Inactive")
</span>
```

---

## 🔷 4 — foreach Loop

The most used loop in Razor — render a list of items:

```cshtml
@* Basic foreach *@
@foreach (var emp in Model)
{
    <tr>
        <td>@emp.Id</td>
        <td>@emp.Name</td>
        <td>@emp.Role</td>
        <td>@emp.Salary.ToString("C0")</td>
    </tr>
}

@* foreach with index (use for loop for index) *@
@{int i = 1;}
@foreach (var emp in Model)
{
    <tr>
        <td>@i</td>
        <td>@emp.Name</td>
    </tr>
    i++;
}

@* Empty state check *@
@if (Model == null || !Model.Any())
{
    <tr><td colspan="5" class="text-center">No employees found.</td></tr>
}
else
{
    @foreach (var emp in Model)
    {
        <tr>
            <td>@emp.Name</td>
        </tr>
    }
}
```

---

## 🔷 5 — for Loop

```cshtml
@* for with index — useful for numbered lists *@
@for (int i = 0; i < Model.Count; i++)
{
    <tr>
        <td>@(i + 1)</td>
        <td>@Model[i].Name</td>
        <td>@Model[i].Salary.ToString("C0")</td>
    </tr>
}
```

---

## 🔷 6 — switch Statement

```cshtml
@switch (Model.Role)
{
    case "Manager":
        <span class="badge bg-danger">Manager</span>
        break;
    case "Developer":
        <span class="badge bg-primary">Developer</span>
        break;
    case "QA":
        <span class="badge bg-warning">QA</span>
        break;
    default:
        <span class="badge bg-secondary">@Model.Role</span>
        break;
}
```

---

## 🔷 7 — Razor Comments

```cshtml
@* This is a Razor comment — NOT sent to the browser *@
@* The browser NEVER sees this text *@

<!-- This is an HTML comment — IS sent to the browser -->
<!-- Users can see this in page source -->

@* Use @* *@ for:
   - Temporarily disabling markup
   - Developer notes that should never reach the client
*@
```

---

## 🔷 8 — Parentheses for Complex Expressions

When your expression is more complex than a simple property, wrap it in `()`:

```cshtml
@* Simple — no parens needed *@
@Model.Name
@emp.Salary

@* Complex — needs parens *@
@(Model.Salary * 0.1)
@(Model.Name.Length > 20 ? Model.Name.Substring(0, 20) + "..." : Model.Name)
@(i + 1)
@(emp.HireDate.Year == DateTime.Now.Year ? "New hire" : "Existing")
@($"₹{Model.Salary:N0}")

@* In HTML attributes — always use parens *@
<div class="@(Model.IsActive ? "active" : "inactive")">
<input value="@(Model.Salary * 12)" />
```

---

## 🔷 9 — Rendering HTML Attributes Conditionally

```cshtml
@* Add disabled attribute conditionally *@
<button type="submit"
        @(Model.IsActive ? "" : "disabled")>
    Save
</button>

@* Add CSS class conditionally *@
<tr class="@(emp.Salary > 80000 ? "table-success" : "")">
    <td>@emp.Name</td>
</tr>

@* Conditional data attributes *@
<div data-id="@emp.Id"
     data-active="@emp.IsActive.ToString().ToLower()">
</div>
```

---

## 🔷 10 — Using C# Variables in Loops

```cshtml
@{
    var total    = Model.Sum(e => e.Salary);
    var avgSalary = Model.Any() ? Model.Average(e => e.Salary) : 0;
}

<table class="table">
    <thead>
        <tr><th>Name</th><th>Salary</th><th>% of Total</th></tr>
    </thead>
    <tbody>
        @foreach (var emp in Model)
        {
            double pct = total > 0 ? (double)(emp.Salary / total * 100) : 0;
            <tr>
                <td>@emp.Name</td>
                <td>@emp.Salary.ToString("C0")</td>
                <td>@pct.ToString("F1")%</td>
            </tr>
        }
    </tbody>
    <tfoot>
        <tr class="table-dark fw-bold">
            <td>Total</td>
            <td>@total.ToString("C0")</td>
            <td>100%</td>
        </tr>
    </tfoot>
</table>
<p>Average Salary: @avgSalary.ToString("C0")</p>
```

---

## 🔷 11 — Escaping `@` — When You Need a Literal `@`

```cshtml
@* To output a literal @ character, write @@ *@

<p>Email us at support@@company.com</p>
@* Renders: Email us at support@company.com *@

@* In email input, @ is fine in the value — only in HTML text it's special *@
<input type="email" value="support@company.com" />  @* fine — inside attribute *@
```

---

## 🔷 Razor Syntax Quick Reference

| Syntax                      | What It Does              | Example                         |
| --------------------------- | ------------------------- | ------------------------------- |
| `@expression`             | Render a value            | `@Model.Name`                 |
| `@(complex)`              | Render complex expression | `@(i + 1)`                    |
| `@{ }`                    | Code block                | `@{ var x = 5; }`             |
| `@if(...){ }`             | Conditional               | `@if(Model.IsActive){ }`      |
| `@foreach(... in ...){ }` | Loop                      | `@foreach(var e in Model){ }` |
| `@for(...){ }`            | Indexed loop              | `@for(int i=0; i<5; i++){ }`  |
| `@switch(...){ }`         | Switch                    | `@switch(Model.Role){ }`      |
| `@* *@`                   | Comment                   | `@* this is hidden *@`        |
| `@@`                      | Literal @ character       | `user@@domain.com`            |
| `@Html.Raw(x)`            | Render unencoded HTML     | `@Html.Raw(content)`          |

---

## ⚠️ Common Razor Syntax Mistakes

| Mistake                                  | What Happens                                    | Fix                                   |
| ---------------------------------------- | ----------------------------------------------- | ------------------------------------- |
| `@Model.Salary * 12`without parens     | Renders `Model.Salary`then ` * 12`as text   | Use `@(Model.Salary * 12)`          |
| Using `@`inside HTML attribute strings | Interpreted as another expression               | `class="@(condition ? "a" : "b")"`  |
| `<!-- @* combined comment *@ -->`      | Confused parser — HTML comment sent to browser | Use `@* *@`for server-side comments |
| Multi-line expression without `@{ }`   | Parse error                                     | Put multi-line logic in `@{ }`block |

---

## ⭐ Interview Quick-Fire

| Question                                                   | Answer                                                                    |
| ---------------------------------------------------------- | ------------------------------------------------------------------------- |
| What does `@`do in Razor?                                | Switches from HTML to C# mode — renders value or starts code             |
| When do you need `@()`parentheses?                       | For complex expressions: arithmetic, ternary, method chains               |
| What is a Razor code block?                                | `@{ }`— runs C# statements without rendering output                    |
| How do you write a Razor comment?                          | `@* comment here *@`— not sent to browser                              |
| How to output a literal `@`symbol?                       | Use `@@`                                                                |
| Does Razor encode output automatically?                    | ✅ Yes — prevents XSS. Use `@Html.Raw()`only for trusted HTML          |
| What's the difference between `@{ }`and `@expression`? | `@{ }`runs code silently.`@expression`evaluates and renders the value |
