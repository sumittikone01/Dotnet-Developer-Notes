# 🏗️ Parameterized Constructor

## 📌 What is it?

> A constructor that **accepts arguments** to initialize an object with specific values at creation time, instead of relying on defaults.

---

## 💻 Code Examples

**Basic:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
    }
}

var p = new Product("Laptop", 55000m);
Console.WriteLine(p.Name);   // "Laptop"
```

**Intermediate — constructor overloading (multiple parameterized constructors):**

```csharp
public class Product
{
    public string Name;
    public decimal Price;
    public string Category;

    public Product(string name, decimal price)
    {
        Name = name;
        Price = price;
        Category = "General";
    }

    public Product(string name, decimal price, string category)
    {
        Name = name;
        Price = price;
        Category = category;
    }
}

var p1 = new Product("Laptop", 55000m);
var p2 = new Product("Laptop", 55000m, "Electronics");
```

**Practical — required data enforced via constructor (real-world pattern):**

```csharp
public class Order
{
    public int CustomerId { get; }
    public DateTime OrderDate { get; }
    public List<OrderItem> Items { get; } = new();

    public Order(int customerId)   // forces CustomerId to always be provided
    {
        if (customerId <= 0)
            throw new ArgumentException("Invalid customer ID");

        CustomerId = customerId;
        OrderDate = DateTime.Now;
    }
}
```

> 💡 Using a parameterized constructor + `{ get; }` (no setter) is a common pattern to guarantee an object can never exist in an invalid state — required data must be provided upfront.

---

## 📊 Default vs Parameterized Constructor

| Aspect          | Default Constructor               | Parameterized Constructor                     |
| --------------- | --------------------------------- | --------------------------------------------- |
| Parameters      | None                              | One or more                                   |
| Use case        | Simple/optional initialization    | Enforce required data at creation             |
| Auto-generated? | Yes, if no constructor is defined | Never auto-generated — must write explicitly |

---

## 🚨 Common Mistakes

- ❌ Not validating input inside a parameterized constructor — allows creation of invalid objects
- ❌ Having too many overloaded constructors — consider a builder pattern or object initializer syntax instead for many optional fields
- ❌ Forgetting: if only parameterized constructors exist, `new ClassName()` (no args) will fail to compile

---

## 🎤 Interview Questions

| Question                                                              | Key Point                                                                |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Why use a parameterized constructor?                                  | To enforce that required data is provided when an object is created      |
| Can a class have multiple parameterized constructors?                 | Yes — this is constructor overloading, differentiated by parameter list |
| What's a good practice when required data must never be null/invalid? | Validate in the constructor and use read-only properties (`{ get; }`)  |

---

## 📝 30-Second Revision Cheat Sheet

- Parameterized constructor = takes arguments, initializes object with specific values
- Never auto-generated — must be explicitly written
- Great for enforcing required data + validation at creation time
- Can be overloaded like regular methods
