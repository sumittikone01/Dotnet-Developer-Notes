
# JSON Objects and Arrays

---

## 📌 Overview

JSON has two container types that hold all data:

| Container        | Syntax  | Purpose                                      |
| ---------------- | ------- | -------------------------------------------- |
| **Object** | `{ }` | Named properties — like a C# class instance |
| **Array**  | `[ ]` | Ordered list of values — like a C# List     |

Understanding these two deeply means you can read and write any JSON in any application.

---

## 🔑 JSON Objects

### What is a JSON Object?

A JSON Object is a collection of **key-value pairs** wrapped in `{ }`.

* Each key is a **string** (double-quoted)
* Each value is any valid JSON type
* Pairs are separated by commas
* Order of keys does NOT matter (unlike arrays)

```json
{
  "id":         1,
  "name":       "Alice Johnson",
  "department": "IT",
  "salary":     75000.00,
  "isActive":   true,
  "manager":    null
}
```

### Object with Nested Object

```json
{
  "id":   1,
  "name": "Alice",
  "address": {
    "street": "123 Main St",
    "city":   "New York",
    "state":  "NY",
    "zip":    "10001"
  },
  "contact": {
    "email": "alice@company.com",
    "phone": "555-1234"
  }
}
```

### Accessing object properties in JavaScript

```javascript
var employee = {
  "id": 1,
  "name": "Alice",
  "address": {
    "city": "New York"
  }
};

// Dot notation
console.log(employee.name);            // "Alice"
console.log(employee.address.city);    // "New York"

// Bracket notation (useful when key has spaces or is dynamic)
console.log(employee["name"]);         // "Alice"
var key = "name";
console.log(employee[key]);            // "Alice"

// Check if a key exists
if ("salary" in employee) { ... }
if (employee.hasOwnProperty("salary")) { ... }
```

### C# equivalent of a JSON Object

```csharp
// This JSON object:
// { "id": 1, "name": "Alice", "salary": 75000 }

// Maps to this C# class:
public class Employee
{
    public int     Id      { get; set; }
    public string  Name    { get; set; }
    public decimal Salary  { get; set; }
}
```

---

## 🔑 JSON Arrays

### What is a JSON Array?

A JSON Array is an **ordered list** of values wrapped in `[ ]`.

* Values are separated by commas
* Values can be any JSON type (including objects and other arrays)
* **Order matters** — index 0 is first, always
* Values don't need to be the same type (though they usually are)

### Array of Strings

```json
["Alice", "Bob", "Carol", "Dave"]
```

### Array of Numbers

```json
[10, 20, 30, 40, 50]
```

### Array of Objects — Most Common in APIs

```json
[
  { "id": 1, "name": "Alice", "department": "IT"      },
  { "id": 2, "name": "Bob",   "department": "HR"      },
  { "id": 3, "name": "Carol", "department": "Finance" }
]
```

### Array of Mixed Types (less common, avoid in APIs)

```json
["Alice", 30, true, null, { "city": "NY" }]
```

### Accessing array items in JavaScript

```javascript
var employees = [
  { "id": 1, "name": "Alice" },
  { "id": 2, "name": "Bob"   }
];

// Access by index
console.log(employees[0]);            // { id: 1, name: "Alice" }
console.log(employees[0].name);       // "Alice"
console.log(employees[1].name);       // "Bob"

// Length
console.log(employees.length);        // 2

// Loop through all
employees.forEach(function(emp) {
  console.log(emp.name);
});

// Find one item
var alice = employees.find(e => e.id === 1);
console.log(alice.name);              // "Alice"

// Filter
var itEmployees = employees.filter(e => e.department === "IT");

// Map (transform)
var names = employees.map(e => e.name);  // ["Alice", "Bob"]
```

### C# equivalent of a JSON Array

```csharp
// This JSON array:
// [ {"id":1,"name":"Alice"}, {"id":2,"name":"Bob"} ]

// Maps to this C#:
List<Employee> employees = new List<Employee>
{
    new Employee { Id = 1, Name = "Alice" },
    new Employee { Id = 2, Name = "Bob"   }
};
```

---

## 🔑 Arrays Inside Objects

Very common pattern — an object that contains a list property:

```json
{
  "department": "IT",
  "headCount": 3,
  "employees": [
    { "id": 1, "name": "Alice", "role": "Developer" },
    { "id": 2, "name": "Bob",   "role": "Tester"    },
    { "id": 3, "name": "Carol", "role": "DevOps"    }
  ]
}
```

