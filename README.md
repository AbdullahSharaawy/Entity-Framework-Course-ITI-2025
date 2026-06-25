# Entity-Framework-Course-ITI-2025

If you want a **professional GitHub README format** that is easy to read and suitable for course notes, use the following structure:

````markdown
# Entity Framework Core (EF Core) - Course Notes

> Comprehensive notes based on the ITI 2025 EF Core instructional session.
>
> Topics covered:
> - EF Core fundamentals
> - Database First approach
> - DbContext & DbSet
> - LINQ Queries
> - Change Tracking
> - CRUD Operations
> - Best Practices

---

# What is EF Core?

**Entity Framework Core (EF Core)** is Microsoft's modern Object-Relational Mapper (ORM) for .NET.

It allows developers to:

- Map database tables to C# classes
- Perform CRUD operations without writing SQL manually
- Query data using LINQ
- Track entity changes automatically

## Main Benefits

### 1. Object Relational Mapping (ORM)

Maps:

| Database | C# |
|-----------|-----------|
| Table | Class |
| Row | Object |
| Column | Property |

### 2. LINQ Support

Write queries using C# instead of raw SQL.

```csharp
var departments = db.Departments
                    .Where(d => d.Capacity > 40);
```

### 3. Change Tracking

EF Core automatically detects:

- Added entities
- Modified entities
- Deleted entities

and generates the required SQL commands.

---

# EF Core Approaches

## Database First

Start with an existing database and generate:

- Entity Classes
- DbContext

from the database schema.

✅ Focus of this course.

## Code First

Start with C# classes and generate the database automatically.

---

# Installation

Install the required NuGet packages.

## SQL Server Provider

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

## EF Core Tools

```powershell
Install-Package Microsoft.EntityFrameworkCore.Tools
```

### Other Providers

| Database | Package |
|-----------|-----------|
| SQL Server | Microsoft.EntityFrameworkCore.SqlServer |
| Oracle | Microsoft.EntityFrameworkCore.Oracle |
| SQLite | Microsoft.EntityFrameworkCore.Sqlite |

---

# DbContext and DbSet

## DbContext

The gateway between your application and the database.

Responsibilities:

- Managing database connections
- Tracking entities
- Executing queries
- Saving changes

## DbSet<T>

Represents a table in the database.

Example:

```csharp
public partial class AppContext : DbContext
{
    public virtual DbSet<Department> Departments { get; set; }

    public virtual DbSet<Student> Students { get; set; }
}
```

---

# Database First (Scaffolding)

Generate entities and DbContext from an existing database.

## Scaffold Command

```powershell
Scaffold-DbContext `
"Server=.;Database=TestAdo;Trusted_Connection=True;TrustServerCertificate=True" `
Microsoft.EntityFrameworkCore.SqlServer `
-OutputDir Models `
-ContextDir Data `
-Context AppContext
```

## Command Parameters

| Parameter | Description |
|------------|------------|
| `-OutputDir Models` | Entity classes location |
| `-ContextDir Data` | DbContext location |
| `-Context AppContext` | Context class name |
| `-Force` | Overwrite existing files |

---

# EF Core Power Tools

Alternative visual approach:

1. Right Click Project
2. EF Core Power Tools
3. Reverse Engineer
4. Create Database Connection
5. Select Tables
6. Configure Naming Options
7. Generate Models

---

# Querying Data with LINQ

EF Core translates LINQ expressions into SQL queries.

---

## Deferred Execution

Query is built but not executed immediately.

```csharp
var results = db.Departments
                .Where(d => d.Capacity > 40);
```

Execution occurs when:

```csharp
foreach(var item in results)
{
}
```

or

```csharp
results.ToList();
```

---

## Immediate Execution

Query executes immediately.

```csharp
var dept = db.Departments
             .FirstOrDefault(d => d.Id == 50);
```

---

# Query Examples

## Filtering + Projection

```csharp
using var db = new AppContext();

var results = db.Departments
    .Where(d => d.Capacity > 40)
    .Select(d => new
    {
        d.DeptId,
        d.DeptName
    });

foreach(var item in results)
{
    Console.WriteLine($"{item.DeptId}: {item.DeptName}");
}
```

---

## Retrieve Single Record

```csharp
var department = db.Departments
                   .FirstOrDefault(d => d.DeptId == 50);
```

---

## View Generated SQL

```csharp
var sql = query.ToQueryString();
```

Useful for debugging and performance analysis.

---

# Change Tracking

EF Core tracks entity states automatically.

| State | Description |
|---------|---------|
| Unchanged | No modifications |
| Added | New entity |
| Modified | Existing entity changed |
| Deleted | Entity marked for deletion |

---

# CRUD Operations

## Update

```csharp
var dept = db.Departments
             .FirstOrDefault(d => d.DeptId == 1);

dept.DeptName = "Java";

db.SaveChanges();
```

Generated SQL:

```sql
UPDATE Departments ...
```

---

## Insert

```csharp
var newDept = new Department
{
    DeptId = 2,
    DeptName = "Testing",
    Capacity = 100,
    Status = true
};

db.Departments.Add(newDept);

db.SaveChanges();
```

Generated SQL:

```sql
INSERT INTO Departments ...
```

---

## Delete

```csharp
var deptToDelete = db.Departments
                     .FirstOrDefault(d => d.DeptId == 2);

db.Departments.Remove(deptToDelete);

db.SaveChanges();
```

Generated SQL:

```sql
DELETE FROM Departments ...
```

---

# SaveChanges()

`SaveChanges()` is the unit of work in EF Core.

It:

- Scans tracked entities
- Detects changes
- Generates SQL statements
- Executes them as a transaction

⚠️ Without calling `SaveChanges()`, no changes are persisted to the database.

---

# Best Practices

## Use Partial Classes

Never modify generated entity files directly.

### Generated File

```csharp
public partial class Department
{
    public int DeptId { get; set; }

    public string DeptName { get; set; }
}
```

### Custom File

```csharp
public partial class Department
{
    public override string ToString()
    {
        return $"ID: {DeptId}, Name: {DeptName}";
    }
}
```

### Why?

When you re-run scaffolding:

```powershell
Scaffold-DbContext ...
```

generated files are overwritten, but your custom partial classes remain safe.

---

# Summary

EF Core provides:

- ORM capabilities
- Automatic mapping
- LINQ-based querying
- Change tracking
- Database First scaffolding
- CRUD operations with minimal code

By combining **DbContext**, **DbSet**, **LINQ**, and **Change Tracking**, developers can build data-driven .NET applications efficiently while reducing boilerplate code.
````

This format follows the style commonly used in high-quality GitHub repositories and renders cleanly with headings, tables, code blocks, callouts, and summaries.
