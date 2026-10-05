# 06_Returning_JSON

> How ASP.NET Core actually SERIALIZES your C# objects into the JSON that gets sent back to the client — the default `System.Text.Json` serializer, how to control its output (casing, nulls, enums), and how it interacts with the result helpers (`Ok`, etc.) you've been using since `01_API_Controllers.md`.

> Continues **K.Web_API**. Every `Ok(product)` call in earlier chapters relied on this serialization happening automatically — this chapter is what's actually going on, and how to shape it.

## 📌 What is it?

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    Product product = _service.GetProductById(id);
    return Ok(product); // <-- 'product' gets serialized to JSON HERE, automatically
}
```

```json
// What the client actually receives:
{
  "id": 5,
  "name": "Widget",
  "price": 49.99,
  "isActive": true
}
```

By default, ASP.NET Core uses **`System.Text.Json`** (built into .NET) to perform this conversion — no extra configuration needed for basic cases.

## 🤔 Why do we need to understand the serializer's behavior?

| Situation                                                                                   | Why it matters                                                                   |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Your C# property is`IsActive` but the frontend (Kendo UI/JavaScript) expects `isActive` | Default casing conversion handles this — but it's useful to know it's happening |
| A`null` field should be OMITTED from the JSON entirely, not sent as `"field": null`     | Needs explicit serializer configuration                                          |
| An`enum` should serialize as its STRING name, not its underlying number                   | Needs an explicit converter                                                      |
| Circular references between objects (e.g., Product → Category → Products...)              | Causes a serialization exception unless handled                                  |

## 🌍 Real-world analogy

The JSON serializer is like a **translator converting your spoken thoughts (C# objects) into a foreign language (JSON text)** for someone who doesn't speak C#. By default, the translator uses sensible, standard phrasing (camelCase property names) — but you CAN give the translator specific instructions ("always use formal titles," "never mention certain topics") to control exactly how the translation comes out.

## 📊 Default `System.Text.Json` Behaviors

| Behavior                                       | Default                                                                     |
| ---------------------------------------------- | --------------------------------------------------------------------------- |
| Property naming                                | Converts C#`PascalCase` → JSON `camelCase` automatically               |
| `null` values                                | INCLUDED in the output by default (`"field": null`)                       |
| Enums                                          | Serialized as their underlying NUMBER by default (e.g.,`0`, `1`, `2`) |
| Case sensitivity (deserializing incoming JSON) | Case-INSENSITIVE by default when BINDING incoming requests                  |
| Circular references                            | THROWS an exception by default (needs explicit handling)                    |

## ⚙️ Internal working — configuring global JSON options

```csharp
// Program.cs
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        // Omit null values from the output entirely
        options.JsonSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull;

        // Serialize enums as their STRING names, not numbers
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());

        // Handle circular references gracefully instead of throwing
        options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    });
```

```
BEFORE (default):                           AFTER (configured):

{                                            {
  "id": 5,                                    "id": 5,
  "name": "Widget",                            "name": "Widget",
  "description": null,     ← included         (description OMITTED — was null)
  "status": 1               ← numeric enum     "status": "Active"   ← string enum
}                                            }
```

## 📊 Per-Property Control with Attributes

| Attribute                                                         | Effect                                                      |
| ----------------------------------------------------------------- | ----------------------------------------------------------- |
| `[JsonPropertyName("product_name")]`                            | Overrides the JSON property name for this specific property |
| `[JsonIgnore]`                                                  | Excludes this property from serialization entirely          |
| `[JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]` | Excludes ONLY if the value is null                          |
| `[JsonPropertyOrder(1)]`                                        | Controls the ORDER properties appear in the output JSON     |

```csharp
public class ProductDto
{
    public int Id { get; set; }

    [JsonPropertyName("product_name")] // overrides camelCase default for this one property
    public string Name { get; set; } = "";

    [JsonIgnore] // NEVER included in the JSON, regardless of value
    public string InternalNotes { get; set; } = "";

    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
    public decimal? DiscountPrice { get; set; } // omitted ONLY if null/default
}
```

## 💻 Code examples

### Basic — the default, automatic JSON serialization behind `Ok(...)`

```csharp
public class ProductDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public decimal Price { get; set; }
}

