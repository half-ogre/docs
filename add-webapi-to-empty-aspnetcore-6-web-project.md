# Add WebAPI to Empty ASP.NET Core 6 Web Project

Here are the steps to add an API controller to an empty ASP.NET Core 6 web project.

## 0. Create Empty Project
If starting from scratch:

```
dotnet new web -o <project-name>
```

## 1. Invoke `AddControllers`
Invoke the `AddControllers` extension method on the `builder`'s services collection prior to invoking `builder.Build`:

```
var builder = WebApplication.CreateBuilder(args);

// ...

builder.Services.AddControllers();

// ...

var app = builder.Build();
```

## 2. Invoke `UseHttpsRedirection`, `UseAuthorization`, and `MapControllers`
Invoke the `UseHttpsRedirection`, `UseAuthorization`, and `MapControllers` extension methods on the `app` after invoking `builder.Build()` and before invoking `app.Run()`:

```
var app = builder.Build();

// ...

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

// ...

app.Run();
```

## 3. Add an API Controller

And a new controller class decorated with the `[ApiController]` attribute. Example:

```
[ApiController]
public class ServiceStateController : ControllerBase
{
    private readonly ILogger<ServiceStateController> _logger;

    public ServiceStateController(ILogger<ServiceStateController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    [Route("health-state")]
    public IActionResult Get()
    {
        // ...

        return StatusCode(200);
    }
}
```

## 4. Optional: Add Swagger Docs

Add the `Swashbuckle.AspNetCore` NuGet package from the command-line:

```
dotnet add package Swashbuckle.AspNetCore
```

Invoke the `AddEndpointsApiExplorer` and `AddSwaggerGen` extension methods on the `builder`'s services collection prior to invoking `builder.Build`:

```
var builder = WebApplication.CreateBuilder(args);

// ...

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// ...

var app = builder.Build();
```

Invoke the `UseSwagger` and `UseSwaggerUI` extension methods for the development environment on the `app`, after invoking `builder.Build()` and before invoking `app.Run()`:

```
var app = builder.Build();

// ...

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

// ...

app.Run();
```
