
# 05_Custom_Validation_Attributes

> **Custom Validation Attributes** = your own `[Attribute]`-based validation rules, built by inheriting `ValidationAttribute`, for business rules that ASP.NET Core's built-in attributes (`[Required]`, `[Range]`, `[EmailAddress]`, etc.) can't express.

> Part of **H.Model_Handling**. Builds on the built-in Data Annotations you already use on ViewModels — this chapter is what to do once a validation rule is specific to YOUR business logic and no built-in attribute fits.

## 📌 What is it?

Built-in attributes cover generic cases:

```csharp
public class ProductViewModel
{
    [Required]
    public string Name { get; set; } = "";

    [Range(0.01, 100000)]
    public decimal Price { get; set; }
}
```

But business rules are often more specific — "the discount price must be LESS than the regular price," "this date must be in the future," "this SKU must match our company's exact format." For these, you write a **custom validation attribute**.

## 🤔 Why do we need it?

| Problem with only built-in attributes                                                            | How custom attributes help                                                        |
| ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Built-ins only validate ONE property in isolation                                                | A custom attribute can compare TWO properties (e.g.,`EndDate > StartDate`)      |
| Business-specific formats (SKU codes, internal ID patterns) aren't covered by generic attributes | Encapsulate the exact company rule in one reusable attribute                      |
| Repeating the same custom`if` check in every Controller action that needs it                   | Write the rule ONCE as an attribute, reuse it everywhere via`[MyRule]`          |
| Validation logic scattered across Controllers                                                    | Keeps validation declarative and colocated with the ViewModel property it governs |

## 🌍 Real-world analogy

Built-in attributes are like a **generic bouncer's checklist** ("must be 18+", "must have ID") that works at any club. A custom validation attribute is like a **venue-specific rule** ("must be on tonight's guest list") — a rule unique to YOUR business that no generic checklist would know to check.

## ⚙️ Internal working — inheriting `ValidationAttribute`

```csharp
public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is DateTime date && date <= DateTime.Now)
        {
            return new ValidationResult(ErrorMessage ?? "Date must be in the future.");
        }
        return ValidationResult.Success; // null return also means "valid"
    }
}
```

```
Model binding pipeline:

1. Request arrives → ASP.NET Core binds JSON/form data to the ViewModel
2. For EACH property, the framework checks its validation attributes
3. Your custom attribute's IsValid() method runs
4. Returns ValidationResult.Success → property passes
   Returns a ValidationResult with a message → added to ModelState.IsValid = false
5. Controller checks ModelState.IsValid (see 10_Model_Validation_in_Web_API.md)
```

## 📊 Two Approaches — Single-Property vs Cross-Property Validation

| Approach                                | Base class                                                 | Use case                                                                      |
| --------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Single-property**               | `ValidationAttribute` applied directly to ONE property   | "This date must be in the future," "This string must match a SKU pattern"     |
| **Cross-property (whole-object)** | `IValidatableObject` implemented on the ViewModel itself | "EndDate must be after StartDate" — needs to see MULTIPLE properties at once |

```csharp
// Cross-property validation — implement IValidatableObject on the model itself
public class PromotionViewModel : IValidatableObject
{
    public DateTime StartDate { get; set; }
    public DateTime EndDate { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (EndDate <= StartDate)
        {
            yield return new ValidationResult(
                "End date must be after the start date.",
                new[] { nameof(EndDate) }); // associates the error with the EndDate field specifically
        }
    }
}
```

## 💻 Code examples

### Basic — a simple custom attribute (single property)

```csharp
public class SkuFormatAttribute : ValidationAttribute
{
    private static readonly Regex SkuPattern = new(@"^[A-Z]{3}-\d{5}$"); // e.g., "ABC-12345"

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (value is string sku && !SkuPattern.IsMatch(sku))
        {
            return new ValidationResult(ErrorMessage ?? "SKU must be in the format ABC-12345.");
        }
        return ValidationResult.Success;
    }
}

public class ProductViewModel
{
    [Required]
    [SkuFormat(ErrorMessage = "Invalid SKU format.")]
    public string Sku { get; set; } = "";
}
```

### Intermediate — a custom attribute that compares TWO properties (via reflection)

