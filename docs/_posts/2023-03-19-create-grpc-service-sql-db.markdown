---
layout: post
title:  "Create a gRPC service for sql server db with database-first approach"
date:   2023-03-19 22:12:00 +0530
categories: jekyll update
# published: false
---

We will look at how to create a gRpc service that connects to a sql database and perform the CRUD operations on it

> **Prerequisites**  
>Visual Studio 2022  
>Dotnet 7 Runtime

Lets create a new gRpc service project using Visual Studio   
![image](\images\2023-03-19-create-grpc-service-sql-db\Create Project 2023-03-19 215236.png){: width="400" }

The solution would like this:    
![image](\images\2023-03-19-create-grpc-service-sql-db\Solution Initial 2023-03-19 215715.png)

By default, gRPC uses Protocol Buffers, Google’s mature open source mechanism for serializing structured data (although it can be used with other data formats such as JSON). Here’s a quick intro to how it works.

So we have to create a `products.proto` file that defines the gRpc service`ProductsService` with the data structure for exchanging product data between the clients(we will create the clients later).

The proto file would look like this.



```go

syntax = "proto3";

option csharp_namespace = "GrpcService1";
package products;
service ProductsService {
    rpc GetAll(Empty) returns(Products);
    rpc GetById(ProductRowIdFilter) returns(Product);
    rpc Post(Product) returns(Product);
    rpc Put(Product) returns(Product);
    rpc Delete(ProductRowIdFilter) returns(Empty);
}
message Empty {}
message Product {
    int32 ProductRowId = 1;
    string ProductId = 2;
    string ProductName = 3;
    string CategoryName = 4;
    string Manufacturer = 5;
    int32 Price = 6;
}
message ProductRowIdFilter {
    int32 ProductRowId = 1;
}
message Products {
    repeated Product items = 1;
}
```
We have to include the package reference for 

```csharp
 <PackageReference Include="Grpc.AspNetCore" Version="2.49.0" />
```

and include the `Protobuf` in the `ItemGroup` tag
```csharp
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
    <Protobuf Include="Protos\products.proto" GrpcServices="Server" />
  </ItemGroup>
```
Now build the project and right click the `products.proto` file and check the properties shows:
![image](\images\2023-03-19-create-grpc-service-sql-db\gRpc proto properties 2023-03-19 222431.png)

Now we have to get ready with our `DbContext` and `Models` to interact with our sql server.
We are going to use the `Databse First` approach and use EntityFramework Core as the ORM.

So get ready with package references as the normal practice.
```csharp
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="7.0.4" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="7.0.4">
 <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="7.0.4" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer.Design" Version="1.1.6" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="7.0.4">
```
We are using the local db for simplicity and going to run the SQL script as follows on the localdb using SSMS.

```sql
Create Database Blazor
USE [Blazor]
CREATE TABLE [dbo].[products]
  (
     [productrowid] [INT] IDENTITY(1, 1) NOT NULL,
     [productid]    [VARCHAR](20) NOT NULL,
     [productname]  [VARCHAR](200) NOT NULL,
     [categoryname] [VARCHAR](200) NOT NULL,
     [manufacturer] [VARCHAR](200) NOT NULL,
     [price]        [INT] NOT NULL,
     PRIMARY KEY CLUSTERED ( [productrowid] ASC )WITH (pad_index = OFF,
     statistics_norecompute = OFF, ignore_dup_key = OFF, allow_row_locks = on,
     allow_page_locks = on, optimize_for_sequential_key = OFF) ON [PRIMARY],
     UNIQUE NONCLUSTERED ( [productid] ASC )WITH (pad_index = OFF,
     statistics_norecompute = OFF, ignore_dup_key = OFF, allow_row_locks = on,
     allow_page_locks = on, optimize_for_sequential_key = OFF) ON [PRIMARY]
  )
ON [PRIMARY]
go
```
Its time to run some `dotnet-ef` commands in the terminal to create our `DbContext` and `Models` with Db-First approach.

```powershell
dotnet tool install --global dotnet-ef --version 7.0.4
dotnet ef dbcontext scaffold "Data Source=.;Initial Catalog=Blazor;Integrated Security=SSPI;Encrypt=False;TrustServerCertificate=False;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```
Please make sure to run the commands from the folder with the project file.

If everything goes well you will now see your `Models` and `BlazorContext` in the a folder Models.
![image](\images\2023-03-19-create-grpc-service-sql-db\Models and Context 2023-03-19 224025.png)

