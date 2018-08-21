---
title: "Boilerplate for ASP.NET Web API"
excerpt: "A starting point for developing Web APIs using modern framework and best practices."
tags: 
    - EntityFramework
    - ASP.NET
    - Web API
    - Dependency Injection
    - Generic Repository
    - .Net Core
    - SQL Server
date:   2018-08-06 00:00:00 -0600
#categories: ASP.NET
---

Several sofware developers asked me the same question before **"Have you ever worked on any projects that started from scratch?"** which is a question I never thought about myself before. I think that is a really good question. 

The reason people asking this kind of question is probably because most of the time software developers first entering the workforce or been working in the industry for a few years; often, they just work along with other people on an existing project, or joining a company that already has all the mature/legacy applications; there is not always the opportunity for every developer to start something from scratch, even if you are working for a startup. 

So, I thought I create a boilerplate project that people can re-use when starting a new projects using ASP.NET. 

The example I am using is a typical Web API application with a database backend. I decided to use ASP.NET Core Web API project in Visual Studio (this is same for ASP.NET Web Application) along with Azure SQL Server Database (DBaaS) for conveniencce. If you do not have Microsoft Azure account which you can obtain from [here](https://azure.microsoft.com/en-us/){:target="_blank"}, or use Microsoft SQL Server Expression edition which is always a good option.

## Create a database project using Visual Studio
The project type is available under `SQL Server` tab. Then, setup folder structure per screenshot below and create the table or stored procedure scripts accordingly. 
![Creating a Web API Project]({{"/assets/images/boilerplate-web-api/DBProject.JPG"}})

Once all the scripts are ready, right-click on the database project and choose *Schema Compare...*, follow the wizard to setup the database connection string. The databse connection can be your local database, or a remote database such as Azure SQL Database service.
Finally click on *Update* to update the target database in SQL Server based on your databse project script.

![Creating a Web API Project]({{"/assets/images/boilerplate-web-api/SQLConnection.JPG"}})

> This is a great way to source control your database project/script.

## Create a Web API Project in Visual Studio
I generally use this naming convention *{Ccompany}.{Component}* or *{Company}.{Context}.{Component}* when creating a new project.
This tutorial focus on `.NET Core 2.0` per screenshot below.
![Creating a Web API Project]({{"/assets/images/boilerplate-web-api/WebAPIProject.JPG"}})

## Create a Data Access Layer (DAL) using .net core library
> Project naming convention {Company}.{Component}.DAL

When creating any libraries, always keep reusability in mind as often a library is shared or referenced by mulitple application. Ideally, the library should be published to a private nuget store or [nuget.org](https://www.nuget.org/){:target="_blank"} if it is generic enough. I will try to produce a blog in the near future how to publish librarys into nuget in VSTS. 

#### Add nuget packages to the DAL project

```bash
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Microsoft.VisualStudio.Web.CodeGeneration.Design
```

After the packages are added, create a new folder *Models* in the DAL lib project.
Run this command in the Package Manager console, ensure the `Data Source` and `Initial Catalog` is updated accordingly. This will create the models based on the existing database.
```bash
Scaffold-DbContext "Data Source=.;Initial Catalog=Temp;integrated security=True;MultipleActiveResultSets=true" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```
I am a fan of using generic repository pattern to communicate with database for reasons. 
1. Loose coupling with the backend database or persistent storage with an abstraction layer
2. Increases the testability
3. I'm lazy and do not want to create a whole lot of corresponding Repository classes, so using generic is much eaiser

Go to Web API Project `Startup.cs` class, replace `ConfigureServices` method with the code below. This basically uses the out-of-box dependency injection framework of .net core.
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    //Please note, this connection should be coming from appsettings.json, not being hardcoded.
    var connection = @"Server=.;Database=Temp;Trusted_Connection=True;ConnectRetryCount=0";
    services.AddEntityFrameworkSqlServer()
                .AddDbContext<TempContext>((serviceProvider, options) => options.UseSqlServer(connection)
                .UseInternalServiceProvider(serviceProvider));

    services.AddScoped<IRepository<Customer>, Repository<Customer>>();
}
```

Create a new `CustomerController` in the Web API Project.  
```csharp
[Route("api/[controller]")]
public class CustomerController : Controller
{
    private readonly IRepository<Customer> _customerRepository;

    public CustomerController(IRepository<Customer> customerRepository)
    {
        _customerRepository = customerRepository;
    }

    [HttpGet]
    public IEnumerable<Customer> Get()
    {
        return _customerRepository.GetAll();
    }
    [HttpGet("{id}")]
    public Customer Get(int id)
    {
        return _customerRepository.GetAll().Where(x => x.Id == id).SingleOrDefault();
    }
}
```
Build and run your project, you should now have a Web API application up and running with a SQL backend database using generic repository pattern 
The complete source code can be found on my github repo [https://github.com/bwwilliam/GenericRepositoryPattern](https://github.com/bwwilliam/GenericRepositoryPattern){:target="blank"}