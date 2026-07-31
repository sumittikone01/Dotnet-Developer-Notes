
# 02 — View Responsibilities

## 📌 What is it?

The **View** is responsible for **presentation only** — turning data (usually a Model or ViewModel passed from the Controller) into HTML that gets sent to the browser. In ASP.NET Core MVC, Views are written using the **Razor syntax** (`.cshtml` files).

## 🤔 Why do we need it?

Separating presentation from logic means:

- Designers/front-end devs can work on Views without touching business logic
- The same data (Model) could theoretically be rendered by different Views (e.g., a mobile-optimized version)
- Views stay simple, testable via visual/manual QA rather than needing complex unit tests

## 🧠 Intuition

The View is a **template with blanks to fill in**. The Controller decides *what* data to send; the View decides *how* to display it. The View should never decide *what* the data should be — that's not its job.

## 💻 Code example

```csharp
// Controller
public IActionResult Details(int id)
{
    var product = _productService.GetById(id);
    var viewModel = new ProductDetailsViewModel
    {
        ProductName = product.Name,
        FormattedPrice = product.Price.ToString("C"),
        CanAddToCart = product.IsInStock()
    };
    return View(viewModel); // View() looks for Views/Product/Details.cshtml
}
```

```html
@* Views/Product/Details.cshtml *@
@model ProductDetailsViewModel

<h2>@Model.ProductName</h2>
<p>Price: @Model.FormattedPrice</p>

@if (Model.CanAddToCart)
{
    <button class="btn btn-primary">Add to Cart</button>
}
else
{
    <p class="text-muted">Out of stock</p>
}
```

Notice: the View only makes **display decisions** (show button or not) based on data it was *given* — it doesn't query the database or decide business rules itself (`CanAddToCart` was already computed in the Controller/Model).

## ⚙️ What belongs in a View

- HTML markup + Razor syntax for rendering data
- Simple presentation logic: loops (`@foreach`), conditionals (`@if`) for **display purposes**
- Calls to Tag Helpers / HTML Helpers for form elements, links, etc.
- References to partial views / view components for reusable UI chunks

## 🚨 What does NOT belong in a View

- Database queries or calls to repositories/services directly
- Complex business logic ("is this user allowed to see this price?") — that decision should be made upstream, and the View just renders the *result*
- Heavy C# logic — if you're writing loops inside loops with complex branching in a `.cshtml` file, that logic likely belongs in the Controller or a ViewModel property instead

## 📊 Views vs Partial Views vs View Components (quick preview — detailed later in F.Views)

| Type                     | Purpose                                                                                                                     |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| **View**           | Full page (or section) tied to a specific Controller action                                                                 |
| **Partial View**   | Reusable chunk of markup, no independent logic, rendered with a Model passed in                                             |
| **View Component** | Mini self-contained MVC (has its own logic + view) — used for things like a cart summary widget that appears on many pages |

## 🚨 Common mistakes

- Injecting a `DbContext` or Service directly into a View to fetch extra data — this bypasses the Controller entirely and violates separation of concerns (even though ASP.NET Core technically allows `@inject` for this — use sparingly, mainly for things like `SignInManager` in Identity-related partials).
- Writing complex conditional business logic in Razor (`@if (user.Role == "Admin" && order.Status != "Cancelled" && ...)`) — compute this as a boolean property on the ViewModel instead (`Model.CanEditOrder`).
- Duplicating formatting logic across multiple Views instead of using Tag Helpers, HTML Helpers, or ViewModel properties.

## 💡 Best practices

- Push decision-making ("should this be shown?") into the ViewModel as a pre-computed boolean/property; keep the View's `@if` blocks simple and readable.
- Use **Partial Views** or **View Components** for any markup repeated across pages (headers, cart widgets, sidebars).
- Keep Razor code blocks (`@{ ... }`) minimal — a View heavy with C# logic is a smell that logic has leaked into the wrong layer.

## 🎤 Interview questions

1. What's the difference between a Partial View and a View Component, conceptually?
2. Why is it discouraged to inject a database context directly into a View, even though it's technically possible?
3. Where should the decision "should this button be visible?" be computed — in the View or the ViewModel? Why?
4. What's the relationship between a Controller action, the data it returns, and the View file it renders?

## 📝 30-second revision cheat sheet

- View = presentation only, written in Razor (`.cshtml`).
- Should NOT contain business logic, DB access, or complex decision-making — push that to Controller/ViewModel.
- Use Partial Views (simple reusable markup) and View Components (self-contained mini-MVC) for reusable UI.
- Rule of thumb: if a View needs deep C# logic, that logic belongs elsewhere.
