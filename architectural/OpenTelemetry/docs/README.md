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

  - Agregar OTLP/gRPC Service
 
  - Agregar OTLP/HTTP

    > **NOTE**
    > <br />OTLP/HTTP utiliza cargas útiles de Protobuf codificadas en formato binario o en formato JSON.
