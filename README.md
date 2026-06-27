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



# session 1.2

## Key Concepts

- **EF Core Setup & Dependencies**: Installing the required NuGet packages.
- **Entity Creation**: Defining C# classes to represent database tables.
- **DbContext & DbSet**: Setting up the gateway between the application and the database.
- **Connection Configuration**: Specifying the database provider and connection string.
- **Migrations**: Keeping the database schema in sync with your C# models.
- **Model Configuration Strategies**: Guiding EF Core on how to map classes to tables using:
  - Conventions
  - Data Annotations
  - Fluent API
- **Interacting with Data**: Querying data using LINQ and proper resource disposal.

---

## 1. EF Core Setup & Dependencies

To work with Entity Framework Core using SQL Server, you must install the following NuGet packages via the NuGet Package Manager:

- `Microsoft.EntityFrameworkCore.SqlServer` (Provides the SQL Server database provider).
- `Microsoft.EntityFrameworkCore.Tools` (Enables EF Core Package Manager Console commands like Migrations).

---

## 2. Entity Creation

You start by defining models (Entities) in C# that will map to tables in your database.

```csharp
namespace Demo.Models
{
    public class Department
    {
        public int DeptId { get; set; }
        public string DeptName { get; set; }
        public int? Capacity { get; set; } // Nullable property
    }
}
```

---

## 3. DbContext Setup & Connection Configuration

The `DbContext` class acts as the bridge to the database. You define `DbSet<T>` properties for your entities (which translate to tables) and override the `OnConfiguring` method to specify the connection string.

> **Tip:** You don't necessarily need a full SQL Server instance installed. Visual Studio comes with a lightweight LocalDB (`(localdb)\mssqllocaldb`) which is excellent for development.

```csharp
using Microsoft.EntityFrameworkCore;
using Demo.Models;

namespace Demo.Context
{
    public class ITIContext : DbContext
    {
        // Define DbSets for your entities
        public DbSet<Department> Departments { get; set; }
        public DbSet<Student> Students { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // Configure connection to SQL Server or LocalDB
            optionsBuilder.UseSqlServer(
                "Server=(localdb)\\mssqllocaldb;Database=ITIDatabase;Trusted_Connection=True;TrustServerCertificate=True;");
        }
    }
}
```

---

## 4. Migrations

Migrations translate your C# model changes into SQL commands to update the database schema.

Use the **Package Manager Console (PMC)** to execute these commands:

| Command | Description |
|----------|-------------|
| `Add-Migration <Name>` | Scaffolds a new migration file based on changes in your models. Use a descriptive name (e.g., `InitialCreate`, `AddCapacityColumn`). |
| `Update-Database` | Applies pending migrations to the database, actually creating/modifying the tables. |
| `Remove-Migration` | Removes the last unapplied migration from your code if you need to revert and make changes before updating the database. |

---

## 5. Model Configuration Approaches

EF Core needs to know how to create tables and columns (e.g., what is the Primary Key? Is a column nullable?). It determines this using three main strategies.

### A. Conventions

Default rules EF Core follows automatically.

- A property named `Id` or `<ClassName>Id` (e.g., `DepartmentId`) is automatically configured as the **Primary Key** and made an **Identity** (auto-incrementing) column.
- Reference types (like `string`) map to `nvarchar(MAX)` and are nullable by default (if C# Nullable Reference Types are disabled).

### B. Data Annotations

Attributes applied directly to entity properties for explicit configuration. It is easier to use than Fluent API but less comprehensive.

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class Student
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int StdId { get; set; }

    [Required]
    [StringLength(20)]
    [Column("StudentName")]
    public string Name { get; set; }

    public int? Age { get; set; }
}
```

### C. Fluent API

The most powerful configuration method, written inside the `OnModelCreating` method in the `DbContext`.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Department>(entity =>
    {
        entity.HasKey(d => d.DeptId);

        entity.Property(d => d.DeptName)
              .IsRequired()
              .HasMaxLength(20)
              .HasColumnName("DepartmentName");
    });
}
```

> **Important Precedence Rule:** `Fluent API > Data Annotations > Conventions`

---

## 6. Interacting with Data & Best Practices

To query or manipulate data, instantiate the `DbContext` within a `using` block. The `DbContext` holds onto database connections and resources; the `using` statement ensures the `Dispose()` method is called automatically.

