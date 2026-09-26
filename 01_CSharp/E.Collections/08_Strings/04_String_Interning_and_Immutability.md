# 🪞 String Interning & Immutability

## 📌 What is it?

**String Interning** is a memory-optimization technique where the .NET runtime stores **only one copy** of each unique string literal in a special internal pool (the **intern pool**), and reuses that same object whenever the identical string value appears again — rather than creating duplicate objects for identical text.

```csharp
string a = "Hello";
string b = "Hello";

Console.WriteLine(ReferenceEquals(a, b));   // true! — both point to the SAME interned object
```

## 🤔 Why does this matter?

Since strings are **immutable** (see `01_String_Basics.md`), it's completely safe for multiple variables to share the exact same underlying string object — nobody can accidentally mutate it and affect other references. This safety is *precisely* what makes interning possible and beneficial: identical literal strings can be deduplicated in memory without any risk.

## 🌍 Real-world analogy

A **public library's single reference copy of a dictionary** 📖 — instead of printing a private copy of the dictionary for every single person who wants to look up a word, everyone just refers to the **same shared copy**, because nobody is allowed to write in it (immutability = safe to share).

## ⚙️ How Interning Works — Compile-Time Literals

```csharp
string a = "Hello";
string b = "Hello";
string c = "Hel" + "lo";   // compiler combines this into "Hello" AT COMPILE TIME → also interned!

Console.WriteLine(ReferenceEquals(a, b));   // true
Console.WriteLine(ReferenceEquals(a, c));   // true — compiler-evaluated constant, same interned string
```

String literals known at **compile time** are automatically interned by the runtime.

## 🚨 Runtime-Constructed Strings Are NOT Automatically Interned!

```csharp
string a = "Hello";
string b = new string(new char[] { 'H', 'e', 'l', 'l', 'o' });   // built at RUNTIME

Console.WriteLine(a == b);                    // true — value equality (strings always compare by value)
Console.WriteLine(ReferenceEquals(a, b));     // false! — DIFFERENT objects in memory, despite equal content
```

This is the classic interview gotcha: **`==` still returns `true`** (value comparison, as covered in `01_String_Basics.md`), but `ReferenceEquals()` reveals they are genuinely **different objects** in memory — because `b` was constructed dynamically at runtime, not known at compile time, so it wasn't automatically interned.

## 💻 Manually Interning a Runtime String

```csharp
string a = "Hello";
string b = new string(new char[] { 'H', 'e', 'l', 'l', 'o' });

string internedB = string.Intern(b);

Console.WriteLine(ReferenceEquals(a, internedB));   // true! — now explicitly forced into the intern pool
```

`string.Intern()` manually adds a runtime-created string to the shared pool (or returns the existing pooled reference if an identical value is already there).

## 📊 Why Immutability is the Enabling Foundation

```
If strings were MUTABLE and interned:

string a = "Hello";
string b = "Hello";   // same object as "a", due to interning

a[0] = 'J';   // if this were allowed...
// b would ALSO silently become "Jello" — because they're the SAME object!
// This would be a catastrophic, unpredictable bug.
```

This is *exactly why* interning is only safe **because** strings are immutable — nobody can ever accidentally corrupt a shared string for everyone else referencing it.

## 🖼 Memory Diagram

```
Intern Pool:
  "Hello" ──┬── referenced by variable a
            └── referenced by variable b
          
new string(...) → "Hello" (separate copy, NOT in the pool by default)
                    └── referenced by variable c (different object!)
```

## 🚨 Common Mistakes

- ❌ Using `ReferenceEquals()` (or `object.ReferenceEquals`) to check string **equality** in application logic — you almost always want value equality (`==` or `.Equals()`), not reference equality; `ReferenceEquals` is really only useful for understanding/debugging interning behavior itself.
- ❌ Assuming ALL identical strings are automatically the same object — only compile-time literal strings (and manually interned ones) are guaranteed to be pooled; runtime-constructed strings are not, by default.
- ❌ Over-relying on manual `string.Intern()` calls for performance — interning has its own overhead (checking/adding to the pool) and is rarely necessary in typical application code; it mainly matters for very specific memory-optimization scenarios with huge volumes of repeated string data.
- ❌ Confusing string interning with `StringBuilder`'s mutability — they're unrelated concepts; interning is about deduplicating *immutable* string objects in memory, not about making strings editable.

## 💡 Best Practices

- Always use `==` or `.Equals()` for string equality checks in real code — never rely on `ReferenceEquals()` for correctness.
- Understand interning as a **memory optimization detail** the runtime handles mostly automatically for literals — you rarely need to manage it manually.
- Only reach for `string.Intern()` in genuinely memory-constrained scenarios with massive amounts of repeated string data (e.g. parsing huge datasets with many duplicate string values) — measure first before optimizing.

## 🎤 Interview Questions

1. What is string interning, and why is it only safe because strings are immutable?
2. Why does `ReferenceEquals()` return `false` for two runtime-constructed strings with identical content, while `==` returns `true`?
3. Are all string literals automatically interned? What about strings built at runtime?
4. When, if ever, would you manually call `string.Intern()` in production code?

## 📝 30-second Revision Cheat Sheet

- String interning = runtime stores ONE shared copy per unique literal string value, reused across the app.
- Only **safe** because strings are **immutable** — no risk of one reference corrupting another's shared data.
- Compile-time literals are auto-interned; runtime-constructed strings (`new string(...)`) are NOT, by default.
- `==` always compares by value; `ReferenceEquals()` reveals whether two strings are the literal same object.
- `string.Intern()` manually forces a runtime string into the shared pool — rarely needed in practice.
