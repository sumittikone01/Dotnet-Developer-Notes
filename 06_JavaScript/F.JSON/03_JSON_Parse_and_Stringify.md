# JSON Parse and Stringify

---

## 📌 Overview

When data travels between a browser and a server, it travels as a  **plain text string** .
JSON gives us two built-in operations to convert between text and live objects:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                   │
│   JavaScript Object          JSON String (text)                  │
│   (live, in memory)          (for storage or transfer)           │
│                                                                   │
│   { id: 1, name: "Alice" }                                       │
│                                                                   │
│         ──── JSON.stringify() ────►  '{"id":1,"name":"Alice"}'   │
│                                                                   │
│         ◄─── JSON.parse()    ────   '{"id":1,"name":"Alice"}'    │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

| Method                  | Direction        | Use when                          |
| ----------------------- | ---------------- | --------------------------------- |
| `JSON.stringify(obj)` | Object → String | Sending data to server via AJAX   |
| `JSON.parse(str)`     | String → Object | Reading data received from server |

---

## 🔑 JSON.parse() — String to Object

### Basic Usage

```javascript
// You have a JSON string (e.g., received from server)
var jsonString = '{"id": 1, "name": "Alice", "salary": 75000}';

// Parse it into a JavaScript object
var employee = JSON.parse(jsonString);

// Now you can use it like any JS object
console.log(employee.id);       // 1
console.log(employee.name);     // "Alice"
console.log(employee.salary);   // 75000
console.log(typeof employee);   // "object"
console.log(typeof jsonString); // "string"
```

### Parsing an Array String

```javascript
var jsonArrayStr = '[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]';

var employees = JSON.parse(jsonArrayStr);

console.log(employees.length);       // 2
console.log(employees[0].name);      // "Alice"
console.log(Array.isArray(employees)); // true

employees.forEach(function(emp) {
  console.log(emp.id + ": " + emp.name);
});
```

### Parsing Nested JSON

```javascript
var json = '{"name":"Alice","address":{"city":"New York","zip":"10001"}}';
var obj = JSON.parse(json);

console.log(obj.address.city);   // "New York"
console.log(obj.address.zip);    // "10001"
```

### parse() with Reviver Function (Advanced)

The reviver function runs on **every** key-value pair after parsing. Used to transform values — most commonly to convert date strings into real `Date` objects.

```javascript
var json = '{"name":"Alice","hireDate":"2021-06-15","salary":75000}';

var emp = JSON.parse(json, function(key, value) {
    // key = the property name, value = the raw parsed value
    if (key === "hireDate") {
        return new Date(value);  // convert string → Date object
    }
    return value;  // return unchanged for everything else
});

console.log(emp.name);                    // "Alice"
console.log(emp.hireDate instanceof Date); // true
console.log(emp.hireDate.getFullYear());   // 2021
```

### Error Handling — ALWAYS wrap parse() in try-catch

If the string is not valid JSON, `JSON.parse()` throws a `SyntaxError`. Your app will crash if you don't handle it.

```javascript
function safeParseJSON(jsonString) {
    try {
        return JSON.parse(jsonString);
    } catch (error) {
        console.error("Invalid JSON:", error.message);
        return null; // or a default value
    }
}

// Safe usage
var result = safeParseJSON('{"name":"Alice"}');    // { name: "Alice" }
var bad    = safeParseJSON("this is not JSON");    // null (no crash)
var bad2   = safeParseJSON("{'name':'Alice'}");    // null (single quotes = invalid)
```

---

## 🔑 JSON.stringify() — Object to String

### Basic Usage

```javascript
// You have a JavaScript object
var employee = {
    id:         1,
    name:       "Alice",
    department: "IT",
    salary:     75000
};

// Convert it to a JSON string for sending to server
var jsonString = JSON.stringify(employee);

console.log(jsonString);
// '{"id":1,"name":"Alice","department":"IT","salary":75000}'

console.log(typeof jsonString);  // "string"
```

### Stringify an Array

```javascript
var employees = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
];

var str = JSON.stringify(employees);
console.log(str);
// '[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]'
```

### stringify() with Pretty Printing (Indentation)

The second and third parameters of `stringify` control formatting. Very useful for debugging.

```javascript
var employee = { id: 1, name: "Alice", address: { city: "NY" } };

// No formatting (compact — use for sending over network)
JSON.stringify(employee);
// '{"id":1,"name":"Alice","address":{"city":"NY"}}'

// Pretty print with 2-space indent (use for debugging/logging)
JSON.stringify(employee, null, 2);
// {
//   "id": 1,
//   "name": "Alice",
//   "address": {
//     "city": "NY"
//   }
// }

// Pretty print with 4-space indent
JSON.stringify(employee, null, 4);

// Pretty print with tab indent
JSON.stringify(employee, null, "\t");
```

### stringify() with Replacer — Filter/Transform Properties

The second parameter (replacer) can be an **array** or a  **function** .

```javascript
var employee = {
    id:       1,
    name:     "Alice",
    salary:   75000,
    password: "secret123"  // sensitive — don't send this!
};

// Replacer as ARRAY — only include listed keys
var safe = JSON.stringify(employee, ["id", "name"]);
console.log(safe);
// '{"id":1,"name":"Alice"}'  — password and salary excluded

// Replacer as FUNCTION — transform or filter each value
var result = JSON.stringify(employee, function(key, value) {
    if (key === "password") return undefined; // exclude this key
    if (key === "salary")   return value * 1.1; // apply 10% raise
    return value; // return unchanged
});
console.log(result);
// '{"id":1,"name":"Alice","salary":82500}'
```