```javascript
var dept = { ...above json... };

// Access the array
console.log(dept.employees.length);        // 3
console.log(dept.employees[0].name);       // "Alice"
console.log(dept.employees[2].role);       // "DevOps"

// Loop
dept.employees.forEach(emp => {
  console.log(emp.name + " - " + emp.role);
});
```

---

## 🔑 Objects Inside Arrays

Another common pattern — a list where each item has nested data:

```json
[
  {
    "id": 1,
    "name": "Alice",
    "skills": ["C#", "SQL", "JavaScript"],
    "address": { "city": "New York", "state": "NY" }
  },
  {
    "id": 2,
    "name": "Bob",
    "skills": ["Python", "Docker"],
    "address": { "city": "Chicago", "state": "IL" }
  }
]
```

```javascript
var team = [ ...above json... ];

// Get Bob's first skill
console.log(team[1].skills[0]);         // "Python"

// Get Alice's city
console.log(team[0].address.city);      // "New York"

// Get all cities
var cities = team.map(p => p.address.city);  // ["New York", "Chicago"]

// Find people who know SQL
var sqlDevs = team.filter(p => p.skills.includes("SQL"));
```

---

## 🔑 Empty Objects and Arrays

Both are valid JSON — very common in API responses:

```json
{
  "employees": [],      ← empty array  — no employees yet
  "metadata": {},       ← empty object — no metadata
  "total": 0
}
```

```javascript
var data = { employees: [], metadata: {}, total: 0 };

// Check if empty
if (data.employees.length === 0) {
  console.log("No employees found");
}

if (Object.keys(data.metadata).length === 0) {
  console.log("No metadata");
}
```

---

## 📊 Object vs Array — When to Use Which

| Situation              | Use                             | Example                                      |
| ---------------------- | ------------------------------- | -------------------------------------------- |
| Single record          | Object `{ }`                  | One employee's details                       |
| Multiple records       | Array of objects `[{ }, { }]` | List of all employees                        |
| Fixed known properties | Object `{ }`                  | A person with name, age, email               |
| Variable-count items   | Array `[ ]`                   | Tags, roles, phone numbers                   |
| Key-value lookup       | Object `{ }`                  | `{ "NY": "New York", "CA": "California" }` |
| Ordered sequence       | Array `[ ]`                   | Steps in a process                           |

---

## 💻 Real Examples from ASP.NET Web API

### Controller returning an object (single employee)

```csharp
[HttpGet("{id}")]
public IActionResult GetById(int id)
{
    var emp = _db.Employees.Find(id);
    return Ok(new {
        id         = emp.Id,
        name       = emp.Name,
        department = emp.Department,
        salary     = emp.Salary
    });
}
// JSON sent to browser:
// { "id": 1, "name": "Alice", "department": "IT", "salary": 75000 }
```

### Controller returning an array (list)

```csharp
[HttpGet]
public IActionResult GetAll()
{
    var list = _db.Employees
        .Select(e => new { id = e.Id, name = e.Name })
        .ToList();
    return Ok(list);
}
// JSON sent to browser:
// [ {"id":1,"name":"Alice"}, {"id":2,"name":"Bob"} ]
```

### Consuming array response in AJAX

```javascript
$.get("/api/employees", function(data) {
    // data is already a JS array — jQuery parses JSON automatically
    data.forEach(function(emp) {
        console.log(emp.id + ": " + emp.name);
    });

    // Build an HTML list
    var html = data.map(emp =>
        `<li>${emp.name} — ${emp.department}</li>`
    ).join("");
    $("#employeeList").html("<ul>" + html + "</ul>");
});
```

---

## ❓ Interview Questions

**Q: What is the difference between a JSON object and a JSON array?**

> An object uses `{ }` with named key-value pairs — order doesn't matter. An array uses `[ ]` with ordered indexed values — order matters. Use objects for single records with named properties; use arrays for collections of items.

**Q: Can a JSON array contain different types?**

> Technically yes — JSON allows `["Alice", 30, true, null]`. But in practice, API arrays always contain the same type for consistency and easy iteration.

**Q: What is the index of the first element in a JSON array?**

> 0. Arrays are zero-indexed.

**Q: How do you access a property inside an object inside an array?**

> Use `array[index].property` — for example `employees[0].name`.

**Q: Can JSON object keys be numbers?**

> No. Keys in JSON objects must always be strings (double-quoted). Numbers as keys are not valid JSON.
