# Web API

## Summary

In this guide, we are going to go through the process of creating an ASP.NET Web API. We will then connect that Web API to a database and host them in Azure.

### Create Web API

Our first step will be to create the Web API project. Open Visual Studio 2017 and select File > New > Project.

Select ASP.NET Core Web application, and name your project Handicard.

In the New ASP.NET Core Web Application dialog, select the API template and click OK.

Run your app to make sure you've got your setup correct.

If you get a dialog box that asks if you should trust the IIS Express certificate, select Yes. In the Security Warning dialog that appears next, select Yes.

The default values API controller GET method is called, and should return the following JSON

```JSON
["value1","value2"]
```

Right click your solution, Add > New Folder. Name the folder _Models_.

### Create First Models

Right click the _Models_ folder and select Add > Class.  Name the class Hole and select Add.

Replace the contents of the file with the following code:

```c#
namespace Handicard.Models
{
    public class Hole
    {
        public long Id { get; set; }
        public long CourseId { get; set; }
        public int HoleNumber { get; set; }
        public int Par { get; set; }
        public int StrokeIndex { get; set; }
    }
}
```

Right click the _Models_ folder and select Add > Class.  Name the class Course and select Add.

Replace the contents of the file with the following code:

```c#
using System.Collections.Generic;

namespace Handicard.Models
{
    public class Course
    {
        public long Id { get; set; }
        public string Name { get; set; }
        public ICollection<Hole> Holes { get; set; }
    }
}
```

### Create DbContext

Right click the _Models_ folder and select Add > Class. Name the class HandicardContext and click Add.

Replace the template code with the following:

```c#
using Microsoft.EntityFrameworkCore;

namespace Handicard.Models
{
    public class HandicardContext : DbContext 
    {
        public HandicardContext(DbContextOptions<TodoContext> options) : base(options)
        {

        }
        
        public DbSet<Course> Courses { get; set; }
    }
}
```

### Register the context with dependency injection

Open the appsettings.Development.json file in your project. Here is where you want to define your connection string for you database.

```json
{
"Logging": {
    "LogLevel": {
        "Default": "Debug",
        "System": "Information",
        "Microsoft": "Information"
        }
    },
    "ConnectionStrings": {
        "HandicardContext": "Server=(localdb)\\mssqllocaldb;Database=Handicard;Trusted_Connection=True;ConnectRetryCount=0"
    }
}
```

Defining the connection string here allows it to be set by an environment variable when it comes to publishing your application to a live environment.

Open the Startup.cs file in your project.

Add the following code to the `ConfigureServices` method:

```c#
var connection = Configuration.GetConnectionString("HandicardContext");
services.AddDbContext<BloggingContext>
    (options => options.UseSqlServer(connection));
```

### Create the database

Tools > NuGet Package Manager  > Package Manager Console

Run the following commands:

```
Add-Migration InitialCreate
Update-Database
```

After creating the database, select View > SQL Server Object Explorer. Right click SQL Server > Refresh. Expand the (localdb)\MSSQLLocalDB and you should see a database named Handicard. When you expand the database you should see two tables, for Courses and Hole.

### Add API Controller

Right click the _Controllers_ folder.

Select Add  New Item.

In the Add New Item dialog, select the API Controller Class template.

Name the class _CourseController_, and select Add.

Replace the template code with the following 

```c#
using System.Collections.Generic;
using System.Threading.Tasks;
using Handicard.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

// For more information on enabling Web API for empty projects, visit https://go.microsoft.com/fwlink/?LinkID=397860

namespace HandicardApi.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CoursesController : ControllerBase
    {
        private readonly HandicardContext _context;

        public CoursesController(HandicardContext context)
        {
            _context = context;
        }

        // GET: api/courses
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Course>>> Get()
        {
            return await _context.Courses.Include(c => c.Holes).ToListAsync();
        }

        // GET api/courses/5
        [HttpGet("{id}")]
        public async Task<ActionResult<Course>> Get(long id)
        {
            var course = await _context.Courses.Include(c => c.Holes).FirstOrDefaultAsync(c => c.Id == id);

            if (course == null)
            {
                return NotFound();
            }

            return course;
        }

        // POST api/<controller>
        [HttpPost]
        public async Task<ActionResult<Course>> Post(Course course)
        {
            _context.Courses.Add(course);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(Get), new { id = course.Id }, course);
        }
    }
}
```

We've created a controller that exposes 3 functions, GET, GET{id} and POST. These 3 functions allow us to create new courses and retrieve existing ones in the database.

### Publish your API and DB



## Setup Azure Account