---

## 🔑 What stringify() Cannot Serialize

Some JavaScript values cannot be represented in JSON. `stringify()` handles them silently:

```javascript
var obj = {
    name:      "Alice",
    greet:     function() { return "Hi"; }, // ← function
    birthday:  new Date("1990-01-15"),       // ← Date object
    score:     undefined,                    // ← undefined
    ratio:     Infinity,                     // ← Infinity
    bad:       NaN,                          // ← NaN
    sym:       Symbol("id")                  // ← Symbol
};

var str = JSON.stringify(obj);
console.log(str);
// '{"name":"Alice","birthday":"1990-01-15T00:00:00.000Z"}'
// ↑ function: REMOVED
// ↑ undefined: REMOVED
// ↑ Infinity: becomes null
// ↑ NaN: becomes null
// ↑ Symbol: REMOVED
// ↑ Date: converted to ISO string automatically
```

| JS Value      | In stringify()               |
| ------------- | ---------------------------- |
| `function`  | Property removed entirely    |
| `undefined` | Property removed entirely    |
| `Symbol`    | Property removed entirely    |
| `Date`      | Converted to ISO 8601 string |
| `Infinity`  | Becomes `null`             |
| `NaN`       | Becomes `null`             |
| `null`      | Stays as `null`            |

---

## 💻 Real Usage — AJAX with stringify and parse

### Sending data to ASP.NET Controller (stringify)

```javascript
// Collect form data
var employeeData = {
    name:       $("#Name").val(),
    department: $("#Department").val(),
    salary:     parseFloat($("#Salary").val()),
    isActive:   $("#IsActive").is(":checked")
};

// Send via AJAX — must stringify the body
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",   // tell server it's JSON
    data:        JSON.stringify(employeeData),   // ← stringify here
    success: function(response) {
        console.log("Created:", response);
    },
    error: function(xhr) {
        console.error("Error:", xhr.responseText);
    }
});
```

### Receiving data from server (parse — usually automatic)

```javascript
// jQuery automatically parses JSON when dataType is "json"
$.ajax({
    url:      "/Employee/GetById/1",
    type:     "GET",
    dataType: "json",           // jQuery auto-parses the response
    success: function(employee) {
        // employee is already a JS object — no manual parse needed
        console.log(employee.name);      // "Alice"
        console.log(employee.salary);    // 75000
    }
});

// With Fetch API — you call .json() which parses automatically
fetch("/Employee/GetById/1")
    .then(response => response.json())  // ← parses internally
    .then(employee => {
        console.log(employee.name);
    });

// Manual parse — when you have raw string from non-AJAX source
var rawString = localStorage.getItem("cachedEmployee");
if (rawString) {
    var employee = JSON.parse(rawString);
    console.log(employee.name);
}
```

### Storing and reading from localStorage

```javascript
var employee = { id: 1, name: "Alice", dept: "IT" };

// Store — must stringify (localStorage only accepts strings)
localStorage.setItem("currentEmployee", JSON.stringify(employee));

// Read back — must parse
var stored = localStorage.getItem("currentEmployee");
var emp    = JSON.parse(stored);
console.log(emp.name);  // "Alice"
```

---

## 🔑 Deep Clone Trick with JSON

A popular trick to create a true deep copy of an object (no shared references):

```javascript
var original = {
    name:    "Alice",
    address: { city: "New York" },
    skills:  ["C#", "SQL"]
};

// Shallow copy — address and skills are SHARED references
var shallow = Object.assign({}, original);
shallow.address.city = "Chicago";
console.log(original.address.city);  // "Chicago" ← original changed!

// Deep clone with JSON — address and skills are fully independent
var deep = JSON.parse(JSON.stringify(original));
deep.address.city = "Chicago";
console.log(original.address.city);  // "New York" ← original unchanged ✅

// ⚠️ Limitation: loses functions, undefined, Date objects
```

---

## 📊 parse() vs stringify() Quick Reference

```
JSON.parse(string)    →    JavaScript Object/Array
─────────────────────────────────────────────────
Input : '{"name":"Alice","salary":75000}'
Output: { name: "Alice", salary: 75000 }

JSON.stringify(object)    →    JSON String
─────────────────────────────────────────────────
Input : { name: "Alice", salary: 75000 }
Output: '{"name":"Alice","salary":75000}'
```

---

## ❓ Interview Questions

**Q: What does JSON.parse() do?**

> It converts a JSON-formatted string into a JavaScript object or array that you can work with in code.

**Q: What does JSON.stringify() do?**

> It converts a JavaScript object or array into a JSON string for storage, transmission, or display.

**Q: What happens if you pass invalid JSON to JSON.parse()?**

> It throws a `SyntaxError`. You should always wrap `JSON.parse()` in a `try-catch` block.

**Q: What values does JSON.stringify() not include?**

> Functions, `undefined`, and Symbols are removed. `Infinity` and `NaN` become `null`. Dates become ISO strings.

**Q: How do you pretty-print JSON?**

> Use the third parameter: `JSON.stringify(obj, null, 2)` for 2-space indent.

**Q: Why do we use JSON.stringify when sending AJAX POST requests?**

> The HTTP request body must be a string. `JSON.stringify()` converts the object to a JSON string, and setting `contentType: "application/json"` tells the server how to interpret it.

**Q: Does jQuery's $.ajax automatically parse JSON responses?**

> Yes, when `dataType: "json"` is set, or when the server returns `Content-Type: application/json`, jQuery parses the response automatically.

---
