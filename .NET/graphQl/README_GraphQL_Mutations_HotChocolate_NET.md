# 🚀 GraphQL Mutations in .NET with Hot Chocolate

This guide demonstrates how to implement **GraphQL Mutations** using **.NET** and the **Hot Chocolate** library. Mutations allow clients to modify data (e.g., Create, Update, Delete operations) via the GraphQL API.

---

## 📌 Table of Contents

- [📦 Prerequisites](#-prerequisites)
- [🛠️ Project Setup](#️-project-setup)
- [📐 Define Data Models](#-define-data-models)
- [🔁 Implement Mutations](#-implement-mutations)
- [📥 Create Input Types](#-create-input-types)
- [🧩 Configure GraphQL Server](#-configure-graphql-server)
- [📡 Sample Mutation Queries](#-sample-mutation-queries)
- [📁 Recommended Folder Structure](#-recommended-folder-structure)
- [📚 References](#-references)

---

## 📦 Prerequisites

Before starting, ensure you have the following:

- [.NET 7 SDK+](https://dotnet.microsoft.com/en-us/download)
- [Hot Chocolate](https://chillicream.com/)
- [Entity Framework Core](https://learn.microsoft.com/en-us/ef/core/)
- Basic understanding of C# and GraphQL

---

## 🛠️ Project Setup

### 1. Create a new project

```bash
dotnet new web -n GraphQLDemo
cd GraphQLDemo
```

### 2. Add required packages

```bash
dotnet add package HotChocolate.AspNetCore
dotnet add package HotChocolate.Data.EntityFramework
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

---

## 📐 Define Data Models

Create a `Course` class.

```csharp
// Models/Course.cs
namespace GraphQLDemo.Models
{
    public class Course
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Subject { get; set; }
        public Guid InstructorId { get; set; }
    }
}
```

---

## 🔁 Implement Mutations

Mutations define write operations like `createCourse`, `updateCourse`, and `deleteCourse`.

```csharp
// GraphQL/Mutations/CourseMutation.cs
using GraphQLDemo.Data;
using GraphQLDemo.Models;
using GraphQLDemo.Types;
using Microsoft.EntityFrameworkCore;

namespace GraphQLDemo.GraphQL.Mutations
{
    [ExtendObjectType(OperationTypeNames.Mutation)]
    public class CourseMutation
    {
        public async Task<Course> CreateCourseAsync(CourseInput input, [Service] AppDbContext context)
        {
            var course = new Course
            {
                Name = input.Name,
                Subject = input.Subject,
                InstructorId = input.InstructorId
            };

            context.Courses.Add(course);
            await context.SaveChangesAsync();
            return course;
        }

        public async Task<Course?> UpdateCourseAsync(int id, CourseInput input, [Service] AppDbContext context)
        {
            var course = await context.Courses.FindAsync(id);
            if (course == null) return null;

            course.Name = input.Name;
            course.Subject = input.Subject;
            course.InstructorId = input.InstructorId;

            await context.SaveChangesAsync();
            return course;
        }

        public async Task<bool> DeleteCourseAsync(int id, [Service] AppDbContext context)
        {
            var course = await context.Courses.FindAsync(id);
            if (course == null) return false;

            context.Courses.Remove(course);
            await context.SaveChangesAsync();
            return true;
        }
    }
}
```

---

## 📥 Create Input Types

Input types are used to accept structured data in GraphQL mutations.

```csharp
// GraphQL/Types/CourseInput.cs
namespace GraphQLDemo.Types
{
    public class CourseInput
    {
        public string Name { get; set; } = default!;
        public string Subject { get; set; } = default!;
        public Guid InstructorId { get; set; }
    }
}
```

---

## 🧩 Configure GraphQL Server

Set up EF Core and Hot Chocolate in `Program.cs`.

```csharp
// Program.cs
using GraphQLDemo.Data;
using GraphQLDemo.GraphQL.Mutations;
using GraphQLDemo.Types;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseInMemoryDatabase("CourseDb"));

builder.Services
    .AddGraphQLServer()
    .AddMutationType<CourseMutation>()...