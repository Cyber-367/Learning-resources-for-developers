# 📘 Full Guide: GraphQL + Hot Chocolate + Entity Framework Core in .NET

This `README.md` file is a complete beginner-to-intermediate guide that explains **GraphQL**, **Hot Chocolate**, and **Entity Framework Core (EF Core)**—with explanations, code snippets, and integration patterns.

---

## 📌 Table of Contents

1. [What is GraphQL?](#what-is-graphql)
2. [What is Hot Chocolate?](#what-is-hot-chocolate)
3. [What is Entity Framework Core?](#what-is-entity-framework-core)
4. [How They Work Together](#how-they-work-together)
5. [Complete Example Project Structure](#complete-example-project-structure)
6. [Code Examples](#code-examples)
7. [Tools and Resources](#tools-and-resources)
8. [Conclusion](#conclusion)

---

## 🧠 What is GraphQL?

**GraphQL** is a query language and runtime for APIs that gives clients the power to ask exactly what they need—nothing more, nothing less.

### ✅ Features
- Ask for specific fields
- Strongly-typed schemas
- Fetch multiple related resources in one request
- Works over HTTP with a single endpoint

### 🔍 Sample Query
```graphql
{
  books {
    title
    author
  }
}
```

### 🧾 Sample Response
```json
{
  "data": {
    "books": [
      { "title": "1984", "author": "George Orwell" },
      { "title": "Hamlet", "author": "William Shakespeare" }
    ]
  }
}
```

---

## 🔥 What is Hot Chocolate?

**Hot Chocolate** is a .NET library for building GraphQL servers. It simplifies the creation of GraphQL APIs using C# or .NET Core applications.

### ✅ Features
- Fully supports GraphQL schema
- Code-first and schema-first support
- Banana Cake Pop GraphQL IDE
- Integration with EF Core and ASP.NET Core

### 🛠️ Example Setup
```csharp
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();
```

You define your types and resolvers in C# classes instead of a schema file.

---

## 🗃️ What is Entity Framework Core (EF Core)?

**Entity Framework Core** is an Object-Relational Mapper (ORM) that lets you interact with a database using .NET classes.

### ✅ Features
- Eliminates raw SQL in most cases
- Uses LINQ for querying
- Code-First or Database-First options
- Migrations for schema evolution

### 🧱 Example Model
```csharp
public class Book {
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```

### 📦 DbContext Example
```csharp
public class AppDbContext : DbContext {
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}
    public DbSet<Book> Books { get; set; }
}
```

---

## 🔗 How They Work Together

These three components combine to create a modern API stack:

| Layer             | Role                                       |
|------------------|--------------------------------------------|
| EF Core           | Connects to the database                   |
| Hot Chocolate     | Exposes the data via GraphQL              |
| GraphQL           | Provides the client query interface       |

### 🔄 Flow:
1. EF Core loads data from the database.
2. Hot Chocolate maps C# types to GraphQL schema.
3. GraphQL clients fetch data using flexible queries.

---

## 📂 Complete Example Project Structure
```
GraphQLDemoAPI/
├── Data/
│   └── AppDbContext.cs
├── GraphQL/
│   └── Query.cs
├── Models/
│   └── Book.cs
├── Program.cs
└── appsettings.json
```

---

## 💻 Code Examples

### 1. Model
```csharp
public class Book {
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```

### 2. DbContext
```csharp
public class AppDbContext : DbContext {
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}
    public DbSet<Book> Books { get; set; }
}
```

### 3. GraphQL Query Resolver
```csharp
public class Query {
    [UseDbContext(typeof(AppDbContext))]
    public IQueryable<Book> GetBooks([ScopedService] AppDbContext context) {
        return context.Books;
    }
}
```

### 4. Program.cs (Minimal API)
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddPooledDbContextFactory<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>();

var app = builder.Build();

app.MapGraphQL();
app.Run();
```

### 5. appsettings.json
```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=GraphQLDemo;Trusted_Connection=True;"
  }
}
```

---

## 🧰 Tools and Resources

| Tool               | Purpose                                    |
|--------------------|--------------------------------------------|
| Hot Chocolate      | GraphQL server framework for .NET          |
| EF Core            | ORM for database interactions              |
| Banana Cake Pop    | Built-in GraphQL playground for testing    |
| SQL Server/PostgreSQL | Supported backends                     |
| Visual Studio / VS Code | IDEs for development                |

---

## ✅ Conclusion

By combining **GraphQL**, **Hot Chocolate**, and **Entity Framework Core**, you can build highly efficient and scalable APIs with minimal boilerplate. This stack is especially useful for modern .NET applications that need flexibility in data fetching and strong typing end-to-end.

Feel free to expand this project with:
- Mutations for create/update/delete
- Authorization
- Filtering, sorting, pagination

---

### 📚 Further Reading
- [Hot Chocolate Docs](https://chillicream.com/docs/hotchocolate)
- [GraphQL Official Site](https://graphql.org/)
- [EF Core Docs](https://learn.microsoft.com/en-us/ef/core/)

