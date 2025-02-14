# Aspire.Dashboard

## Step by step

### Step 1. Agregar soporte para gRCP 
- Agregar referencia a gRPC al projecto
  
  ```xml
  <PackageReference Include="Grpc.AspNetCore" Version="2.67.0" />
  ```

- Agregar servicio a gRPC al injector de dependencias
  
  ```csharp
  // Add services to the container.
  services.AddGrpc();
  ```

  ---

  ### Step 2. Agregar [OpenTelemetry Protocol Specification](https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#opentelemetry-protocol-specification)

  - Agregar el [Protocol Buffers schema](https://github.com/open-telemetry/opentelemetry-proto/tree/main/opentelemetry/proto)
    
    ```
    Aspire.Dashboard.csproj
    Otlp/
    └── opentelemetry/
        └── proto/
            └── collector/
            └── common/
            └── logs/
            └── metrics/
            └── profiles/
            └── resource/
            └── trace/
    ```
    
  - Carga los archivos Protobuf definitions en el projecto
    
    ```xml
    <ItemGroup>
    	<!-- Build service and client types. Integration tests use the client types to call OTLP services. -->
    	<Protobuf Include="Otlp\**\*.proto">
    		<ProtoRoot>Otlp</ProtoRoot>
    	</Protobuf>
    </ItemGroup>
    ```

  - Crear OTLP/gRPC Service
 
  - Crear OTLP/HTTP

    > **NOTE**
    > <br />OTLP/HTTP utiliza cargas útiles de Protobuf codificadas en formato binario o en formato JSON.

  - Agregar OTLP/gRPC y OTLP/HTTP Service al injector de dependencias
    
    ```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            Services = services;
    
            // Add services to the container.
            Services.AddGrpc();
    
            Services.AddRazorComponents()
                .AddInteractiveServerComponents();
    
    
            // OTLP services.
            Services.AddTransient<OtlpLogsService>();
            Services.AddTransient<OtlpTraceService>();
            Services.AddTransient<OtlpMetricsService>();
        }
    
        public void Configure(WebApplication app, IWebHostEnvironment env)
        {
            // Configure the HTTP request pipeline.
            if (!app.Environment.IsDevelopment())
            {
                app.UseExceptionHandler("/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }
    
            app.UseHttpsRedirection();
    
            app.UseAntiforgery();
    
            // +
            app.MapStaticAssets();
            app.MapRazorComponents<App>()
                .AddInteractiveServerRenderMode();
    
            // OTLP HTTP services.
            app.MapHttpOtlpApi(/*dashboardOptions.Otlp*/);
    
            // OTLP gRPC services.
            app.MapGrpcService<OtlpGrpcTraceService>();
            app.MapGrpcService<OtlpGrpcMetricsService>();
            app.MapGrpcService<OtlpGrpcLogsService>();
        }
    ```

---

### Step 3. Habilitar Logger debug

```json
{
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Aspire.Dashboard": "Debug"
        }
    }
}
```

---

### Step 4. Agrega excepciones en .gitignore

```gitignore
!src/Aspire.Dashboard/Otlp/opentelemetry/proto/logs/
!src/Aspire.Dashboard/Otlp/opentelemetry/proto/collector/logs/
```