Now in the Services folder add a new class file and name it as ProductsAppService.cs.

Here is the full code of the “ProductsAppService.cs” class

```csharp
using Grpc.Core;
using GrpcService;
using GrpcService.Models;
using System.Linq;
using System.Threading.Tasks;
using static GrpcService.ProductsService;
namespace GrpcService.Services
{
    public class ProductsAppService : ProductsServiceBase
    {
        public BlazorContext dbContext;
        public ProductsAppService(BlazorContext DBContext)
        {
            dbContext = DBContext;
        }
        #region GetAll
        public override Task<Products> GetAll(Empty request, ServerCallContext context)
        {
            Products response = new Products();
            var products = from prd in dbContext.Products
                           select new Product()
                           {
                               ProductRowId = prd.ProductRowId,
                               ProductId = prd.ProductId,
                               ProductName = prd.ProductName,
                               CategoryName = prd.CategoryName,
                               Manufacturer = prd.Manufacturer,
                               Price = prd.Price
                           };
            response.Items.AddRange(products.ToArray());
            return Task.FromResult(response);
        }
        #endregion
        #region GetById
        public override Task<Product> GetById(ProductRowIdFilter request, ServerCallContext context)
        {
            var product = dbContext.Products.Find(request.ProductRowId);
            var searchedProduct = new Product()
            {
                ProductRowId = product.ProductRowId,
                ProductId = product.ProductId,
                ProductName = product.ProductName,
                CategoryName = product.CategoryName,
                Manufacturer = product.Manufacturer,
                Price = product.Price
            };
            return Task.FromResult(searchedProduct);
        }
        #endregion
        #region PostInsert
        public override Task<Product> Post(Product request, ServerCallContext context)
        {
            var prdAdded = new Models.Product()
            {
                ProductId = request.ProductId,
                ProductName = request.ProductName,
                CategoryName = request.CategoryName,
                Manufacturer = request.Manufacturer,
                Price = request.Price
            };
            var res = dbContext.Products.Add(prdAdded);
            dbContext.SaveChanges();
            var response = new Product()
            {
                ProductRowId = res.Entity.ProductRowId,
                ProductId = res.Entity.ProductId,
                ProductName = res.Entity.ProductName,
                CategoryName = res.Entity.CategoryName,
                Manufacturer = res.Entity.Manufacturer,
                Price = res.Entity.Price
            };
            return Task.FromResult<Product>(response);
        }
        #endregion
        #region PUTUPDATE
        public override Task<Product> Put(Product request, ServerCallContext context)
        {
            Models.Product prd = dbContext.Products.Find(request.ProductRowId);
            if (prd == null)
            {
                return Task.FromResult<Product>(null);
            }
            prd.ProductRowId = request.ProductRowId;
            prd.ProductId = request.ProductId;
            prd.ProductName = request.ProductName;
            prd.CategoryName = request.CategoryName;
            prd.Manufacturer = request.Manufacturer;
            prd.Price = request.Price;
            dbContext.Products.Update(prd);
            dbContext.SaveChanges();
            return Task.FromResult<Product>(new Product()
            {
                ProductRowId = prd.ProductRowId,
                ProductId = prd.ProductId,
                ProductName = prd.ProductName,
                CategoryName = prd.CategoryName,
                Manufacturer = prd.Manufacturer,
                Price = prd.Price
            });
        }
        #endregion
        #region DELETE
        public override Task<Empty> Delete(ProductRowIdFilter request, ServerCallContext context)
        {
            Models.Product prd = dbContext.Products.Find(request.ProductRowId);
            if (prd == null)
            {
                return Task.FromResult<Empty>(null);
            }
            dbContext.Products.Remove(prd);
            dbContext.SaveChanges();
            return Task.FromResult<Empty>(new Empty());
        }
        #endregion
    }
}
```
As a final step register your gRPC service endpoints and dbContext in the `Program.cs`

```csharp
 builder.Services.AddDbContext<BlazorContext>();
....

app.MapGrpcService<GreeterService>();
app.MapGrpcService<ProductsAppService>();
```
Start your application. If you did everything right you would be welcomed with a console of a gRPC service hosted successfully.
![image](\images\2023-03-19-create-grpc-service-sql-db\gRPC Service Serving 2023-03-19 224957.png)

***Happy coding....***