```csharp
using (ITIContext db = new ITIContext())
{
    var departments = db.Departments.ToList();

    foreach (var item in departments)
    {
        Console.WriteLine(
            $"ID: {item.DeptId} | Name: {item.DeptName} | Capacity: {item.Capacity}");
    }
}
```



# session 1.3

## Key Concepts

* Setting up `DbContext` and Connection Strings
* Entity Configuration (Conventions, Data Annotations, Fluent API)
* Working with Migrations
* One-to-Many (1:N) Relationships
* Many-to-Many (N:M) Relationships (Implicit vs. Explicit Join Tables)
* One-to-One (1:1) and Self-Referencing Relationships
* LocalDB vs. SQL Server Setup

---

## 1. Setting up `DbContext` and Connection Strings

To start working with Entity Framework Core, you need to define your entities and configure your `DbContext`. `DbContext` acts as the bridge between your entities and the database.

### Required NuGet Packages

* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Tools`

### Notes

* **OnConfiguring**: Used to supply the connection string.
* **DbSet**: Properties representing the tables in the database.

```csharp
public class ITIContext : DbContext
{
    public DbSet<Department> Departments { get; set; }
    public DbSet<Student> Students { get; set; }
    public DbSet<Course> Courses { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Configuring the connection to SQL Server
        optionsBuilder.UseSqlServer(
            "Server=.;Database=ITI_DB;Trusted_Connection=True;");
    }
}
```

---

## 2. Entity Configuration

EF Core needs help determining how to map your C# classes to database tables. You can achieve this using three approaches:

| Approach               | Description                                                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| **Naming Conventions** | EF Core automatically recognizes `Id` or `ClassNameId` as the Primary Key.                      |
| **Data Annotations**   | Attributes placed directly on entity properties (e.g., `[Key]`, `[Required]`).                  |
| **Fluent API**         | Written inside `OnModelCreating` in the `DbContext`. Offers the highest level of configuration. |

### Data Annotations Example

```csharp
public class Student
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int StudentId { get; set; }

    [Required]
    [StringLength(50)]
    public string Name { get; set; }
}
```

### Fluent API Example

If you do not want to pollute your entity classes with attributes, or if you need advanced configurations (like composite keys), use Fluent API.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Course>(c =>
    {
        c.HasKey(e => e.CourseId);

        c.Property(e => e.CourseId)
         .ValueGeneratedNever();

        c.Property(e => e.CourseName)
         .IsRequired()
         .HasMaxLength(20);
    });
}
```

> **Callout:** Fluent API takes precedence over Data Annotations, which in turn take precedence over default Naming Conventions.

---

## 3. Migrations

Migrations are used to keep the database schema in sync with your C# models. EF Core tracks these changes using a `ModelSnapshot` file.

| Command                | Description                                                           |
| ---------------------- | --------------------------------------------------------------------- |
| `Add-Migration <Name>` | Scaffolds a new migration file based on model changes.                |
| `Update-Database`      | Applies pending migrations to the physical database.                  |
| `Remove-Migration`     | Safely reverts the last unapplied migration and updates the snapshot. |

> **Warning:** Never delete migration files manually from the folder. Always use `Remove-Migration` so EF Core can update the `ModelSnapshot` accordingly.

---

## 4. One-to-Many (1:N) Relationships

A common relationship, such as a `Department` having many `Students`.

### Entity Configuration

```csharp
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }

    // Collection navigation property
    public virtual ICollection<Student> Students { get; set; }
        = new HashSet<Student>();
}

public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }

    [ForeignKey("Department")]
    public int DeptNumber { get; set; }

    // Reference navigation property
    public virtual Department Department { get; set; }
}
```

### Fluent API Equivalent

```csharp
modelBuilder.Entity<Department>()
    .HasMany(d => d.Students)
    .WithOne(s => s.Department)
    .HasForeignKey(s => s.DeptNumber)
    .IsRequired(true);
```

---

## 5. Many-to-Many (N:M) Relationships

A single `Student` can enroll in many `Courses`, and a single `Course` can have many `Students`.

### Implicit Join Table

Simply add collection navigation properties on both sides, and EF Core will automatically generate the join table.

### Explicit Join Table (With Payload)

If the join table requires additional attributes (such as a `Grade`), create a join entity manually.

