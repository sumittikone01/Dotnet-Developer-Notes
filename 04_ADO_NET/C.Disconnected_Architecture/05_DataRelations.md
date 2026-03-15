
# 05 — DataRelations

---

## 🎯 One-Line Definition

> **`DataRelation` creates a parent-child link between two DataTables inside a DataSet — like a foreign key in memory — letting you navigate from parent rows to their child rows and back.**

---

## 🔷 What is DataRelation?

When a DataSet holds multiple tables, `DataRelation` defines how they connect.
It mirrors a **foreign key relationship** — but entirely in memory.

> 💡 Think of `DataRelation` like a **family tree** — Departments are the parents, Employees are the children. You can say "show me all children of the IT department" — that's exactly what DataRelation does in memory.

---

## 🔷 The Problem It Solves

```
WITHOUT DataRelation:
  DataSet has: Employees table  +  Departments table
  You want: "which employees belong to Department 3?"
  You have to loop manually and check DeptId = 3 yourself

WITH DataRelation:
  DataSet has: Employees + Departments + a DataRelation between them
  You want: "which employees belong to this department row?"
  deptRow.GetChildRows("EmpDept")  → returns all matching employee rows instantly
```

---

## 🔷 The Structure

```
DataSet
│
├── DataTable "Departments"       ← PARENT table
│   ├── Id (PrimaryKey)
│   └── Name
│
├── DataTable "Employees"         ← CHILD table
│   ├── Id
│   ├── Name
│   └── DeptId  ← foreign key → links to Departments.Id
│
└── DataRelation "EmpDept"        ← the link
    ├── ParentTable:  Departments.Id
    └── ChildTable:   Employees.DeptId
```

---

## 🔷 Creating a DataRelation

```csharp
DataSet ds = new DataSet();

// ── Step 1: Fill tables from DB ───────────────────────────────────
string cs = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";

using (SqlDataAdapter da1 = new SqlDataAdapter("SELECT Id, Name FROM Department", cs))
    da1.Fill(ds, "Departments");

using (SqlDataAdapter da2 = new SqlDataAdapter("SELECT Id, Name, DeptId FROM Employee", cs))
    da2.Fill(ds, "Employees");

// ── Step 2: Get the columns to link ──────────────────────────────
DataColumn parentCol = ds.Tables["Departments"].Columns["Id"];     // parent key
DataColumn childCol  = ds.Tables["Employees"].Columns["DeptId"];   // foreign key

// ── Step 3: Create the DataRelation ──────────────────────────────
DataRelation relation = new DataRelation(
    "EmpDept",    // relation name — used to reference it later
    parentCol,    // parent column
    childCol      // child column
);

// ── Step 4: Add it to the DataSet ────────────────────────────────
ds.Relations.Add(relation);

Console.WriteLine($"Relations defined: {ds.Relations.Count}");
```

---

## 🔷 Navigate Parent → Children

```csharp
// For each Department, find all its Employees
foreach (DataRow deptRow in ds.Tables["Departments"].Rows)
{
    Console.WriteLine($"\nDepartment: {deptRow["Name"]}");

    // GetChildRows using the relation name
    DataRow[] employees = deptRow.GetChildRows("EmpDept");

    foreach (DataRow emp in employees)
    {
        Console.WriteLine($"   └── {emp["Name"]}");
    }
}

// Output:
// Department: IT
//    └── Alice
//    └── Carol
// Department: HR
//    └── Bob
```

---

## 🔷 Navigate Child → Parent

```csharp
// For each Employee, find their Department
foreach (DataRow empRow in ds.Tables["Employees"].Rows)
{
    // GetParentRow using the relation name
    DataRow dept = empRow.GetParentRow("EmpDept");

    string deptName = dept != null ? dept["Name"].ToString() : "No Department";
    Console.WriteLine($"{empRow["Name"]}  →  {deptName}");
}

// Output:
// Alice  →  IT
// Bob    →  HR
// Carol  →  IT
```

---

## 🔷 Complete Real Example

