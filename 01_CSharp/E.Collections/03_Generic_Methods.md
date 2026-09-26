
# 🔧 Generic Methods

## 📌 What is it?

A **Generic Method** is a method with its own type parameter(s), independent of whether the containing class is generic — letting a single method definition work across multiple types.

```csharp
public T Max<T>(T a, T b) where T : IComparable<T>
{
    return a.CompareTo(b) > 0 ? a : b;
}

int biggerInt = Max(5, 10);              // T inferred as int → 10
string biggerStr = Max("apple", "banana"); // T inferred as string → "banana"
```

## 🤔 Why do we need it?

Sometimes you need type-generic behavior in just **one method**, without making the entire containing class generic. Generic methods let you scope the flexibility exactly where it's needed — a regular (non-generic) class can still contain individual generic methods.

```csharp
// A perfectly ordinary, non-generic class...
public class MathHelper
{
    // ...can still contain a generic method
    public T Max<T>(T a, T b) where T : IComparable<T>
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
}
```

## 🌍 Real-world analogy

A **universal translator device** 🎙️ that works with **any spoken language** you feed into it, even though the device itself (the class) isn't "a French device" or "a Spanish device" — the flexibility lives in *how it processes input*, not in the device's fixed identity.

## ⚙️ Type Inference — Usually No Need to Specify `<T>` Explicitly

```csharp
int result = Max(5, 10);              // compiler infers T = int automatically
string result2 = Max("a", "b");        // compiler infers T = string automatically

// You CAN specify explicitly if needed/clearer:
int result3 = Max<int>(5, 10);
```

The compiler looks at the **arguments you pass** and figures out what `T` must be — you rarely need to write `<T>` explicitly at the call site.

## 💻 Practical Example: A Generic Swap Method

```csharp
public void Swap<T>(ref T a, ref T b)
{
    T temp = a;
    a = b;
    b = temp;
}

int x = 1, y = 2;
Swap(ref x, ref y);
Console.WriteLine($"{x}, {y}");   // "2, 1"

string s1 = "Hello", s2 = "World";
Swap(ref s1, ref s2);
Console.WriteLine($"{s1}, {s2}"); // "World, Hello"
```

One `Swap<T>` method now works for `int`, `string`, or literally any type — no duplicated `SwapInt()`, `SwapString()` methods needed.

## 💻 Generic Methods in LINQ (You've Already Been Using These!)

```csharp
List<int> numbers = new List<int> { 1, 2, 3 };
List<string> names = new List<string> { "a", "b" };

// Where<T>, Select<T, TResult> — both generic methods, working across ANY element type
var evens = numbers.Where(n => n % 2 == 0);
var upper = names.Select(n => n.ToUpper());
```

Every single LINQ method is a generic extension method — this is *why* the same `.Where()`/`.Select()` syntax works identically whether you're filtering a list of `int`, `string`, or a custom class.

## 📊 Generic Method in a Non-Generic Class vs Generic Class Method — Comparison

| Scenario                                                                                         | Example                                                                                                             |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| Generic method in a**non-generic** class                                                   | `class Helper { T Max<T>(T a, T b) ... }` — flexibility scoped to just this method                               |
| Method inside an**already-generic** class                                                  | `class Box<T> { T Get() ... }` — `T` here comes from the *class's* type parameter, shared by all its members |
| Generic method**inside** a generic class, with its **own** additional type parameter | `class Box<T> { TResult Convert<TResult>(Func<T, TResult> converter) ... }` — combines both                      |

```csharp
public class Box<T>
{
    public T Value { get; set; }

    // This method introduces its OWN separate type parameter, TResult,
    // independent of the class's T
    public TResult ConvertTo<TResult>(Func<T, TResult> converter)
    {
        return converter(Value);
    }
}

var intBox = new Box<int> { Value = 42 };
string asString = intBox.ConvertTo(v => v.ToString());   // T = int, TResult = string
```

## 🚨 Common Mistakes

- ❌ Making an entire class generic when only **one method** actually needs the flexibility — unnecessarily generic-izes every member of the class instead of scoping it to just the method that needs it.
- ❌ Explicitly specifying `<T>` at every call site out of habit — usually unnecessary; let the compiler infer it from the arguments unless there's genuine ambiguity.
- ❌ Forgetting that a generic method's type parameter is completely independent from its containing class's type parameter (if the class is also generic) — they can be constrained differently and don't have to match.

## 💡 Best Practices

- Scope genericity to the smallest unit that needs it — a single generic method in an otherwise-plain class, if that's all that's required.
- Let the compiler infer type parameters from arguments — only specify `<T>` explicitly when the compiler genuinely can't infer it (e.g. when the type parameter doesn't appear in any parameter, only in the return type).
- Apply constraints (`where T : ...`) to generic methods just as you would to generic classes, whenever the method body needs specific capabilities from `T` (see `04_Generic_Constraints.md`).

## 🎤 Interview Questions

1. Can a non-generic class contain a generic method? Give an example of why you might want that.
2. How does the compiler infer the type parameter for a call like `Max(5, 10)` without you writing `Max<int>(5, 10)`?
3. Give a widely-used example of generic methods you've likely used without realizing it.
4. If a generic class has type parameter `T` and one of its methods introduces its own `TResult`, are these two type parameters related in any way?

## 📝 30-second Revision Cheat Sheet

- Generic method = `T Max<T>(T a, T b)` — flexibility scoped to just one method, class itself can be non-generic.
- Type inference usually means you don't need to write `<T>` explicitly at the call site.
- A generic method inside a generic class can introduce its **own separate** type parameter (e.g. `TResult`).
- LINQ's `.Where()`, `.Select()`, etc. are all generic methods — you use this pattern constantly already.