```csharp
public class StudentCourse
{
    public int StudentId { get; set; }
    public int CourseId { get; set; }
    public int? Grade { get; set; }

    public virtual Student Student { get; set; }
    public virtual Course Course { get; set; }
}
```

### Composite Key Configuration

Data Annotations cannot create composite keys. Use Fluent API instead.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<StudentCourse>()
        .HasKey(sc => new { sc.StudentId, sc.CourseId });
}
```

---

## 6. One-to-One & Self-Referencing Relationships

### One-to-One (1:1)

Example: An `Instructor` and an `Office`.

* Place navigation properties on both sides.
* Place the Foreign Key on the dependent side.
* If the Foreign Key is nullable (`int?`), the relationship is optional.
* If the Foreign Key is non-nullable (`int`), the relationship is mandatory.

### Self-Referencing Relationship

Example: A `Student` having another `Student` as a leader.

* Create a Foreign Key that references the same entity.
* The Foreign Key should be nullable because the top-level leader has no leader.

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }

    public int? LeaderId { get; set; }

    [ForeignKey("LeaderId")]
    public virtual Student Leader { get; set; }
}
```

---

## 7. LocalDB Configuration

If you do not have a full SQL Server instance installed, Visual Studio ships with `LocalDB`. You can easily swap your connection string to target this lightweight local server.

### Update the `OnConfiguring` Method

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer(
        @"Server=(localdb)\MSSQLLocalDB;Database=MonthTwoDB;Trusted_Connection=True;");
}
```



# session 2.1

## Key Concepts

* **EF Core Architecture Review**: DbContext, DbSets, and the role of tracking.
* **Inheritance Mapping Strategies**: Managing object-oriented inheritance in relational databases using TPH, TPT, and TPC.
* **Entity Tracking Optimization**: Using `AsNoTracking` to boost performance in read-only scenarios.
* **Client-Side vs. Server-Side Evaluation**: Understanding how LINQ queries are translated and executed.
* **Eager Loading**: Fetching related data (Navigation Properties) using `Include` and `ThenInclude`.

---

## Explanation

### 1. EF Core Architecture Review

Entity Framework Core acts as an Object-Relational Mapper (ORM), bridging your .NET Core application and the Database. It translates LINQ queries into SQL and tracks the state of retrieved objects (Added, Modified, Deleted, Unchanged) so changes can be persisted via `SaveChanges()`.

> **Callout:** For EF Core to function, you need standard packages like `Microsoft.EntityFrameworkCore.SqlServer` for the database provider and `Microsoft.EntityFrameworkCore.Tools` to manage migrations.

---

### 2. Inheritance Mapping Strategies

When dealing with object-oriented inheritance (e.g., `Student` and `Employee` inheriting from `Person`), relational databases don't natively understand class hierarchies. EF Core provides three strategies to map these models:

#### A. Table-Per-Hierarchy (TPH)

This is the **default behavior** in EF Core. The entire inheritance hierarchy is mapped to a **single table**.

* EF Core adds a **`Discriminator`** column to distinguish the entity type (e.g., "Student" or "Employee").
* **Pros:** High performance (no SQL `JOIN`s needed).
* **Cons:** Columns specific to a child class will be `NULL` for other child classes.

#### B. Table-Per-Type (TPT)

Each class (base and derived) is mapped to its own table. Derived tables are linked to the base table using a Foreign Key that also acts as a Primary Key.

* **Configuration:** Configured via the Fluent API.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>().UseTptMappingStrategy();
}

```

* **Pros:** Data is normalized without `NULL` columns.
* **Cons:** Reading data requires complex `JOIN`s, which can negatively impact performance.

#### C. Table-Per-Concrete-Class (TPC)

A table is created *only* for concrete classes (classes that can be instantiated). Abstract base classes do not get their own tables. All properties from the base class are duplicated into the tables of the derived classes.

* **Configuration:** Make the base class `abstract` and apply the strategy.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>().UseTpcMappingStrategy();
}

