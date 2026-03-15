# JSON Structure

---

## 📌 What is JSON?

**JSON (JavaScript Object Notation)** is a lightweight, text-based data format used to store and exchange data between a server and a client.

* It is **language-independent** — every language (C#, Python, Java, JS) can read/write it
* It is **human-readable** — you can open it in Notepad and understand it
* It is the **standard format** for Web API responses and AJAX communication
* It replaced XML in most modern web applications because it is smaller and simpler

> **Simple analogy:** JSON is like a structured text message that both the browser and the server can understand perfectly.

---

## 📐 JSON vs XML — Why JSON Won

```
XML (old way) — verbose, hard to read
─────────────────────────────────────
<employee>
  <id>1</id>
  <name>Alice</name>
  <salary>50000</salary>
</employee>

JSON (modern way) — clean, compact
────────────────────────────────────
{
  "id": 1,
  "name": "Alice",
  "salary": 50000
}
```

| Feature             | JSON                 | XML                   |
| ------------------- | -------------------- | --------------------- |
| Size                | Smaller              | Larger (tag overhead) |
| Readability         | Easier               | Harder                |
| Parsing speed       | Faster               | Slower                |
| Native JS support   | Yes (`JSON.parse`) | No (needs DOMParser)  |
| Comments allowed    | ❌ No                | ✅ Yes                |
| Supports attributes | ❌ No                | ✅ Yes                |
| Used in REST APIs   | ✅ Standard          | Rare                  |

---

## 🔑 The 6 JSON Data Types

JSON supports exactly  **6 data types** . Nothing more.

```
┌─────────────────────────────────────────────────────────┐
│               JSON DATA TYPES                           │
├──────────────┬────────────────┬────────────────────────┤
│ Type         │ Example        │ Notes                  │
├──────────────┼────────────────┼────────────────────────┤
│ String       │ "Alice"        │ MUST use double quotes │
│ Number       │ 42  or  3.14   │ int or float, no quotes│
│ Boolean      │ true / false   │ lowercase only         │
│ null         │ null           │ lowercase only         │
│ Object       │ { "k": "v" }   │ key-value pairs        │
│ Array        │ [1, 2, 3]      │ ordered list           │
└──────────────┴────────────────┴────────────────────────┘
```

```json
{
  "name":       "Alice",         ← String  (double quotes required)
  "age":        30,              ← Number  (no quotes)
  "salary":     50000.50,        ← Number  (decimals OK)
  "isActive":   true,            ← Boolean (lowercase)
  "middleName": null,            ← null    (lowercase)
  "address":    { "city": "NY" },← Object
  "tags":       ["dev", "lead"]  ← Array
}
```

---

## ⚠️ JSON Syntax Rules — The Strict Ones

JSON is  **very strict** . One mistake = the entire JSON is invalid.

### Rule 1 — Keys MUST be double-quoted strings

```json
✅ { "name": "Alice" }
❌ { name: "Alice" }       ← no quotes on key = INVALID
❌ { 'name': 'Alice' }     ← single quotes = INVALID
```

### Rule 2 — Strings MUST use double quotes

```json
✅ { "city": "New York" }
❌ { "city": 'New York' }   ← single quotes = INVALID
```

### Rule 3 — No trailing commas

```json
✅ { "a": 1, "b": 2 }
❌ { "a": 1, "b": 2, }     ← trailing comma = INVALID
```

### Rule 4 — No comments

```json
❌ { "name": "Alice" }  // this is Alice   ← comments NOT allowed
```

### Rule 5 — Booleans and null are lowercase

```json
✅ { "active": true,  "data": null }
❌ { "active": True,  "data": NULL }   ← capitals = INVALID
```

### Rule 6 — Numbers have no special formatting

```json
✅ { "price": 1234.56 }
❌ { "price": $1,234.56 }   ← currency symbols/commas = INVALID
❌ { "price": 1_000 }       ← underscore = INVALID
```

---

## 🗂️ JSON Document Structure

A valid JSON document is **either** an object `{ }` or an array `[ ]` at the root level.

```json
// ✅ Root is an Object — most common for single records
{
  "id": 1,
  "name": "Alice"
}

// ✅ Root is an Array — most common for lists
[
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob" }
]

// ❌ Root is a string — NOT valid standalone JSON
"Hello"

// ❌ Root is a number — NOT valid standalone JSON
42
```

---

## 🔍 How JSON Looks in Real API Responses

### Single object response (GET /api/employees/1)

```json
{
  "id": 1,
  "name": "Alice Johnson",
  "department": "IT",
  "salary": 75000,
  "hireDate": "2021-06-15",
  "isActive": true,
  "manager": null
}
```

### List response (GET /api/employees)

```json
[
  { "id": 1, "name": "Alice", "department": "IT" },
  { "id": 2, "name": "Bob",   "department": "HR" },
  { "id": 3, "name": "Carol", "department": "Finance" }
]
```

### Wrapped response (common in ASP.NET APIs)

```json
{
  "success": true,
  "message": "Data loaded",
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ],
  "total": 2
}
```

### Error response

```json
{
  "success": false,
  "statusCode": 400,
  "message": "Validation failed",
  "errors": {
    "name":   ["Name is required"],
    "salary": ["Salary must be positive"]
  }
}
```

### Kendo Grid specific response

```json
{
  "Data":   [ { "id": 1, "name": "Alice" } ],
  "Total":  150,
  "Errors": null
}
```

---

## 🔗 Key JSON Terms

| Term                      | Meaning                                                     |
| ------------------------- | ----------------------------------------------------------- |
| **Key**             | The name/label (always a string):`"name"`                 |
| **Value**           | The data:`"Alice"`,`42`,`true`,`null`,`{}`,`[]` |
| **Property**        | A key-value pair:`"name": "Alice"`                        |
| **Object**          | A `{ }`containing properties                              |
| **Array**           | A `[ ]`containing ordered values                          |
| **Nested**          | Objects/arrays inside other objects/arrays                  |
| **Serialization**   | Converting C# object → JSON string                         |
| **Deserialization** | Converting JSON string → C# object                         |

---

## ❓ Interview Questions

**Q: What are the valid data types in JSON?**

> String, Number, Boolean, null, Object, Array. Exactly 6.

**Q: Can JSON have comments?**

> No. JSON does not support comments at all. This is a common gotcha.

**Q: What is the difference between JSON and JavaScript objects?**

> JSON keys must be double-quoted strings. JavaScript object keys can be unquoted. JSON has no functions, undefined, or Date types. JSON is a string format; a JS object is a live in-memory structure.

**Q: Why does JSON use double quotes instead of single quotes?**

> The JSON spec (RFC 7159) strictly defines double quotes for strings. Single quotes are valid in JavaScript but not in JSON.

**Q: What happens if JSON has a trailing comma?**

> It is invalid JSON. `JSON.parse()` will throw a `SyntaxError`.
