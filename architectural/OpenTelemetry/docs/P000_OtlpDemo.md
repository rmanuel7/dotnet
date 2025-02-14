# OtlpAPI

## Step by step

### Step 1. Agregar referencias OpenTelemetry al projecto

```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="9.0.0" />
    <PackageReference Include="OpenTelemetry.Exporter.OpenTelemetryProtocol" Version="1.11.1" />
    <PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.11.1" />
    <PackageReference Include="OpenTelemetry.Instrumentation.AspNetCore" Version="1.11.0" />
    <PackageReference Include="OpenTelemetry.Instrumentation.Http" Version="1.11.0" />
    <PackageReference Include="OpenTelemetry.Instrumentation.Runtime" Version="1.11.0" />
  </ItemGroup>
```

### Step 2. Agregar servicio de OpenTelemetry al contenedor de dependencias

```csharp
// Adding observability code to an app yourself.
builder.Logging.AddOpenTelemetry(configure: logging =>
{
    logging.IncludeScopes = true;
    logging.IncludeFormattedMessage = true;
});

builder.Services.AddOpenTelemetry()
    .WithTracing(configure: static tracing =>
    {
        tracing.AddHttpClientInstrumentation();
        tracing.AddAspNetCoreInstrumentation();
    })
    .WithMetrics(configure: static metrics =>
    {
        metrics.AddHttpClientInstrumentation();
        metrics.AddAspNetCoreInstrumentation();
        metrics.AddRuntimeInstrumentation();
    })
    .UseOtlpExporter();
```

### Step 3. Configurar las variables de entorno

```json
{
    "profiles": {
        "http": {
            "environmentVariables": {
                "OTEL_EXPORTER_OTLP_ENDPOINT": "http://localhost:5130",
                "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc"
            }
        },
        "https": {
            "environmentVariables": {
                "OTEL_EXPORTER_OTLP_ENDPOINT": "https://localhost:7173",
                "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc"
            }
        }
    }
}
```
