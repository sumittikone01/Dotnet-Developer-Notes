
# 🧩 Local Functions

## 📌 What is it?

> A method **defined inside another method**, only usable within that containing method's scope.

Introduced in C# 7+, local functions help encapsulate small helper logic that doesn't need to exist outside its parent method.

---

## 🧠 Intuition

```csharp
void ProcessOrder(Order order)
{
    if (!IsValid(order))          // local function called here
        throw new InvalidOperationException();

    // ... process order

    bool IsValid(Order o)          // LOCAL FUNCTION — only visible inside ProcessOrder
    {
        return o != null && o.Quantity > 0;
    }
}
```

> `IsValid` cannot be called from anywhere outside `ProcessOrder` — it's fully private to that method's scope.

---

## 📊 Local Functions vs Lambda Expressions vs Private Methods

| Aspect                            | Local Function                                           | Lambda / Anonymous Method                     | Private Method                  |
| --------------------------------- | -------------------------------------------------------- | --------------------------------------------- | ------------------------------- |
| Scope                             | Inside containing method only                            | Inside containing method only (as a variable) | Entire class                    |
| Can be reused elsewhere in class? | ❌ No                                                    | ❌ No                                         | ✅ Yes                          |
| Supports recursion easily?        | ✅ Yes, naturally                                        | ⚠️ Awkward (needs workarounds)              | ✅ Yes                          |
| Performance                       | Slightly better (no delegate allocation by default)      | Delegate allocation overhead                  | No overhead                     |
| Best for                          | Small helper logic used once, close to where it's needed | Short inline callbacks (LINQ, event handlers) | Reusable logic across the class |

---

## 💻 Code Examples

**Basic:**

```csharp
int CalculateTotal(int quantity, decimal price)
{
    ValidateInputs();   // local function call

    return (int)(quantity * price);

    void ValidateInputs()   // local function — can access outer variables!
    {
        if (quantity < 0) throw new ArgumentException("Quantity can't be negative");
        if (price < 0) throw new ArgumentException("Price can't be negative");
    }
}
```

**Intermediate — recursion inside a local function:**

```csharp
int Factorial(int n)
{
    return Calculate(n);

    int Calculate(int x)   // local recursive function
    {
        if (x == 0) return 1;
        return x * Calculate(x - 1);
    }
}
```

**Practical — real-world validation helper inside a service method:**

```csharp
public OrderDto CreateOrder(OrderRequest request)
{
    EnsureValidRequest();

    var order = new OrderDto
    {
        CustomerId = request.CustomerId,
        Total = request.Quantity * request.UnitPrice
    };
    return order;

    void EnsureValidRequest()
    {
        if (request.Quantity <= 0)
            throw new ArgumentException("Quantity must be positive");
        if (request.CustomerId <= 0)
            throw new ArgumentException("Invalid customer");
    }
}
```

---

## 🚨 Common Mistakes

- ❌ Using a local function when the logic is actually reusable across multiple methods — should be a private method instead
- ❌ Overusing local functions for large/complex logic — hurts readability of the containing method
- ❌ Confusing local functions with lambdas — local functions are declared with normal method syntax, not assigned to a variable

---

## 🎤 Interview Questions

| Question                                                           | Key Point                                                                                                            |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| What's the main benefit of a local function over a private method? | Scopes the helper logic tightly to where it's used — avoids polluting the class with single-use private methods     |
| Can a local function access variables from the containing method?  | Yes — it captures the enclosing scope, similar to a closure                                                         |
| Local function vs lambda — when to choose which?                  | Local function for named, possibly recursive helper logic; lambda for short inline callbacks (e.g., LINQ predicates) |

---

## 📝 30-Second Revision Cheat Sheet

- Local function = method defined inside another method, scoped to it only
- Naturally supports recursion, can access outer method's variables
- Use for small, single-use helper logic — not reusable across the class (use private method for that)
- Different from lambdas — declared with normal method syntax, not stored in a variable