```

#### Mapping Strategy Comparison

| Feature | TPH (Table Per Hierarchy) | TPT (Table Per Type) | TPC (Table Per Concrete) |
| --- | --- | --- | --- |
| **Number of Tables** | 1 for the whole hierarchy | 1 for every class (base & derived) | 1 for every concrete derived class |
| **Performance** | Fastest (No Joins) | Slowest (Requires Joins) | Fast (No Joins, but duplicated schema) |
| **Nullability** | Has unneeded `NULL` columns | Highly normalized | Highly normalized |
| **Discriminator** | Yes | No | No |

---

### 3. Entity Tracking Optimization

EF Core tracks the state of entities retrieved from the database by default. While this is necessary for updates, the tracking overhead is wasteful if you only intend to *read* data.

#### Disabling Tracking for a Specific Query

To boost performance for read-only queries, use `.AsNoTracking()`. The entity state will remain `Detached`, and `SaveChanges()` will ignore it.

```csharp
var student = db.Students.AsNoTracking().FirstOrDefault(s => s.Id == 1);
// Modification will NOT be saved to the database
student.Name = "Hamada"; 
db.SaveChanges(); 

```

#### Disabling Tracking Globally

If a `DbContext` is primarily used for reporting or reading data, you can disable tracking globally in `OnConfiguring`:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer("YourConnectionString")
                  .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
}

```

---

### 4. Client-Side vs. Server-Side Evaluation

When writing LINQ queries, EF Core attempts to translate the entire expression into SQL to run on the **Server (Database)**.

If your LINQ query contains a custom C# function that SQL Server doesn't understand, EF Core behavior depends on where the function is used:

* **In `Select` (Projection):** EF Core retrieves the required raw data from the server, then evaluates the custom function in the application memory (Client-Side).
* **In `Where` (Filtering):** EF Core **throws a runtime exception** because it cannot execute the C# method on the database server to filter the rows.

> **Best Practice:** Keep operations within the database server as much as possible. If you strictly need client-side evaluation for a filter, you must pull the data into memory first using `.ToList()`, but be warned—this fetches the entire table and degrades performance heavily.

```csharp
// BAD: Throws exception because MyCustomFunction cannot be translated to SQL
var result = db.Students.Where(s => MyCustomFunction(s.Name)).ToList();

// WORKAROUND: Pull to memory first (Not recommended for large tables)
var result = db.Students.ToList().Where(s => MyCustomFunction(s.Name)).ToList();

```

---

### 5. Eager Loading Related Data

By default, Navigation Properties (related entities) are `NULL` when you query a record. To load related data simultaneously in one query, use **Eager Loading**.

* Use **`Include()`** to load direct relationships.
* Use **`ThenInclude()`** to load nested relationships (relationships of the included entity).

```csharp
var student = db.Students
    .Include(s => s.Department)                  // Loads the Department the Student belongs to
    .ThenInclude(d => d.Courses)                 // Loads the Courses inside that Department
    .FirstOrDefault(s => s.StudentId == 1);

Console.WriteLine(student.Department.DeptName);

```

> **Warning:** Overusing `Include` and `ThenInclude` can lead to a **Cartesian Explosion**. This occurs when multiple one-to-many relationships are joined, multiplying the returned rows exponentially and crashing application performance. Always monitor the generated SQL using tools like SQL Server Profiler.



# session 2.2

## Key Concepts

* **Explicit Loading vs. Lazy Loading:** Strategies for efficiently fetching related data.
* **Configuration Separation:** Refactoring `OnModelCreating` by organizing Fluent API configurations into separate modular classes using `IEntityTypeConfiguration<T>`.
* **Assembly Configuration Loading:** Dynamically registering all entity configurations at once using `ApplyConfigurationsFromAssembly`.
* **Raw SQL Queries:** Executing raw SQL statements mapped directly to entities using `FromSql`.
* **Custom View Mapping (`SqlQuery`):** Mapping raw SQL query results to custom Data Transfer Objects (DTOs) or View Models.
* **Stored Procedures:** Executing stored procedures via raw SQL and scripting them programmatically through EF Core Migrations using `migrationBuilder.Sql()`.

## Explanation

### 1. Explicit Loading

Explicit loading allows you to retrieve the main entity first and deliberately issue a secondary query to fetch its related data later. This development effort provides granular control, allowing you to delay loading heavy relational data until your application's logic explicitly requires it.

* **For single objects (1-to-1 or Many-to-1):** Use the `Reference()` method.
* **For collections (1-to-Many):** Use the `Collection()` method.

```csharp
// Fetch the main entity first
var student = db.Students.FirstOrDefault(s => s.Id == 1);

// Explicitly load a single related object (Navigation Property)
db.Entry(student).Reference(s => s.Department).Load();

// Explicitly load a related collection 
var department = db.Departments.FirstOrDefault(d => d.Id == 1);
db.Entry(department).Collection(d => d.Students).Load();

```

