
# Setting up Entity Framework Core with Hot Chocolate GraphQL in .NET

This README provides step-by-step instructions on how to integrate **Entity Framework Core** with **Hot Chocolate** to create a GraphQL API in .NET.

## Table of Contents

- [Introduction](#introduction)
- [Project Setup](#project-setup)
- [Database Context and Model](#database-context-and-model)
- [Creating the Database](#creating-the-database)
- [Query Setup](#query-setup)
- [Mutation Setup](#mutation-setup)
- [Registering Services](#registering-services)
- [Running the Application](#running-the-application)
- [GraphQL Testing](#graphql-testing)
- [Conclusion](#conclusion)

---

## Introduction

Entity Framework Core (EF Core) is an Object-Relational Mapper (ORM) for .NET. Hot Chocolate is a modern GraphQL server for .NET. Together, they allow efficient access to databases via GraphQL.

---

## Project Setup

1. Create a new Web API project:

```bash
dotnet new web -n GraphQLEFCoreDemo
cd GraphQLEFCoreDemo
```

2. Add required packages:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Data.EntityFramework
```

---

## Database Context and Model

### Book Model

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```

### AppDbContext

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
    public DbSet<Book> Books { get; set; }
}
```

---

## Creating the Database

1. Add connection string in `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BookDb;Trusted_Connection=True;"
  }
}
```

2. Configure `Program.cs`:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

3. Create migration and database:

```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

## Query Setup

```csharp
public class Query
{
    [UseDbContext(typeof(AppDbContext))]
    [UseFiltering]
    [UseSorting]
    public IQueryable<Book> GetBooks([ScopedService] AppDbContext context) =>
        context.Books;
}
```

---

## Mutation Setup

```csharp
public class Mutation
{
    [UseDbContext(typeof(AppDbContext))]
    public async Task<Book> AddBookAsync(string title, string author, [ScopedService] AppDbContext context)
    {
        var book = new Book { Title = title, Author = author };
        context.Books.Add(book);
        await context.SaveChangesAsync();
        return book;
    }
}
```

---

## Registering Services

In `Program.cs`:

```csharp
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddFiltering()
    .AddSorting()
    .AddProjections()
    .AddAuthorization()
    .AddType<Book>();
```

---

## Running the Application

```bash
dotnet run
```

Open [https://localhost:5001/graphql](https://localhost:5001/graphql) and use Banana Cake Pop to test.

---

## GraphQL Testing

### Query:

```graphql
query {
  books {
    id
    title
    author
  }
}
```

### Mutation:

```graphql
mutation {
  addBook(title: "EF Core Guide", author: "MS Docs") {
    id
    title
    author
  }
}
```

---

## Conclusion

This README walked through setting up Entity Framework Core with Hot Chocolate in .NET. You now have a working GraphQL API backed by a SQL Server database, capable of querying and modifying data using EF Core.
