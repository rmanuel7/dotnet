# [Instrumentation](https://opentelemetry.io/docs/languages/net/getting-started/#instrumentation)

Next we’ll install the instrumentation [NuGet packages from OpenTelemetry](https://www.nuget.org/profiles/OpenTelemetry) that will generate the telemetry, and set them up.

1. Add the packages
```powershell
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Instrumentation.AspNetCore
dotnet add package OpenTelemetry.Exporter.Console
```

2. Setup the OpenTelemetry code
   
   - In Program.cs, replace the following lines:
     
   ```csharp
   var builder = WebApplication.CreateBuilder(args);
   var app = builder.Build();
   ```

   - With:

    ```csharp
    using OpenTelemetry.Logs;
    using OpenTelemetry.Metrics;
    using OpenTelemetry.Resources;
    using OpenTelemetry.Trace;
    
    var builder = WebApplication.CreateBuilder(args);
    
    const string serviceName = "roll-dice";
    
    builder.Logging.AddOpenTelemetry(options =>
    {
        options
            .SetResourceBuilder(
                ResourceBuilder.CreateDefault()
                    .AddService(serviceName))
            .AddConsoleExporter();
    });
    builder.Services.AddOpenTelemetry()
          .ConfigureResource(resource => resource.AddService(serviceName))
          .WithTracing(tracing => tracing
              .AddAspNetCoreInstrumentation()
              .AddConsoleExporter())
          .WithMetrics(metrics => metrics
              .AddAspNetCoreInstrumentation()
              .AddConsoleExporter());
    
    var app = builder.Build();
    ```
