
# GraphQL in .NET with Hot Chocolate

This guide explains how to work with **Queries**, **Mutations**, and **Subscriptions** in a GraphQL API using **Hot Chocolate** in .NET.

## Table of Contents

- [Introduction](#introduction)
- [Setup](#setup)
- [Queries](#queries)
- [Mutations](#mutations)
- [Subscriptions](#subscriptions)
- [Running the Server](#running-the-server)
- [Sample Client Request](#sample-client-request)

---

## Introduction

**Hot Chocolate** is a GraphQL server implementation for .NET. It allows you to build a strongly typed GraphQL API using C#. It supports queries, mutations, and subscriptions, making it easy to build interactive, real-time applications.

---

## Setup

1. Create a new .NET Web API project:

```bash
dotnet new web -n GraphQLDemo
cd GraphQLDemo
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Subscriptions
dotnet add package HotChocolate.Data.EntityFramework
```

2. Define a model (e.g., Book):

```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
}
```

3. Create a dummy data service:

```csharp
public class BookService
{
    private readonly List<Book> books = new()
    {
        new Book { Id = 1, Title = "C# in Depth", Author = "Jon Skeet" },
        new Book { Id = 2, Title = "Pro ASP.NET Core", Author = "Adam Freeman" }
    };

    public IEnumerable<Book> GetBooks() => books;
    public Book AddBook(Book book)
    {
        book.Id = books.Count + 1;
        books.Add(book);
        return book;
    }
}
```

---

## Queries

Used to fetch data.

```csharp
public class Query
{
    public IEnumerable<Book> GetBooks([Service] BookService service) => service.GetBooks();
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddSingleton<BookService>();
builder.Services.AddGraphQLServer()
    .AddQueryType<Query>();
```

Test in Banana Cake Pop:

```graphql
query {
  books {
    id
    title
    author
  }
}
```

---

## Mutations

Used to modify data.

```csharp
public class Mutation
{
    public Book AddBook(string title, string author, [Service] BookService service)
    {
        return service.AddBook(new Book { Title = title, Author = author });
    }
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>();
```

Mutation query:

```graphql
mutation {
  addBook(title: "New Book", author: "Jane Doe") {
    id
    title
    author
  }
}
```

---

## Subscriptions

Used for real-time updates.

1. Define event sender service and Mutation publishing:

```csharp
public class Mutation
{
    public async Task<Book> AddBook(string title, string author,
        [Service] BookService service,
        [Service] ITopicEventSender sender)
    {
        var book = service.AddBook(new Book { Title = title, Author = author });
        await sender.SendAsync("OnBookAdded", book);
        return book;
    }
}
```

2. Subscription class:

```csharp
public class Subscription
{
    [Subscribe]
    [Topic("OnBookAdded")]
    public Book OnBookAdded([EventMessage] Book book) => book;
}
```

3. Update `Program.cs`:

```csharp
builder.Services
    .AddGraphQLServer()
    .AddQueryType<Query>()
    .AddMutationType<Mutation>()
    .AddSubscriptionType<Subscription>()
    .AddInMemorySubscriptions();
```

4. Add WebSocket middleware:

```csharp
app.UseWebSockets();
app.MapGraphQL();
```

GraphQL subscription:

```graphql
subscription {
  onBookAdded {
    id
    title
    author
  }
}
```

---

## Running the Server

```bash
dotnet run
```

Navigate to `https://localhost:5001/graphql` and open Banana Cake Pop to test queries, mutations, and subscriptions.

---

## Sample Client Request

```bash
curl -X POST https://localhost:5001/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ books { id title author } }"}'
```

---

## Conclusion

This guide demonstrated how to set up and use GraphQL Queries, Mutations, and Subscriptions in a .NET app using Hot Chocolate. You can now build flexible, real-time, strongly typed APIs using .NET.
