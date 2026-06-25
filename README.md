# Entity-Framework-Course-ITI-2025



# session 1.1

This guide provides a comprehensive overview of Entity Framework Core based on the ITI 2025 instructional session. It covers the core concepts, setup, Database-First approach, querying, and change tracking.

## Key Concepts

- **Object-Relational Mapper (ORM):** EF Core is an ORM that maps relational database tables to business objects (C# classes) and vice versa.
- **Automation:** Replaces manual ADO.NET boilerplate (connections, commands, mappers) by automatically generating entities and handling CRUD operations.
- **Approaches:**
  - **Database First:** Starting with an existing database and generating C# entities and contexts from it.
  - **Code First:** Starting with C# classes and generating the database. (Focus of this session is Database First).
- **Core Features:**
  1. **Mapping:** Automatically maps tables to classes and columns to properties.
  2. **LINQ to Entities:** Allows writing C# LINQ queries that are automatically translated into SQL statements.
  3. **Change Tracking:** Monitors changes to objects in memory to efficiently apply inserts, updates, and deletes to the database.

## Setup & Installation

EF Core is component-based. You only install the packages you need.

### Required Packages

To use EF Core with SQL Server and enable database scaffolding, install the following packages via NuGet Package Manager or the Package Manager Console (PMC):

```powershell
# Core SQL Server provider
Install-Package Microsoft.EntityFrameworkCore.SqlServer

# Tools for migrations and scaffolding (Reverse Engineering)
Install-Package Microsoft.EntityFrameworkCore.Tools
```

> Note: If you were using a different database like Oracle or SQLite, you would install Microsoft.EntityFrameworkCore.Oracle or Microsoft.EntityFrameworkCore.Sqlite respectively.

## DbContext and DbSet

### DbContext

The DbContext is a gateway or session between your C# application and the database. It manages the connection and executes database operations.

### DbSet<T>

Inside the DbContext, a DbSet<T> represents a specific table. It allows you to query and save instances of the entity class.

```csharp
public partial class AppContext : DbContext
{
    // DbSet represents the Departments table
    public virtual DbSet<Department> Departments { get; set; }

    // DbSet represents the Students table
    public virtual DbSet<Student> Students { get; set; }
}
```

## Database First Approach (Scaffolding)

To generate C# entities and the DbContext from an existing database, we use Reverse Engineering (Scaffolding).

### Using the CLI (Package Manager Console)

Use the Scaffold-DbContext command to generate the models:

```powershell
Scaffold-DbContext "Server=.;Database=TestAdo;Trusted_Connection=True;TrustServerCertificate=True" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -ContextDir Data -Context AppContext
```

### Flags Explained

- `-OutputDir Models`: Places generated entity classes inside a Models folder.
- `-ContextDir Data`: Places the generated DbContext inside a Data folder.
- `-Context AppContext`: Names the context class AppContext.
- `-Force`: Overwrites existing files if the database schema changes and you need to re-scaffold.

### Using EF Core Power Tools

Alternatively, you can install the EF Core Power Tools extension for Visual Studio. It provides a visual UI to perform Reverse Engineering:

1. Right-click the project -> EF Core Power Tools -> Reverse Engineer.
2. Create the database connection.
3. Select the required tables.
4. Configure naming conventions (e.g., Pluralize/Singularize) and output directories.

## Querying Data with LINQ

EF Core translates C# LINQ expressions into raw SQL queries.

### Deferred vs. Immediate Execution

| Concept | Explanation | Example |
|----------|-------------|----------|
| Deferred Execution | The query is built in memory but not sent to the SQL Server until the data is explicitly iterated over (e.g., using `foreach` or calling `.ToList()`). | `var results = db.Departments.Where(d => d.Capacity > 40);` |
| Immediate Execution | The query is executed against the database immediately. | `var dept = db.Departments.FirstOrDefault(d => d.Id == 50);` |

### Query Examples

#### 1. Retrieving filtered data with Projection (Select)

```csharp
using var db = new AppContext();

var results = db.Departments
    .Where(d => d.Capacity > 40)
    .Select(d => new { d.DeptId, d.DeptName }); // Projection (Anonymous Object)

foreach (var item in results)
{
    Console.WriteLine($"{item.DeptId}: {item.DeptName}");
}
```

#### 2. Retrieving a single record

```csharp
var department = db.Departments.FirstOrDefault(d => d.DeptId == 50);
```

> Tip: You can inspect the exact SQL query EF Core generates by calling the `.ToQueryString()` method on an `IQueryable` variable.

## Change Tracking and CRUD Operations

EF Core automatically tracks the state of entities retrieved from the database (Unchanged, Modified, Added, Deleted).

### 1. Update (Modified State)

When you modify a retrieved object, EF Core marks its state as Modified.

```csharp
var dept = db.Departments.FirstOrDefault(d => d.DeptId == 1);
dept.DeptName = "Java"; // State becomes Modified

// Generates an UPDATE SQL statement
db.SaveChanges();
```

### 2. Insert (Added State)

To insert a new record, instantiate a new object and use `.Add()`.

```csharp
var newDept = new Department
{
    DeptId = 2,
    DeptName = "Testing",
    Capacity = 100,
    Status = true
};

db.Departments.Add(newDept); // State becomes Added

// Generates an INSERT SQL statement
db.SaveChanges();
```

### 3. Delete (Deleted State)

To delete a record, fetch it first, then use `.Remove()`.

```csharp
var deptToDelete = db.Departments.FirstOrDefault(d => d.DeptId == 2);
db.Departments.Remove(deptToDelete); // State becomes Deleted

// Generates a DELETE SQL statement
db.SaveChanges();
```

> **Callout:** `SaveChanges()` acts as a transaction unit. It scans all tracked objects in memory and generates the necessary SQL operations simultaneously. If you don't call `SaveChanges()`, no changes will be made to the database!

## Best Practices & Tips

### Extending Generated Classes (Partial Classes)

Because EF Core generates entities as partial classes, you should never write custom code (like overriding `ToString()`) directly inside the generated files. If you re-run the `Scaffold-DbContext` command (e.g., after altering a column in SQL Server), your custom code will be deleted.

**Solution:** Create a separate file with the same class name and `partial` keyword.

#### File 1: Generated by EF Core (Do not touch)

```csharp
public partial class Department
{
    public int DeptId { get; set; }
    public string DeptName { get; set; }
}
```

#### File 2: Your Custom Code (Safe from overwriting)

```csharp
public partial class Department
{
    public override string ToString()
    {
        return $"ID: {DeptId}, Name: {DeptName}";
    }
}
```