### 2. Lazy Loading

Lazy loading defers the initialization of related data until the exact moment the navigation property is accessed in the code. EF Core achieves this by generating proxy classes at runtime that override your navigation properties to fetch data seamlessly.

**Implementation Steps:**

1. **Mark Navigation Properties as `virtual**`: This is mandatory so the proxy class can override them.
```csharp
public virtual Department Department { get; set; }
public virtual ICollection<Student> Students { get; set; } = new List<Student>();

```


2. **Install the Proxies Package via CLI**:
```bash
dotnet add package Microsoft.EntityFrameworkCore.Proxies

```


3. **Enable Lazy Loading in `DbContext**`:
```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseLazyLoadingProxies() // Enable Proxies
        .UseSqlServer("Your_Connection_String");
}

```



> **Performance Tip:** While convenient for desktop applications, Lazy Loading can lead to the "N+1 Query Problem" in web applications, causing significant performance bottlenecks. For web APIs, Eager Loading (`Include`) or Explicit Loading is heavily recommended.

### 3. Refactoring Fluent API Configurations

As your database schemas and domain models grow, keeping all Fluent API configurations tightly coupled inside `OnModelCreating` becomes unmanageable. The standard architectural practice is to isolate each entity's configuration into its own dedicated class.

**Creating the Configuration Class:**

```csharp
public class StudentConfiguration : IEntityTypeConfiguration<Student>
{
    public void Configure(EntityTypeBuilder<Student> builder)
    {
        builder.HasKey(s => s.Id);
        builder.Property(s => s.Name).IsRequired().HasMaxLength(50);
        
        // Configuring relationships
        builder.HasOne(s => s.Department)
               .WithMany(d => d.Students)
               .HasForeignKey(s => s.DeptId);
    }
}

```

**Applying Configurations in DbContext:**
Instead of applying them one by one, you can scan the assembly and intelligently load all classes implementing `IEntityTypeConfiguration<T>` with a single method call.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Scans the given assembly and applies all configuration classes dynamically
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(StudentConfiguration).Assembly);
    
    base.OnModelCreating(modelBuilder);
}

```

### 4. Raw SQL Queries and Stored Procedures

When standard LINQ operations aren't sufficient for complex data aggregations or when executing legacy SQL logic, EF Core provides pathways for raw SQL execution.

#### Mapping to Tracked Entities

Use `FromSql` or `FromSqlInterpolated` when your raw SQL returns complete, tracked entity objects.

```csharp
int targetAge = 20;
// Using string interpolation automatically generates parameterized SQL to prevent SQL injection
var olderStudents = db.Students
    .FromSql($"SELECT * FROM Students WHERE Age > {targetAge}")
    .ToList();

```

#### Mapping to Custom DTOs / Views

If your SQL query only selects a subset of columns (e.g., `SELECT Id, Name FROM Students`), mapping it to the original `Student` entity will throw an `InvalidOperationException` because EF Core requires all properties to be present. Instead, build a DTO (Data Transfer Object) class and map the projection using `SqlQuery`.

```csharp
// 1. Create a DTO / View Model representing the projection
public class StudentView 
{
    public int StudentId { get; set; }
    public string Name { get; set; }
}

// 2. Query and map the subset data directly to the DTO
var partialData = db.Database
    .SqlQuery<StudentView>($"SELECT Id AS StudentId, Name FROM Students")
    .ToList();

```

#### Executing and Creating Stored Procedures

You can execute an existing stored procedure using `FromSql` just like a regular query:

```csharp
var students = db.Students.FromSql($"EXEC GetStudents").ToList();

```

To **create** a Stored Procedure, Database View, or Trigger seamlessly inside the database using Code-First Migrations, generate an empty migration script and utilize `migrationBuilder.Sql()`:

```bash
dotnet ef migrations add CreateGetDepartmentsProc

```

```csharp
public partial class CreateGetDepartmentsProc : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Define the execution logic 
        migrationBuilder.Sql(@"
            CREATE PROCEDURE GetDepartments
            AS
            BEGIN
                SELECT * FROM Departments
            END
        ");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Rollback instruction
        migrationBuilder.Sql("DROP PROCEDURE GetDepartments");
    }
}

```
