
# 🧊 Immutability & readonly

## 📌 What is it?

> **Immutability** means an object's state cannot change after creation. In C#, `readonly` (fields), `{ get; }`/`{ get; init; }` (properties), and `readonly struct` are the main tools to enforce it.

---

## 🤔 Why do we need it?

- **Thread safety** — immutable objects can be shared across threads without locks (no risk of concurrent modification)
- **Predictability** — an object's state can't change unexpectedly elsewhere in the code
- **Easier debugging** — fewer places where a bug could have mutated the object

---

## 💻 Code Examples

**Basic — readonly field:**

```csharp
public class Order
{
    public readonly DateTime CreatedDate;   // can only be set in constructor or inline

    public Order()
    {
        CreatedDate = DateTime.Now;   // ✅ allowed here
    }

    public void UpdateDate()
    {
        // CreatedDate = DateTime.Now;  ❌ Compile error — can't modify after construction
    }
}
```

**Intermediate — fully immutable class (common ViewModel/DTO pattern):**

```csharp
public class ProductDto
{
    public string Name { get; }
    public decimal Price { get; }

    public ProductDto(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

var dto = new ProductDto("Laptop", 55000m);
// dto.Price = 60000m;  ❌ Compile error — no setter exists at all
```

**Practical — modern immutability with `init` (C# 9+) and records:**

```csharp
public class ProductDto
{
    public string Name { get; init; }
    public decimal Price { get; init; }
}

var dto = new ProductDto { Name = "Laptop", Price = 55000m };   // set via object initializer
// dto.Price = 60000m;  ❌ Compile error — init-only, can't change after creation

// records (C# 9+) take this further — built-in immutability + value equality
public record ProductRecord(string Name, decimal Price);
var rec = new ProductRecord("Laptop", 55000m);
var rec2 = rec with { Price = 60000m };   // creates a NEW record with one field changed — original unchanged
```

> More on `init` in `D.Language_Features/10_init_Only_Setters.md` and Records in `D.Language_Features/07_Records.md`.

---

## 📊 readonly Field vs const vs init

| Keyword             | Set at                                   | Per-instance?         | Can depend on runtime values? |
| ------------------- | ---------------------------------------- | --------------------- | ----------------------------- |
| `const`           | Compile-time                             | ❌ No — same for all | ❌ No                         |
| `readonly`        | Runtime (constructor)                    | ✅ Yes                | ✅ Yes                        |
| `init` (property) | Runtime (object initializer/constructor) | ✅ Yes                | ✅ Yes                        |

---

## 🚨 Common Mistakes

- ❌ Thinking `readonly` on a reference-type field makes the OBJECT immutable — it only prevents REASSIGNING the reference; the object's own internal fields could still be mutated if they're not also readonly
- ❌ Overusing mutable state in multi-threaded/concurrent contexts (like ASP.NET Core request handling) instead of leaning on immutability for safety
- ❌ Confusing `readonly` (field-level) with `const` — see `03_Variables_and_Constants.md` for the full comparison

---

## 🎤 Interview Questions

| Question                                                | Key Point                                                                                                               |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| What's the benefit of immutability?                     | Thread safety, predictability, fewer bugs from unexpected state mutation                                                |
| Does`readonly` make an object fully immutable?        | Not necessarily — only prevents reassigning the field itself; nested mutable objects can still change internally       |
| What's the difference between`init` and `readonly`? | `init` is for properties, settable via object initializer syntax; `readonly` is for fields, settable in constructor |

---

## 📝 30-Second Revision Cheat Sheet

- Immutability = state can't change after creation → thread-safe, predictable
- `readonly` (field) → settable only in constructor
- `init` (property, C# 9+) → settable only during object initialization
- `readonly` on a reference type only locks the REFERENCE, not necessarily the object's internal mutability
