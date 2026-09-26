# 🔤 String Basics

## 📌 What is it?

`string` in C# is a sequence of characters (`char`) representing text — but with one crucial, easy-to-miss characteristic: **strings are immutable**. Once created, a string's content can never be changed — every "modification" actually creates a brand-new string.

```csharp
string name = "Rohit";
name = name + " Kumar";   // this does NOT modify the original — it creates a NEW string object
```

## 🤔 Why is immutability the core concept here?

```csharp
string a = "Hello";
string b = a;
a = a + " World";

Console.WriteLine(a);   // "Hello World"
Console.WriteLine(b);   // "Hello" — unaffected! "a" now points to a completely NEW string object
```

When you "change" `a`, you're not editing the original characters in memory — you're creating an entirely new string object and repointing `a` to it. The original `"Hello"` string still exists unchanged (and `b` still points to it) until garbage collected.

## 🌍 Real-world analogy

A **printed book** 📖 — you can't edit a word directly on a printed page. If you want different text, you print an **entirely new book**. `string` behaves the same way: any "edit" is really a whole new object, not a modification of the existing one.

## ⚙️ Key Characteristics

| Property                     | Detail                                                                                                         |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Type**               | Reference type (but behaves specially — see below)                                                            |
| **Mutability**         | Immutable — cannot be changed after creation                                                                  |
| **Underlying storage** | Array of`char` (UTF-16 code units) internally                                                                |
| **Indexing**           | `str[0]` gives the first character — read-only access                                                       |
| **Comparison**         | `==` compares by **value** for strings (special-cased), not by reference (unlike most reference types) |

```csharp
string s = "Hello";
char firstChar = s[0];        // 'H' — read-only indexed access
// s[0] = 'J';                 ❌ Compile error — strings are immutable, can't assign to an index

int length = s.Length;         // 5
```

## 🧠 String Equality — `==` Works by Value (Special Case!)

```csharp
string a = "Hello";
string b = "Hello";

Console.WriteLine(a == b);         // true — value comparison
Console.WriteLine(a.Equals(b));    // true — same result

object oa = a;
object ob = b;
Console.WriteLine(oa == ob);       // true too! — string overloads == specifically for value equality
```

This is unusual: most reference types compare by **reference** with `==` unless you override it. `string` is a special-cased exception — its `==` operator is overloaded to compare **content**, matching how most people intuitively expect string comparison to behave.

## 💻 String Concatenation

```csharp
string first = "Hello";
string second = "World";

string combined1 = first + " " + second;              // operator concatenation
string combined2 = string.Concat(first, " ", second);  // explicit method
string combined3 = $"{first} {second}";                // string interpolation ⭐ (most readable, most common in modern code)
```

## 🚨 Common Mistakes

- ❌ Concatenating strings in a **loop** using `+` — since strings are immutable, each `+=` creates an entirely **new** string object every iteration, copying all previous characters again — this becomes O(n²) for building a large string. Use `StringBuilder` instead for this case (see `03_StringBuilder.md`).

```csharp
// ❌ Inefficient for large loops
string result = "";
for (int i = 0; i < 10000; i++)
    result += i;   // creates a new string object EVERY iteration
```

- ❌ Assuming `string[0] = 'x'` works — it doesn't compile; strings can't be modified in place at all.
- ❌ Forgetting that `string` comparison with `==` is value-based (a helpful exception), while comparing most **other** reference types with `==` compares by reference unless explicitly overridden — easy to misapply this assumption to custom classes.

## 💡 Best Practices

- Use string interpolation (`$"{...}"`) for readable, concise string building — the modern preferred style.
- Use `StringBuilder` instead of repeated `+`/`+=` concatenation inside loops.
- Remember `.Equals()` and `==` behave the same for strings (value comparison) — either is fine, but `==` is more common/readable.
- Understand immutability deeply — it underlies many other string behaviors (thread-safety of strings, string interning — see `04_String_Interning_and_Immutability.md`).

## 🎤 Interview Questions

1. Why is `string` considered immutable, and what actually happens when you "modify" a string?
2. Why does `==` compare strings by value, when most reference types compare by reference?
3. Why is repeated string concatenation in a loop inefficient, and what should you use instead?
4. What's the underlying storage type behind a C# `string`?

## 📝 30-second Revision Cheat Sheet

- `string` = immutable sequence of characters — every "modification" creates a new object.
- `==` is special-cased to compare strings **by value**, unlike most reference types.
- Avoid `+`/`+=` concatenation in loops — O(n²) due to repeated new-string creation; use `StringBuilder`.
- No in-place index assignment (`str[0] = 'x'` doesn't compile).
- Prefer string interpolation (`$"..."`) for building strings.
