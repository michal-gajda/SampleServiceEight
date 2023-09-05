# SampleService in .NET 8

```pwsh
mkdir -p SampleService/src
cd SampleService
dotnet new gitignore
git init
```

## Solution
```pwsh
dotnet new sln --name SampleService
```

## WebApi
```pwsh
dotnet new webapi --name SampleService.WebApi
dotnet sln add SampleService.WebApi
```
### nugets for WebApi
```pwsh
dotnet add SampleService.WebApi package Microsoft.Extensions.Hosting.WindowsServices --version 8.0.0-preview.7.23375.6
```
### Expected file contents
```csharp
using SampleService.Application;
using SampleService.Infrastructure;


var webApplicationOptions = new WebApplicationOptions
{
    ContentRootPath = AppContext.BaseDirectory,
    Args = args,
};

var builder = WebApplication.CreateBuilder(webApplicationOptions);

builder.Host.UseWindowsService(options => options.ServiceName = ServiceConstants.ServiceName);

// Add services to the container.
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);

// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi();

await app.RunAsync();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>

  <ItemGroup>
    <Using Include="MediatR" />
    <Using Include="SampleService.Application.Common.Shared" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="8.0.0-preview.7.23375.9" />
    <PackageReference Include="Microsoft.Extensions.Hosting.WindowsServices" Version="8.0.0-preview.7.23375.6" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.5.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\SampleService.Infrastructure\SampleService.Infrastructure.csproj" />
  </ItemGroup>

</Project>
```
## Infrastructure
```pwsh
dotnet new classlib --name SampleService.Infrastructure
dotnet sln add SampleService.Infrastructure
dotnet add SampleService.WebApi reference SampleService.Infrastructure
```
### nugets for Infrastructure
```pwsh
dotnet add SampleService.Infrastructure package Microsoft.Extensions.Configuration.Abstractions --version 8.0.0-preview.7.23375.6
dotnet add SampleService.Infrastructure package Microsoft.Extensions.Configuration.Binder --version 8.0.0-preview.7.23375.6
dotnet add SampleService.Infrastructure package Microsoft.Extensions.Options.ConfigurationExtensions --version 8.0.0-preview.7.23375.6
```
### Expected file contents
```csharp
namespace SampleService.Infrastructure;

using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
    {
        var assembly = System.Reflection.Assembly.GetExecutingAssembly();
        services.AddMediatR(cfg =>
        {
            cfg.RegisterServicesFromAssembly(assembly);
        });

        return services;
    }
}
```
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <Using Include="MediatR" />
    <Using Include="Microsoft.Extensions.Logging" />
  </ItemGroup>

  <ItemGroup>
    <InternalsVisibleTo Include="SampleService.Infrastructure.UnitTests" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Configuration.Abstractions" Version="8.0.0-preview.7.23375.6" />
    <PackageReference Include="Microsoft.Extensions.Configuration.Binder" Version="8.0.0-preview.7.23375.6" />
    <PackageReference Include="Microsoft.Extensions.Options.ConfigurationExtensions" Version="8.0.0-preview.7.23375.6" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\SampleService.Application\SampleService.Application.csproj" />
  </ItemGroup>

</Project>
```

## Application
```pwsh
dotnet new classlib --name SampleService.Application
dotnet sln add SampleService.Application
dotnet add SampleService.Infrastructure reference SampleService.Application
```
### nugets for Application
```pwsh
dotnet add SampleService.Application package MediatR
dotnet add SampleService.Application package Microsoft.Extensions.Logging.Abstractions --version 8.0.0-preview.7.23375.6
```
### Expected file contents
```csharp title="DependencyInjection.cs"
namespace SampleService.Application;

using Microsoft.Extensions.DependencyInjection;

public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        var assembly = System.Reflection.Assembly.GetExecutingAssembly();
        services.AddMediatR(cfg =>
        {
            cfg.RegisterServicesFromAssembly(assembly);
        });

        return services;
    }
}
```
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <Using Include="MediatR" />
    <Using Include="Microsoft.Extensions.Logging" />
    <Using Include="System.ComponentModel.DataAnnotations" />
    <Using Include="System.Text.Json.Serialization" />
  </ItemGroup>

  <ItemGroup>
    <InternalsVisibleTo Include="SampleService.Application.UnitTests" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="MediatR" Version="12.1.1" />
    <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="8.0.0-preview.7.23375.6" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\SampleService.Domain\SampleService.Domain.csproj" />
  </ItemGroup>

</Project>
```
## Domain
```pwsh
dotnet new classlib --name SampleService.Domain
dotnet sln add SampleService.Domain
dotnet add SampleService.Application reference SampleService.Domain
```