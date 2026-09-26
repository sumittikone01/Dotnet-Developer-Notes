# 03_Expression_Bodied_Members

> **Expression-bodied members** = using `=> expression` instead of a full `{ return expression; }` block body, for methods, properties, constructors, and more — a concise syntax for members whose entire logic is a single expression.

> Continues **G.Modern_CSharp**. Same family as `01`/`02`: syntax sugar that trims boilerplate, this time for member DEFINITIONS rather than string-building or null-handling.

## 📌 What is it?

```csharp
// Traditional method body
public decimal GetTotal()
{
    return Price * Quantity;
}

// Expression-bodied method — same thing, one line
public decimal GetTotal() => Price * Quantity;
```

The `=>` ("goes to" / "arrow") syntax you already know from LINQ lambdas (`p => p.Price`) is reused here at the member-declaration level — for any member whose entire body is a single expression.

## 🤔 Why do we need it?

| Problem with full block bodies for trivial members                                        | How expression bodies help                   |
| ----------------------------------------------------------------------------------------- | -------------------------------------------- |
| A one-line calculated property needs 4 lines of boilerplate (`{ get { return ...; } }`) | Collapses to one clean line                  |
| Simple pass-through/computed methods add visual noise to a class                          | Signals "this member is trivial" at a glance |
| Read-only computed properties are extremely common but verbose the traditional way        | `=>` makes them as short as a field        |

## 🌍 Real-world analogy

Like the difference between writing **"The total is calculated by multiplying price by quantity, and the result is: [calculation]"** versus just writing **"Total = Price × Quantity"** on a whiteboard. Both convey the same fact — the second is just far more direct for something this simple.

## 📊 Where Expression Bodies Can Be Used

| Member type               | Traditional                                                   | Expression-bodied                                 |
| ------------------------- | ------------------------------------------------------------- | ------------------------------------------------- |
| Method                    | `public int Add(int a,int b) { return a+b; }`               | `public int Add(int a, int b) => a + b;`        |
| Read-only property        | `public string FullName { get { return First+" "+Last; } }` | `public string FullName => First + " " + Last;` |
| Property with get AND set | `{ get { return _x; } set { _x = value; } }`                | `{ get => _x; set => _x = value; }`             |
| Constructor               | `public Product(string name) { Name = name; }`              | `public Product(string name) => Name = name;`   |
| Finalizer/Destructor      | `~Product() { Cleanup(); }`                                 | `~Product() => Cleanup();`                      |
| Indexer                   | `public T this[int i] { get { return _items[i]; } }`        | `public T this[int i] => _items[i];`            |

## ⚙️ Internal working — it's purely syntax, not a different runtime mechanism

```csharp
public decimal GetTotal() => Price * Quantity;

// The COMPILER treats this EXACTLY the same as:
public decimal GetTotal()
{
    return Price * Quantity;
}
```

> There is **zero runtime difference** — expression-bodied members compile to identical IL as their block-body equivalents. This is purely a source-code readability choice, exactly like `02_Query_Syntax.md` vs `03_Method_Syntax.md` for LINQ.

## 💻 Code examples

### Basic — expression-bodied read-only properties (extremely common pattern)

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public int Quantity { get; set; }

    // Computed, read-only property — no backing field needed, recalculated every access
    public decimal TotalValue => Price * Quantity;

    public bool IsInStock => Quantity > 0;

    public string DisplayLabel => $"{Name} - {Price:C}"; // combines with 02_String_Interpolation.md
}
```

### Intermediate — expression-bodied methods and a get/set property

```csharp
public class ProductService
{
    private readonly ProductDAL _dal;
    public ProductService(ProductDAL dal) => _dal = dal;  // expression-bodied constructor

    public Product? GetById(int id) => _dal.GetProductById(id);   // simple pass-through method

    public bool Exists(int id) => _dal.GetProductById(id) != null; // single-expression logic
}

public class Temperature
{
    private double _celsius;

    public double Celsius
    {
        get => _celsius;                          // expression-bodied getter
        set => _celsius = value;                    // expression-bodied setter
    }

    public double Fahrenheit
    {
        get => _celsius * 9 / 5 + 32;               // computed getter
        set => _celsius = (value - 32) * 5 / 9;      // computed setter (converts back on assignment)
    }
}
```

### Practical — in a typical Controller/BAL, where it keeps thin wrapper methods clean

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly ProductService _service;
    public ProductsController(ProductService service) => _service = service;

    [HttpGet("{id}")]
    public IActionResult GetById(int id) =>
        _service.GetById(id) is Product product ? Ok(product) : NotFound();
        // combines expression body + pattern matching (see H./Exception_Handling or C#'s pattern matching notes)
}
```

## ⚡ Performance considerations

- **None** — expression-bodied members compile to identical IL as block-bodied equivalents. This is a purely stylistic/readability choice with zero performance impact either way.

## 🚨 Common mistakes

- ❌ Forcing multi-statement logic into an expression body via comma operators or overly clever single-expression tricks — if it needs more than one real "step," a normal block body is clearer.
- ❌ Using expression bodies for properties/methods that will likely need to grow more complex soon — a trivial `=>` member often needs converting back to a block body the moment real logic (validation, logging) is added; that's fine and normal, but don't fight it.
- ❌ Overusing expression bodies to the point that a class becomes a wall of terse one-liners that's harder to scan than clearly separated block-bodied methods — readability, not brevity, is the actual goal.

## 💡 Best practices

- ✅ Use expression bodies for genuinely trivial members: simple computed properties, pass-through methods, straightforward constructors.
- ✅ Switch to a block body the moment a member needs more than one real step (validation, multiple statements, early returns) — don't force it to stay an expression body.
- ✅ Combine naturally with other modern C# features covered in this chapter (string interpolation, null-conditional operators, pattern matching) for concise, readable one-liners.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                      | Answer                                                                                                                                                                               |
| --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| What is an expression-bodied member?                                                          | A member (method, property, constructor, etc.) whose entire body is written as a single expression using`=>`, instead of a `{ }` block                                           |
| Is there a runtime performance difference between expression-bodied and block-bodied members? | No — they compile to identical IL; it's purely a source-code style choice                                                                                                           |
| Name three kinds of members that can be expression-bodied.                                    | Methods, properties (get/set), constructors (also valid: indexers, finalizers)                                                                                                       |
| When should you prefer a full block body over an expression body?                             | When the member's logic requires more than one real step — multiple statements, validation, or branching logic                                                                      |
| How does an expression-bodied get/set property differ from a computed read-only property?     | A read-only expression-bodied property (`=> expr`) has no setter at all; a get/set property can independently define `get => ...` and `set => ...` for two-way computed access |

## 📝 30-second Revision Cheat Sheet

- `=> expression` replaces a `{ return expression; }` block for a single-expression member.
- Works on methods, properties, constructors, indexers, finalizers.
- Zero runtime difference from a block body — pure readability/style choice.
- Best for genuinely trivial members (computed properties, pass-through methods, simple constructors).
- Switch back to a block body the moment logic needs more than one real step.
