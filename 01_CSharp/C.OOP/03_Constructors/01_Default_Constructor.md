
# 🏗️ Default Constructor

## 📌 What is it?

> A **constructor** is a special method that runs automatically when an object is created, used to initialize its state. A **default constructor** takes no parameters.

---

## 🧠 Intuition

```csharp
public class Product
{
    public string Name;

    public Product()   // DEFAULT CONSTRUCTOR — no parameters
    {
        Name = "Unnamed Product";
    }
}

var p = new Product();   // triggers the default constructor
Console.WriteLine(p.Name);  // "Unnamed Product"
```

---

## ⚙️ Key Facts

| Fact                                  | Explanation                                                                                                                                  |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Auto-generated                        | If you write NO constructors at all, C# auto-generates a public parameterless one                                                            |
| Disappears if you add any constructor | Once you define ANY constructor (even parameterized), the auto-generated default one is gone — you must write it explicitly if still needed |
| Same name as class                    | Constructors always share the class's name, no return type (not even`void`)                                                                |
| Runs once per object                  | Called automatically at`new ClassName()`                                                                                                   |

---

## 💻 Code Examples

**Basic — implicit default constructor:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;
    // No constructor written — compiler provides an implicit public parameterless one
}

var p = new Product();   // works — Name = null, Price = 0 (default values)
```

**Intermediate — explicit default constructor with initialization:**

```csharp
public class Product
{
    public string Name;
    public decimal Price;

    public Product()   // explicit default constructor
    {
        Name = "Unnamed";
        Price = 0;
    }
}
```

**Practical — losing the default constructor by adding a parameterized one:**

```csharp
public class Product
{
    public string Name;

    public Product(string name)   // parameterized constructor added
    {
        Name = name;
    }
}

// var p = new Product();   ❌ Compile error! No parameterless constructor exists anymore
var p = new Product("Laptop");   // ✅ must use the parameterized one
```

---

## 🚨 Common Mistakes

- ❌ Assuming the implicit default constructor still exists after adding a custom parameterized constructor — it doesn't, unless you explicitly define one too
- ❌ Forgetting some frameworks (like model binding, deserialization) may require a parameterless constructor to exist

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                                             |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------- |
| What is a default constructor?                       | A parameterless constructor, either auto-generated or explicitly written              |
| Does C# always provide a default constructor?        | Only if you don't define ANY constructor yourself                                     |
| What happens if you add a parameterized constructor? | The compiler-generated default constructor disappears unless you write one explicitly |

---

## 📝 30-Second Revision Cheat Sheet

- Default constructor = parameterless, initializes the object
- Auto-generated ONLY if you write no constructors at all
- Adding any custom constructor removes the implicit default — write it explicitly if still needed
