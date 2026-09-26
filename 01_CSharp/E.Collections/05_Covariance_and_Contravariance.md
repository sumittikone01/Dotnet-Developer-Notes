
# 🔄 Covariance and Contravariance

## 📌 What is it?

**Covariance** and **Contravariance** describe whether a generic type with an inheritance relationship (e.g. `Animal` → `Dog`) can be **safely substituted** for another generic type in a particular direction — specifically for **interfaces and delegates**, not classes.

```csharp
IEnumerable<Dog> dogs = new List<Dog>();
IEnumerable<Animal> animals = dogs;   // ✅ Valid! — covariance in action

// But NOT with List<T> directly:
// List<Animal> animalList = new List<Dog>();  ❌ Compile error — List<T> is NOT covariant
```

## 🤔 Why do we need it?

Intuitively, since `Dog` **is an** `Animal`, you'd expect `IEnumerable<Dog>` to be usable wherever `IEnumerable<Animal>` is expected — after all, a sequence of dogs is a valid sequence of animals to **read from**. But generics don't automatically support this substitution by default; it must be **explicitly declared** on the interface/delegate, because it's only *safe* in certain directions (explained below).

## 🌍 Real-world analogy

Covariance is like being handed a **basket of only apples** when someone asked for "a basket of fruit" 🍎🍏 — perfectly fine, since you can safely treat any apple as a fruit and only ever take things *out* of the basket. Contravariance is the reverse: being handed a **general-purpose fruit processor** when someone specifically needed an "apple processor" — also fine, because a machine that can handle *any* fruit can certainly handle apples put *into* it.

## ⚙️ Covariance — `out T` (Safe for Output/Return Positions)

**Covariance** allows a **more derived type** to be used where a **less derived (base) type** is expected — this works when `T` is only used in **output positions** (return values), never as an input.

```csharp
public interface IProducer<out T>   // "out" = covariant
{
    T Produce();       // ✅ T in OUTPUT position — allowed
    // void Consume(T item);  ❌ T in INPUT position — NOT allowed with "out"
}
```

**Why it's safe**: If something only ever **gives you** a `T` (never accepts one), then substituting a more specific type is harmless — code expecting `Animal` objects will happily accept the more specific `Dog` objects it receives, since every `Dog` genuinely *is* an `Animal`.

### Real Example: `IEnumerable<out T>`

```csharp
IEnumerable<Dog> dogs = new List<Dog> { new Dog(), new Dog() };
IEnumerable<Animal> animals = dogs;   // ✅ valid — IEnumerable<T> only produces T (via foreach), never consumes it

foreach (Animal a in animals)
    Console.WriteLine(a.GetType().Name);   // safely treats each Dog as an Animal
```

## ⚙️ Contravariance — `in T` (Safe for Input Positions)

**Contravariance** allows a **less derived (base) type** to be used where a **more derived type** is expected — the reverse direction, and it works when `T` is only used in **input positions** (parameters), never as a return value.

```csharp
public interface IConsumer<in T>   // "in" = contravariant
{
    void Consume(T item);   // ✅ T in INPUT position — allowed
    // T Produce();          ❌ T in OUTPUT position — NOT allowed with "in"
}
```

**Why it's safe**: If something only ever **accepts** a `T` (never gives one back), then a version that can handle a **more general** type (`Animal`) can safely be used wherever a **more specific** handler (`Action<Dog>`) was expected — because a "can handle any Animal" processor can certainly handle a `Dog` too.

### Real Example: `Action<in T>`

```csharp
Action<Animal> feedAnimal = (Animal a) => Console.WriteLine($"Feeding {a.GetType().Name}");

Action<Dog> feedDog = feedAnimal;   // ✅ valid — Action<Animal> can safely be substituted for Action<Dog>
feedDog(new Dog());                  // works — the Animal-handling logic handles the Dog just fine
```

## 🖼 Visualizing the Direction

```
Class hierarchy:  Animal  ←  Dog (Dog is more derived)

COVARIANCE (out T)   — matches inheritance direction:
   IEnumerable<Dog>  →  can be used as  →  IEnumerable<Animal>
   (more derived)                           (less derived)

CONTRAVARIANCE (in T) — reverses inheritance direction:
   Action<Animal>  →  can be used as  →  Action<Dog>
   (less derived)                          (more derived)
```

## 📊 Covariance vs Contravariance — Side-by-Side

| Aspect                    | Covariance (`out`)                                       | Contravariance (`in`)                                  |
| ------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| Type parameter position   | Output only (return values)                                | Input only (parameters)                                  |
| Direction of substitution | Derived → Base                                            | Base → Derived                                          |
| Common examples           | `IEnumerable<out T>`, `IEnumerator<out T>`             | `Action<in T>`, `IComparer<in T>`                    |
| Mental model              | "Produces T — giving you something MORE specific is fine" | "Consumes T — accepting something MORE general is fine" |

## 📊 Why `List<T>` is Neither Covariant Nor Contravariant

```csharp
// List<Animal> animalList = new List<Dog>();  ❌ NOT allowed
```

`List<T>` uses `T` in **both** input positions (`Add(T item)`) and output positions (`T this[int index]`) — so neither `in` nor `out` can be safely applied. If this were allowed:

```csharp
List<Animal> animals = new List<Dog>();  // (hypothetically, if this compiled)
animals.Add(new Cat());                   // 💥 would insert a Cat into a List<Dog> — TYPE VIOLATION!
```

This is exactly why mutable generic collections like `List<T>` **cannot** be covariant/contravariant — allowing it would break type safety.

## 🚨 Common Mistakes

- ❌ Expecting `List<Dog>` to be assignable to `List<Animal>` — it isn't; only interfaces/delegates explicitly marked `out`/`in` (like `IEnumerable<T>`) support this.
- ❌ Trying to mark an interface's type parameter `out` when it's used in **both** input and output positions — the compiler will reject this, since it can't guarantee safety in both directions simultaneously.
- ❌ Confusing the **direction**: covariance = derived-to-base (like an ordinary upcast), contravariance = base-to-derived (the "surprising" reverse direction).

## 💡 Best Practices

- Use covariance (`out T`) when designing an interface that **only produces/returns** `T` — enables natural, safe substitutability for read-only scenarios.
- Use contravariance (`in T`) when designing an interface/delegate that **only consumes/accepts** `T` as input — enables safe substitutability for "handler"-style use cases.
- Recognize that most built-in mutable collections (`List<T>`, `Dictionary<K,V>`) are intentionally **not** variant — this is a deliberate type-safety decision, not an oversight.

## 🎤 Interview Questions

1. What's the difference between covariance and contravariance, in terms of which direction substitution is allowed?
2. Why is `IEnumerable<out T>` covariant, but `List<T>` is not?
3. Give a real .NET example of a contravariant interface or delegate.
4. What would go wrong if `List<T>` were allowed to be covariant, hypothetically?

## 📝 30-second Revision Cheat Sheet

- Covariance (`out T`) = `T` used only in **output** positions; derived → base substitution allowed. Example: `IEnumerable<out T>`.
- Contravariance (`in T`) = `T` used only in **input** positions; base → derived substitution allowed. Example: `Action<in T>`.
- `List<T>` is neither — `T` appears in both input (`Add`) and output (`this[i]`) positions, so variance would break type safety.
- Only applies to **interfaces and delegates**, not classes.
