---
layout: post
title:  "Let's try GraphQL API with Hot-Chocolate"
date:   2023-04-21 03:46:00 +0530
categories: jekyll update
published: true
---
<div style="text-align: center;">
  <img src="\images\2023-04-21-try-graphql-with-hot-chocolate\Title_Image.jpg" style="width: 500px; height: auto;">
</div>

---

<br>
<br>

The official website of [graphQL](https://graphql.org){:target="_blank"} says:
> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data

***Send a GraphQL query to your API and get exactly what you need, nothing more and nothing less. GraphQL queries always return predictable results. Apps using GraphQL are fast and stable because they control the data they get, not the server.***

I borrowed from the official website, the pictorial representation of a graphQL query and response.

<div style="text-align: center;">
  <img src="\images\2023-04-21-try-graphql-with-hot-chocolate\graphql-example.gif" style="width: 500px; height: auto;">
</div>

---
Compared with a REST API, graphQL API adapts with the client code and data, and gives exactly what it is asked for.

That reduces the graphQL API endpoint to only **one** endpoint to query what we want.

Excited to explore more?

Now a days, I explore my experiments mostly in .net Microsoft stack.

So, just jump in and create a project using Visual Studio.

> **Prerequisites**  
>Visual Studio 2022  
>Dotnet 7 Runtime

Create an API project using Visual Studio

---
  
<br>
 
![image](\images\2023-04-21-try-graphql-with-hot-chocolate\Create API Project.png){: width=600px }

<br>

Additional Info. 

---
  
<br>

![image](\images\2023-04-21-try-graphql-with-hot-chocolate\Additional Info.png)

You may remove the default `WeatherForecast` controller; That's not what we are going to do today 😉

We are trying to model, `Departments`, its `Responsibilities` and the `Managers` involved.

---
  
<br>

First and foremost create 3 interfaces in an Interfaces folder.
```csharp
namespace GraphQLDemo.Interfaces
{
    public interface IDepartmentRepository
    {
    }
}


```

```csharp
namespace GraphQLDemo.Interfaces
{
    public interface IManagerRepository
    {
    }
}


```

```csharp
namespace GraphQLDemo.Interfaces
{
    public interface IResponsibilityRepository
    {
    }
}


```

Similarly, create `Models` folder and the following models:

```csharp
using System.ComponentModel.DataAnnotations;

namespace GraphQLDemo.Models
{
    public class Department
    {
        [Key]
        public Guid Id { get; set; }

        [Required(ErrorMessage = "Department name is required for the department")]
        public string DepartmentName { get; set; }
        public string Description { get; set; }

        public ICollection<Responsibility> Responsibilities { get; set; }
        public Manager Manager { get; set; }
    }
}

```

```csharp
using System.ComponentModel.DataAnnotations.Schema;
using System.ComponentModel.DataAnnotations;

namespace GraphQLDemo.Models
{
    public class Manager
    {
        [Key]
        public Guid Id { get; set; }

        [Required(ErrorMessage = "The manager requires a name")]
        public string Name { get; set; }
        public string Profile { get; set; }

        [ForeignKey("DepartmentId")]
        public Guid DepartmentId { get; set; }
        public Department Department { get; set; }
    }
}

```

```csharp
using System.ComponentModel.DataAnnotations.Schema;
using System.ComponentModel.DataAnnotations;

namespace GraphQLDemo.Models
{
    public class Responsibility
    {
        [Key]
        public Guid Id { get; set; }

        [Required(ErrorMessage = "The bottomline is required")]
        public string BottomLine { get; set; }
        public string Description { get; set; }

        [ForeignKey("DepartmentId")]
        public Guid DepartmentId { get; set; }
        public Department Department { get; set; }
    }
}

```


<div style="text-align: center;">
  <img src="\images\2023-04-21-try-graphql-with-hot-chocolate\Solution after IRepo and Models.png">
</div>
<br>


Time to set up the some Data and Db Context.

Add EF Core package `Microsoft.EntityFrameworkCore`

I am using version `7.0.5`

Let's create a folder `Data` and another subfolder `Configurations`.
And add 3 classes and their codes as follows.

**DepartmentContextConfiguration.cs**

```csharp
using GraphQLDemo.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace GraphQLDemo.Data.Configurations
{
    public class DepartmentContextConfiguration : IEntityTypeConfiguration<Department>
    {
        private Guid[] _ids;
        public DepartmentContextConfiguration(Guid[] ids)
        {
            _ids = ids;
        }
        public void Configure(EntityTypeBuilder<Department> builder)
        {
            builder
                .HasData(
                new Department
                {
                    Id = _ids[0],
                    DepartmentName = "HR",
                    Description = "HR department handles the human resources.",
                },
                new Department
                {
                    Id = _ids[1],
                    DepartmentName = "IT",
                    Description = "IT department handles the technical projects for the clients.",
                },
                new Department
                {
                    Id = _ids[2],
                    DepartmentName = "BD",
                    Description = "BD department handles the business development and business diversifications.",
                });
        }
    
    }
}


```

**ResponsibilityContextConfiguration**

```csharp
using GraphQLDemo.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace GraphQLDemo.Data.Configurations
{
    public class ResponsibilityContextConfiguration : IEntityTypeConfiguration<Responsibility>
    {
        private Guid[] _ids;
        public ResponsibilityContextConfiguration(Guid[] ids)
        {
            _ids = ids;
        }
        public void Configure(EntityTypeBuilder<Responsibility> builder)
        {
            builder
                 .HasData(
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Recruitment.",
                     Description = "They are always a on it.",
                     DepartmentId = _ids[0]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Event Management",
                     Description = "They give the employees the work-life balance.",
                     DepartmentId = _ids[0]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Accessment and Appraisals.",
                     Description = "They keep the employees recognized and self-driven",
                     DepartmentId = _ids[0]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Technical Solutions.",
                     Description = "They solve the problems of clients.",
                     DepartmentId = _ids[1]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Market Due Diligence",
                     Description = "She's good at spying and analyzing the market.",
                     DepartmentId = _ids[2]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Diversification",
                     Description = "She knows how to diversify to take advantage of the changes.",
                     DepartmentId = _ids[2]
                 },
                 new Responsibility
                 {
                     Id = Guid.NewGuid(),
                     BottomLine = "Client Retention",
                     Description = "The rapo and satisfaction of work keeps the client sticked.",
                     DepartmentId = _ids[2]
                 });
        }
    }
}


```
**ManagerContextConfiguration.cs**

```csharp
using GraphQLDemo.Models;
using Microsoft.EntityFrameworkCore;

namespace GraphQLDemo.Data.Configurations
{
    public class ManagerContextConfiguration : IEntityTypeConfiguration<Manager>
    {
        private Guid[] _ids;
        public ManagerContextConfiguration(Guid[] ids) 
        {
            _ids = ids;
        }
        public void Configure(Microsoft.EntityFrameworkCore.Metadata.Builders.EntityTypeBuilder<Manager> builder)
        {
            builder
                .HasData(
                new Manager
                {
                    Id = Guid.NewGuid(),
                    Name = "Batman",
                    Profile = "Batman Begins his journey with fist fight with his enemies. With his immense knowhow he can manage a crowd.",
                    DepartmentId = _ids[0]
                },
               
                new Manager
                {
                    Id = Guid.NewGuid(),
                    Name = "Iron Man",
                    Profile = "Iron Man started his journey in Iron Industries of India. He travelled around the world gaining technical knowledge that no one can beat him in technology and metallergy.",
                    DepartmentId = _ids[1]
                },
               
                new Manager
                {
                    Id = Guid.NewGuid(),
                    Name = "Black Panther",
                    Profile = "Black Panther is seemingly silent but churning from inside. His numbers are so good that he can predict future with a few dice cubes",
                    DepartmentId = _ids[2]
                });
        }
    }
}


```

We can add the `ApplicationDbContext` now inside the `Data` folder.

```csharp
 protected override void OnModelCreating(ModelBuilder builder)
        {
            // Generate three GUIDS and place them in an arrays
            var ids = new Guid[] {Guid.NewGuid(), Guid.NewGuid(), Guid.NewGuid() };

            // Apply configuration for the three contexts in our application
            // This will create the demo data for our GraphQL endpoint.
            builder.ApplyConfiguration(new SuperheroContextConfiguration(ids));
            builder.ApplyConfiguration(new SuperpowerContextConfiguration(ids));
            builder.ApplyConfiguration(new MovieContextConfiguration(ids));
        }

        // Add the DbSets for each of our models we would like at our database
        public DbSet<Department> Departments { get; set; }
        public DbSet<Responsibility> Responsibilities { get; set; }
        public DbSet<Manager> Managers { get; set; }
```

---
<br>


That's it for the data and we have configured our `ApplicationDbContext` along with some data seeded.

---

<br>

Now lets add our repositories.

Create a new folder at root called `Repositories and add 3 new repositories.

```csharp
using GraphQLDemo.Data;

namespace GraphQLDemo.Repositories
{
    public class DepartmentRepository:IDepartmentRepository
    {
        private readonly ApplicationDbContext _dbContext;

        public DepartmentRepository(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }
    }
}


```

```csharp
using GraphQLDemo.Data;
using GraphQLDemo.Interfaces;

namespace GraphQLDemo.Repositories
{
    public class ResponsibilityRepository:IResponsibilityRepository
    {
        private readonly ApplicationDbContext _dbContext;

        public ResponsibilityRepository(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }
    }
}

```

```csharp
using GraphQLDemo.Data;
using GraphQLDemo.Interfaces;

namespace GraphQLDemo.Repositories
{
    public class ManagerRepository:IManagerRepository
    {
        private readonly ApplicationDbContext _dbContext;

        public ManagerRepository(ApplicationDbContext dbContext)
        {
            _dbContext = dbContext;
        }
    }
}

```



![image](\images\2023-04-21-try-graphql-with-hot-chocolate\Solution after Repositories.png)

---
<br>

Now we need to register the services to our application.

But before that, we need to add another Nuget package `Microsoft.EntityFrameworkCore.SqlServer` in order to add the sql server connection along with the dbContext `options` we are passing.

So, now we can add the services

```csharp
// Services for the GraphQL API
builder.Services.AddScoped<IDepartmentRepository, DepartmentRepository>();
builder.Services.AddScoped<IResponsibilityRepository, ResponsibilityRepository>();
builder.Services.AddScoped<IManagerRepository, ManagerRepository>();

// Add Application Db Context options
builder.Services.AddDbContext<ApplicationDbContext>(options =>
options.UseSqlServer(configuration.GetConnectionString("SqlServer")));

```

Now we are ready for creating the migrations with a code-first approach.

But before any further, lets install the packages required ror that
`Microsoft.EntityFrameworkCore.Tools` and `Microsoft.EntityFrameworkCore.Design`.

After successful installation of packages lets run the migration and update the database

---
<br>

![image](\images\2023-04-21-try-graphql-with-hot-chocolate\ScreenRecorderProject9.gif)


---
<br>

Now, we need to create a class `MyQuery` in the `Data` folder, that exposes the required data through the GraphQL. We are gonna expose our `Departments` through our `MyQuery` class.

```csharp
using GraphQLDemo.Models;

namespace GraphQLDemo.Data
{
    public class MyQuery
    {
       public IQueryable<Department> GetDepartments() => new List<Department>().        AsQueryable();
    }
}


```
Next step is to register and expose the graphQL endpoint, that queries the `MyQuery` we have just created.

So let's do that in the `Program.cs`.

```csharp
// Add GraphQL Server
builder.Services.AddGraphQLServer()
                .AddQueryType<MyQuery>();

```

Thanks to `HotChocolate` package we will get a ***Banana Cake Interface*** to query the GraphQL easily.

Lets try that....

Ah....We also need to expose an endpoint for the GraphQl server.

```csharp
app.MapGraphQL("/graphql");

```


In order to make `graphql` your launching default endpoint, change `launchSettings.json`:

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:51063",
      "sslPort": 44333
    }
  },
  "profiles": {
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "graphql", //Change here
      "applicationUrl": "http://localhost:5145",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "https": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "graphql", //Change here
      "applicationUrl": "https://localhost:7196;http://localhost:5145",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "graphql", //Change here
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}

```
Run the application now and check it...

Now you should be able to query the GraphQl through the `Banana Cake Interface`.

![image](\images\2023-04-21-try-graphql-with-hot-chocolate\BananaCake UI.gif)

The better part of it is...our query works...

But we did get only an empty `Department` List.


Let's look back at our querying class `MyQuery` to better understand it.

```csharp
  public class MyQuery
    {
       public IQueryable<Department> GetDepartments() => new List<Department>().        AsQueryable();
    }

```
This explains a lot.

The  `GetDepartments` returns always an empty List of `Department`.

Now let's improve our querying with our `ApplicationDbContext` service.

Update the `MyQuery` with the following code:

```csharp
....
.....
    public class MyQuery
    {
        [UseProjection]
        [UseFiltering]
        [UseSorting]
        public IQueryable<Department> GetDepartments([Service] ApplicationDbContext context) => context.Departments; 
    }
...

```
Also update the GraphQL server registration.

```csharp
// Add GraphQL Server
builder.Services.AddGraphQLServer()
                .AddQueryType<MyQuery>()
                .AddProjections()
                .AddFiltering()
                .AddSorting();
```


Okay...It's time to run the application again.

![image](\images\2023-04-21-try-graphql-with-hot-chocolate\ScreenRecorderProject11.gif)

We can see that we could query the API, with the fields we are interested and we got exactly what we need.

Nothing more nothing less.

Hope this blog helps you to get started with the GraphQL API and build your queries to fullfil your needs.

***Happy coding....***