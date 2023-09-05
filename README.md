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