```csharp
public void LoadAndNavigateRelation()
{
    string cs = @"Server=.\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;";

    DataSet ds = new DataSet("CompanyData");

    // Load both tables
    using (SqlDataAdapter da = new SqlDataAdapter("SELECT Id, Name FROM Department", cs))
        da.Fill(ds, "Departments");

    using (SqlDataAdapter da = new SqlDataAdapter("SELECT Id, Name, Salary, DeptId FROM Employee", cs))
        da.Fill(ds, "Employees");

    // Define relation
    ds.Relations.Add("EmpDept",
        ds.Tables["Departments"].Columns["Id"],
        ds.Tables["Employees"].Columns["DeptId"]
    );

    // Navigate parent to children
    foreach (DataRow dept in ds.Tables["Departments"].Rows)
    {
        DataRow[] empInDept = dept.GetChildRows("EmpDept");

        decimal totalSalary = 0;
        foreach (DataRow emp in empInDept)
            totalSalary += (decimal)emp["Salary"];

        Console.WriteLine($"{dept["Name"]}:");
        Console.WriteLine($"   Employees:    {empInDept.Length}");
        Console.WriteLine($"   Total Salary: {totalSalary:C0}");
    }
}
```

---

## 🔷 Multiple Relations in One DataSet

```csharp
// Departments → Employees
ds.Relations.Add("EmpDept",
    ds.Tables["Departments"].Columns["Id"],
    ds.Tables["Employees"].Columns["DeptId"]);

// Employees → Projects
ds.Relations.Add("EmpProjects",
    ds.Tables["Employees"].Columns["Id"],
    ds.Tables["Projects"].Columns["EmployeeId"]);

// Navigate three levels:
// Department → Employees → Projects
foreach (DataRow dept in ds.Tables["Departments"].Rows)
{
    Console.WriteLine($"Department: {dept["Name"]}");
    foreach (DataRow emp in dept.GetChildRows("EmpDept"))
    {
        Console.WriteLine($"  Employee: {emp["Name"]}");
        foreach (DataRow proj in emp.GetChildRows("EmpProjects"))
        {
            Console.WriteLine($"    Project: {proj["Title"]}");
        }
    }
}
```

---

## 🔷 DataRelation vs SQL JOIN

|                   | DataRelation           | SQL JOIN                |
| ----------------- | ---------------------- | ----------------------- |
| Lives in          | Memory (DataSet)       | SQL Server              |
| Connection needed | ❌ No — fully offline | ✅ Must be connected    |
| Defined once      | ✅ Yes — in DataSet   | Repeated in every query |
| Performance       | Fast (in-memory)       | Network round-trip      |
| Use when          | DataSet already loaded | Data not yet in memory  |

> 📌 DataRelation is the in-memory equivalent of a SQL JOIN — use it when you already have the data in a DataSet and want to navigate between related tables without another DB call.

---

## 🔷 DataRelation Properties

| Property          | What It Returns                 |
| ----------------- | ------------------------------- |
| `RelationName`  | The name you gave the relation  |
| `ParentTable`   | The parent DataTable            |
| `ChildTable`    | The child DataTable             |
| `ParentColumns` | The parent key column(s)        |
| `ChildColumns`  | The child foreign key column(s) |

```csharp
DataRelation rel = ds.Relations["EmpDept"];
Console.WriteLine(rel.ParentTable.TableName);  // Departments
Console.WriteLine(rel.ChildTable.TableName);   // Employees
```

---

## ⚠️ Common Mistakes

| Mistake                                     | What Happens                     | Fix                                                     |
| ------------------------------------------- | -------------------------------- | ------------------------------------------------------- |
| Column names don't match types              | Exception when creating relation | Parent and child columns must have the same data type   |
| Wrong column used (name vs id)              | No child rows returned           | Parent =`Departments.Id`, Child =`Employees.DeptId` |
| Referencing relation before adding to ds    | Relation not found               | Always add relation to `ds.Relations`before using     |
| Typo in relation name in `GetChildRows()` | `ArgumentException`            | Relation name must match exactly                        |

---

## ⭐ Interview Quick-Fire

| Question                                    | Answer                                                              |
| ------------------------------------------- | ------------------------------------------------------------------- |
| What is DataRelation?                       | In-memory parent-child link between two DataTables inside a DataSet |
| SQL equivalent?                             | Foreign key constraint / JOIN                                       |
| How to navigate parent to children?         | `parentRow.GetChildRows("RelationName")`                          |
| How to navigate child to parent?            | `childRow.GetParentRow("RelationName")`                           |
| Where is DataRelation stored?               | `ds.Relations`collection on the DataSet                           |
| Do parent and child columns need same type? | ✅ Yes — must match exactly                                        |
| When to use over SQL JOIN?                  | When data is already in a DataSet — avoids extra DB round-trip     |
