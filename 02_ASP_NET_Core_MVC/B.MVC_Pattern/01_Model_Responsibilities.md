# 01 — Model Responsibilities

## 📌 What is it?

The **Model** in MVC represents the application's **data and business logic**. It is the layer that knows about the "what" — what the data looks like, what rules govern it, and how it's persisted — completely independent of how it's displayed or how requests are handled.

## 🤔 Why do we need it?

Without a clearly defined Model layer, business rules end up scattered across Controllers and Views — making them hard to test, reuse, or change without breaking the UI.

## 🧠 Intuition

The Model is the **"single source of truth"** for your data and rules. Both the Controller and the View depend on it, but it depends on neither of them — a clean, one-directional relationship.

## 📊 Types of "Models" you'll encounter (this trips up many devs)

| Type                                     | Purpose                                                                                                           | Example                                                           |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Domain Model / Entity**          | Represents a real business object, often mapped to a database table                                               | `Product`, `Order`, `Customer`                              |
| **ViewModel**                      | Shaped specifically for what a View needs — may combine multiple entities or omit fields                         | `ProductListViewModel` (Product + category name + review count) |
| **DTO (Data Transfer Object)**     | Used to transfer data across layers/boundaries (e.g., API request/response)                                       | `CreateProductDto`                                              |
| **Business/Service layer classes** | Not strictly "Models" in the folder sense, but part of the business logic often associated with the Model concept | `ProductService`, `OrderValidator`                            |

> ⚠️ In practice, "Model" is an overloaded term. The `Models/` folder in an MVC project often holds a mix of these — but conceptually, keep Domain Models, ViewModels, and DTOs distinct.

## 💻 Code example

```csharp
// Domain Model — represents the real business entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }

    // Business logic CAN live on the model itself
    public bool IsInStock() => StockQuantity > 0;
}

// ViewModel — shaped for a specific View's needs
public class ProductDetailsViewModel
{
    public string ProductName { get; set; }
    public string FormattedPrice { get; set; }   // pre-formatted for display
    public bool CanAddToCart { get; set; }
    public List<string> RelatedProductNames { get; set; }
}
```

Notice: the `ProductDetailsViewModel` doesn't map 1:1 to a database table — it's tailored exactly to what the View needs to render, nothing more.

## ⚙️ What belongs in the Model layer

- Data structure (properties)
- Validation rules (via Data Annotations or Fluent Validation — see `H.Model_Handling`)
- Business rules that are intrinsic to the entity itself (`IsInStock()`, `CalculateDiscount()`)
- Relationships between entities (`Product` has many `Reviews`)

## 🚨 What does NOT belong in the Model layer

- HTTP-specific concerns (query strings, request headers) — that's the Controller's job
- HTML-rendering logic — that's the View's job
- Direct dependency on `HttpContext` — Models should be usable outside a web context entirely (e.g., in unit tests, console apps, background services)

## 🚨 Common mistakes

- Using the **same class** as both the database entity AND the View's data shape — leads to over-exposing internal fields (like password hashes) directly to Views, or forcing awkward nullable fields that only make sense for one context.
- Putting validation logic only in the Controller instead of on the Model/ViewModel via Data Annotations — leads to logic duplication if the same Model is used elsewhere (e.g., an API).
- Giving Models a dependency on `HttpContext`, `Request`, or `Session` — breaks testability and reusability.

## 💡 Best practices

- Use **ViewModels** to decouple what the database looks like from what the UI needs — even if it feels like "extra classes" at first, it pays off quickly.
- Put simple, intrinsic business rules directly on the Model (e.g., `Order.CalculateTotal()`); put cross-entity or complex business rules in a **Service** class instead.
- Keep Models POCOs (Plain Old CLR Objects) — no framework-specific dependencies baked in.

## 🎤 Interview questions

1. What's the difference between a Domain Model, a ViewModel, and a DTO? Give an example of when you'd use each.
2. Why shouldn't a Model class depend on `HttpContext`?
3. Where should validation logic live, and why?
4. Give an example of business logic that correctly belongs on the Model itself vs. logic that belongs in a Service layer.

## 📝 30-second revision cheat sheet

- Model = data + business rules; the "single source of truth," independent of Controller/View.
- Three related-but-distinct types: **Domain Model** (DB entity), **ViewModel** (UI-shaped), **DTO** (transfer across boundaries).
- Models should be plain classes — no `HttpContext`, no HTML, no HTTP-specific logic.
- Intrinsic rules → on the Model; cross-entity/complex rules → in a Service layer.