```csharp
public class GreaterThanAttribute : ValidationAttribute
{
    private readonly string _comparisonProperty;
    public GreaterThanAttribute(string comparisonProperty) => _comparisonProperty = comparisonProperty;

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        var otherProperty = validationContext.ObjectType.GetProperty(_comparisonProperty);
        var otherValue = otherProperty?.GetValue(validationContext.ObjectInstance);

        if (value is IComparable comparable && otherValue is IComparable &&
            comparable.CompareTo(otherValue) <= 0)
        {
            return new ValidationResult(ErrorMessage ?? $"Must be greater than {_comparisonProperty}.");
        }
        return ValidationResult.Success;
    }
}

public class PromotionViewModel
{
    public decimal RegularPrice { get; set; }

    [GreaterThan(nameof(RegularPrice), ErrorMessage = "Regular price must be greater than the discount price.")]
    public decimal DiscountPrice { get; set; } // wait — actually validate RegularPrice > DiscountPrice
}
```

### Practical — using it in your team's Controller → BAL flow

```csharp
[HttpPost]
public IActionResult CreateProduct(ProductViewModel model)
{
    if (!ModelState.IsValid) // custom attribute errors show up here automatically, same as built-ins
    {
        return BadRequest(ModelState);
    }

    var created = _productService.CreateProduct(model); // Controller → BAL → DAL → Stored Procedure
    return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
}
```

## ⚡ Performance considerations

- Validation attributes run on EVERY request that binds the model — keep `IsValid` logic fast; avoid expensive operations (DB calls, external API calls) inside a validation attribute. If a rule genuinely needs a DB check (e.g., "this email isn't already registered"), do that check explicitly in the BAL/service layer instead, not inside a `ValidationAttribute`.
- Reflection-based cross-property attributes (like the `GreaterThan` example) have a small overhead per validation due to `GetProperty`/`GetValue` calls — negligible for typical form sizes, but worth knowing if validating very large or very frequently-submitted models.

## 🚨 Common mistakes

- ❌ Putting database or network calls inside a `ValidationAttribute`'s `IsValid` method — validation attributes should be fast, synchronous, and side-effect-free; business rules requiring I/O belong in the BAL.
- ❌ Forgetting to associate a cross-property error with the SPECIFIC field via `IValidatableObject`'s `memberNames` parameter — without it, the error shows generically instead of highlighting the actual problematic field in the UI (e.g., a Kendo form).
- ❌ Not providing a clear, specific `ErrorMessage` — falling back to a generic framework default that doesn't help the end user understand what to fix.
- ❌ Writing a custom attribute for a rule that a built-in attribute (`[Range]`, `[RegularExpression]`, `[Compare]`) already covers — check the built-in options first.

## 💡 Best practices

- ✅ Use a custom `ValidationAttribute` for single-property, reusable business rules; use `IValidatableObject` for rules that compare multiple properties on the same object.
- ✅ Keep validation attribute logic fast and side-effect-free — no DB/network calls.
- ✅ Always provide clear, specific `ErrorMessage` text tailored to the actual business rule.
- ✅ Check for an existing built-in attribute (`[Compare]`, `[RegularExpression]`, `[Range]`) before writing a custom one — don't reinvent what's already provided.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                              | Answer                                                                                                                                  |
| ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| How do you create a custom single-property validation attribute?                      | Inherit from`ValidationAttribute` and override `IsValid(object value, ValidationContext context)`                                   |
| When would you use`IValidatableObject` instead of a custom `ValidationAttribute`? | When a validation rule needs to compare multiple properties on the same object (e.g., EndDate must be after StartDate)                  |
| Why shouldn't a validation attribute make a database call?                            | Validation attributes should be fast and side-effect-free; DB/network calls belong in the business logic layer, not in model validation |
| What does returning`ValidationResult.Success` (or null) from `IsValid` mean?      | The value is valid — no error is added to`ModelState`                                                                                |
| How do you associate a cross-property validation error with a specific field?         | Pass the field name(s) via the`memberNames` parameter when constructing the `ValidationResult` in `IValidatableObject.Validate`   |

## 📝 30-second Revision Cheat Sheet

- Custom validation = inherit `ValidationAttribute`, override `IsValid()`, for single-property business rules not covered by built-ins.
- Use `IValidatableObject` on the model itself for cross-property rules (e.g., EndDate > StartDate).
- Keep validation logic fast and side-effect-free — no DB/network calls inside `IsValid`.
- Custom attribute errors flow into `ModelState.IsValid` automatically, same as built-in attributes.
- Always give clear, specific `ErrorMessage` text and associate cross-property errors with the right field.