[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var dto = new ProductDto { Id = 5, Name = "Widget", Price = 49.99m };
    return Ok(dto); // automatically serialized — client sees: {"id":5,"name":"Widget","price":49.99}
}
```

### Intermediate — serializing a list for a Kendo Grid

```csharp
[HttpGet]
public IActionResult GetGridData([FromQuery] DataSourceRequest request)
{
    var products = _service.GetAllProducts();
    var result = products.ToDataSourceResult(request); // a Kendo-specific wrapper object

    return Ok(result);
    // Serializes to the exact shape Kendo's DataSource expects:
    // { "Data": [...], "Total": 42, "AggregateResults": null, "Errors": null }
}
```

### Practical — manually controlling serialization for a specific endpoint (bypassing global config)

```csharp
[HttpGet("export")]
public IActionResult ExportAsJson()
{
    var products = _service.GetAllProducts();

    var options = new JsonSerializerOptions
    {
        WriteIndented = true, // pretty-printed, for a human-readable export file
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };

    string json = JsonSerializer.Serialize(products, options);
    return File(Encoding.UTF8.GetBytes(json), "application/json", "products-export.json");
}
```

### Practical — handling a self-referencing object graph (avoiding circular reference errors)

```csharp
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = "";

    [JsonIgnore] // breaks the cycle explicitly — Category → Products → Category → ... would otherwise loop forever
    public List<Product> Products { get; set; } = new();
}

public class Product
{
    public int Id { get; set; }
    public Category Category { get; set; } = new(); // fine — only ONE direction is serialized
}
```

## ⚡ Performance considerations

- `System.Text.Json` is significantly faster and more memory-efficient than the older `Newtonsoft.Json` (which used to be the ASP.NET Core default) — generally no reason to switch to Newtonsoft unless you need a specific feature it has that `System.Text.Json` lacks.
- `WriteIndented = true` (pretty-printing) produces larger payloads and is slightly slower to generate — use it only for human-facing exports/debugging, never for regular API traffic consumed by code (like your Kendo Grid's AJAX calls).
- Avoid serializing deeply nested object graphs with many navigation properties unless genuinely needed — large, deeply-nested JSON payloads cost bandwidth and client-side parsing time; prefer flatter DTOs shaped for exactly what the client needs (ties back to `12_System_Design/I.API_Design/05_GraphQL_Basics.md`'s over-fetching discussion).

## 🚨 Common mistakes

- ❌ Returning raw domain entities with circular references (e.g., `Product.Category.Products...`) without `[JsonIgnore]` or `ReferenceHandler.IgnoreCycles` — causes a serialization exception at runtime.
- ❌ Assuming enum values will serialize as readable strings by default — they serialize as numbers unless `JsonStringEnumConverter` is explicitly configured.
- ❌ Forgetting that incoming JSON property name matching is case-INSENSITIVE by default, which can mask subtle naming mismatches between client and server that would otherwise be caught.
- ❌ Manually building JSON strings via string concatenation instead of using the serializer — error-prone and loses proper escaping/formatting.

## 💡 Best practices

- ✅ Return DTOs specifically shaped for the client, not raw database entities — avoids circular reference issues AND controls exactly what's exposed (recap from `01_API_Controllers.md`).
- ✅ Configure global `JsonSerializerOptions` once in `Program.cs` for consistent behavior (null handling, enum serialization) across the whole API, rather than configuring per-endpoint.
- ✅ Use `[JsonPropertyName]`/`[JsonIgnore]` for the rare cases where a SPECIFIC property needs different behavior than the global default.
- ✅ Reserve `WriteIndented = true` for human-facing exports/debugging only — never for regular machine-consumed API responses.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                       | Answer                                                                                                                                                            |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the default JSON serializer in modern ASP.NET Core?                     | `System.Text.Json`, built into .NET — faster and more memory-efficient than the older Newtonsoft.Json                                                          |
| Does`System.Text.Json` include `null` properties in the output by default? | Yes — you must explicitly configure`DefaultIgnoreCondition` to omit them                                                                                       |
| How do enums serialize by default, and how do you change that?                 | As their underlying numeric value by default; add a`JsonStringEnumConverter` to serialize them as string names instead                                          |
| What causes a circular reference serialization error, and how do you fix it?   | Objects referencing each other in a cycle (e.g., Product → Category → Products); fix with`[JsonIgnore]` on one direction or `ReferenceHandler.IgnoreCycles` |
| Why return DTOs instead of raw entities when returning JSON?                   | Avoids circular reference issues, controls exactly what's exposed, and keeps the API contract independent of the database schema                                  |

## 📝 30-second Revision Cheat Sheet

- `System.Text.Json` is the default serializer — converts PascalCase C# properties to camelCase JSON automatically.
- Nulls are included by default; enums serialize as numbers by default — both configurable via `AddJsonOptions`.
- `[JsonPropertyName]`, `[JsonIgnore]` give per-property control when the global config isn't enough.
- Circular references throw by default — break the cycle with `[JsonIgnore]` or `ReferenceHandler.IgnoreCycles`.
- Return DTOs, not raw entities, for cleaner, cycle-free, intentionally-shaped JSON output.
