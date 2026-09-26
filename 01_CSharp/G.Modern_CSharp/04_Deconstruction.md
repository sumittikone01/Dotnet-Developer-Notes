# 04_Deconstruction

> **Deconstruction** = unpacking an object's data into separate, named variables in one statement — `var (name, price) = product;` — instead of accessing each property individually.

> Continues **G.Modern_CSharp**. Pairs especially well with `02_String_Interpolation.md` (deconstruct, then interpolate the pieces) and sets up `05_Spread_and_Indices.md`'s pattern-based syntax sugar.

## 📌 What is it?

```csharp
// WITHOUT deconstruction — access each property individually
Product product = _dal.GetProductById(5);
string name = product.Name;
decimal price = product.Price;

// WITH deconstruction — unpack BOTH in one statement
var (name, price) = product;
```

Tuples get this "for free" (they're deconstructible by default); custom classes/structs need a `Deconstruct` method added, which C# then recognizes automatically.

## 🤔 Why do we need it?

| Problem without deconstruction                                                          | How it helps                                                         |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Returning multiple values requires an`out` parameter or a custom wrapper class        | Return a tuple, deconstruct it into named variables at the call site |
| Repeatedly writing`result.Item1`, `result.Item2` for an unnamed tuple is unreadable | Deconstruction gives each piece a meaningful local name immediately  |
| Extracting a few fields from an object for a quick calculation feels verbose            | One line pulls out exactly the fields you need, named clearly        |

## 🌍 Real-world analogy

Opening a **care package with pre-labeled compartments** — instead of reaching in and pulling out "item 1," "item 2," "item 3" one at a time and having to remember which is which, you tip it out onto a table where each item is immediately in its own clearly labeled spot: `name`, `price`, `category`.

## ⚙️ Internal working — tuples are deconstructible by default

```csharp
(string Name, decimal Price) GetProductInfo(int id)
{
    var product = _dal.GetProductById(id);
    return (product.Name, product.Price); // returns a TUPLE
}

var (name, price) = GetProductInfo(5); // DECONSTRUCTS the tuple into two separate variables
Console.WriteLine($"{name}: {price:C}"); // combines with 02_String_Interpolation.md
```

## ⚙️ Internal working — making a CUSTOM class deconstructible

```csharp
public class Product
{
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
    public string Category { get; set; } = "";

    // Adding a Deconstruct method makes deconstruction syntax work for THIS class
    public void Deconstruct(out string name, out decimal price)
    {
        name = Name;
        price = Price;
    }
}

Product product = _dal.GetProductById(5);
var (name, price) = product; // now works — calls the Deconstruct method above!
```

```
var (name, price) = product;
         │
         ▼
The COMPILER looks for a method matching:
   void Deconstruct(out string name, out decimal price)
on the Product class, and calls it, assigning the OUT parameters
to the new local variables 'name' and 'price'.
```

## 📊 Deconstruction Targets — where it can be used

| Context                                      | Example                                                                |
| -------------------------------------------- | ---------------------------------------------------------------------- |
| Local variable declaration                   | `var (name, price) = product;`                                       |
| Existing variables (no`var`)               | `(name, price) = product;` (reassigns existing variables)            |
| Discards for unwanted values                 | `var (name, _, category) = product;` (ignore the 2nd value entirely) |
| `foreach` loop over a collection of tuples | `foreach (var (id, name) in idNamePairs) { ... }`                    |
| Pattern matching (`switch` expressions)    | `case (int x, int y) when x == y: ...`                               |

## 💻 Code examples

### Basic — deconstructing a tuple returned from a method

```csharp
public (int Count, decimal Total) GetOrderStats(List<Order> orders)
{
    return (orders.Count, orders.Sum(o => o.Amount));
}

var (count, total) = GetOrderStats(todaysOrders);
Console.WriteLine($"{count} orders totaling {total:C}");
```

### Intermediate — deconstructing in a `foreach` over key-value pairs

```csharp
Dictionary<int, string> productNames = _dal.GetProductNameLookup();

// KeyValuePair<TKey,TValue> is deconstructible OUT OF THE BOX
foreach (var (id, name) in productNames)
{
    Console.WriteLine($"Product #{id}: {name}");
}

// Equivalent WITHOUT deconstruction (more verbose):
foreach (var kvp in productNames)
{
    Console.WriteLine($"Product #{kvp.Key}: {kvp.Value}");
}
```

### Practical — a custom `Deconstruct` on a domain class, with discards

```csharp
public class OrderSummary
{
    public int OrderId { get; set; }
    public string CustomerName { get; set; } = "";
    public decimal Amount { get; set; }
    public string Status { get; set; } = "";

    public void Deconstruct(out int orderId, out string customerName, out decimal amount, out string status)
    {
        orderId = OrderId;
        customerName = CustomerName;
        amount = Amount;
        status = Status;
    }
}

OrderSummary summary = _service.GetOrderSummary(101);

// Only care about orderId and amount — discard the rest with '_'
var (orderId, _, amount, _) = summary;
Console.WriteLine($"Order #{orderId}: {amount:C}");
```

### Deconstruction inside pattern matching (a preview of pattern-matching territory)

```csharp
static string DescribePoint((int X, int Y) point) => point switch
{
    (0, 0) => "Origin",
    (var x, 0) => $"On the X-axis at {x}",
    (0, var y) => $"On the Y-axis at {y}",
    (var x, var y) => $"Point at ({x}, {y})"
};
```

## ⚡ Performance considerations

- Deconstruction compiles down to simple sequential assignments (or `out` parameter calls) — there is **no meaningful performance overhead** compared to accessing properties individually.
- Tuples (`(string, decimal)`) used for deconstruction are `ValueTuple`s by default in modern C# — a **struct**, meaning no extra heap allocation compared to a reference-type wrapper class, which is a nice, free efficiency bonus alongside the readability win.

## 🚨 Common mistakes

- ❌ Adding a `Deconstruct` method with SO many `out` parameters that the resulting deconstruction call becomes just as confusing as accessing properties individually — keep deconstruction to a small, meaningful subset of a type's data.
- ❌ Forgetting that reassigning existing variables via deconstruction (`(name, price) = product;` without `var`) requires those variables to ALREADY exist and be assignable — mixing declared and undeclared variables in one deconstruction isn't allowed without `var` on the whole expression.
- ❌ Overusing discards (`_`) to the point that it's unclear what a deconstruction is even extracting — if you're discarding most of the values, maybe you don't need deconstruction for that call at all.

## 💡 Best practices

- ✅ Use deconstruction for tuples returned from methods that logically return "a couple of related values" (count + total, min + max, etc.) — clearer than `.Item1`/`.Item2`.
- ✅ Add a `Deconstruct` method to a class/struct when there's a natural, small, commonly-needed subset of its properties that callers frequently extract together.
- ✅ Use discards (`_`) for values you genuinely don't need, rather than naming a throwaway variable you'll never use.
- ✅ Prefer named tuple elements (`(int Count, decimal Total)`) over unnamed ones (`(int, decimal)`) — makes the deconstructed variable names self-documenting even before deconstruction happens.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                | Answer                                                                                                                    |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| What is deconstruction in C#?                                                           | Unpacking an object's or tuple's values into separate named variables in a single statement                               |
| Are tuples deconstructible automatically?                                               | Yes —`ValueTuple`s support deconstruction out of the box, no extra code needed                                         |
| How do you make a CUSTOM class support deconstruction?                                  | Add a`Deconstruct` method with `out` parameters for each value you want extractable                                   |
| How do you ignore a value during deconstruction?                                        | Use the discard symbol`_` in place of a variable name for that position                                                 |
| Is there a performance cost to using deconstruction over accessing properties directly? | No — it compiles to simple sequential assignments;`ValueTuple` is a struct, so there's no extra heap allocation either |

## 📝 30-second Revision Cheat Sheet

- `var (a, b) = obj;` unpacks values into named local variables in one statement.
- Tuples (`ValueTuple`) are deconstructible automatically; custom classes need an added `Deconstruct(out ...)` method.
- Use `_` (discard) to skip values you don't need.
- Works in `foreach` (e.g., over `Dictionary` key-value pairs) and in pattern-matching `switch` expressions.
- Zero performance cost — purely a readability improvement over accessing properties/tuple items individually.
