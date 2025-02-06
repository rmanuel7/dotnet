
## [Configure OTLP HTTP with standalone dashboard](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/enable-browser-telemetry?tabs=bash#configure-otlp-http-with-standalone-dashboard)
If the dashboard is used standalone, without the rest of .NET Aspire, the OTLP HTTP endpoint is enabled by default on port `18890`. However, the port must be mapped when the dashboard container is started:

```yml
  monitoring.ui:
    image: mcr.microsoft.com/dotnet/aspire-dashboard:9.0
    container_name: Aspire
    ports:
      - 18888:18888 # UI del dashboard
      - 4317:18889 # OTLP using gRPC para recibir datos
      - 4318:18890 # OTLP using Protobuf over HTTP para recibir datos
      - 18891:18891 # The dashboard connects to a resource service to load and display resource information. .
    environment:
      - ASPNETCORE_URLS=http://*:18888
      - ASPIRE_ALLOW_UNSECURED_TRANSPORT=true
      - DOTNET_DASHBOARD_OTLP_ENDPOINT_URL=http://*:18889
      - DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL=http://*:18890
      - DASHBOARD__RESOURCE__ENABLED=true
      - DOTNET_ASPIRE_SHOW_DASHBOARD_RESOURCES=true
      # - DOTNET_RESOURCE_SERVICE_ENDPOINT_URL=http://localhost:18891
      # - Dashboard__ResourceServiceClient__Url=http://localhost:18891
      - Dashboard__ResourceServiceClient__AuthMode=ApiKey
      - Dashboard__ResourceServiceClient__ApiKey=llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA
      - Dashboard__Otlp__AuthMode=ApiKey
      - Dashboard__Otlp__PrimaryApiKey=llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA

  web.mvc:
    environment:
      - OTEL_EXPORTER_OTLP_PROTOCOL=grpc
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://monitoring.ui:4317
      - OTEL_EXPORTER_OTLP_HEADERS=x-otlp-api-key=llr_-GOLspb476TVjQzlrnXNV9ovg9Oyg4sXKCNJA
```


```powershell
info: Aspire.Dashboard.DashboardWebApplication[0]
      Now listening on: http://[::]:18888
info: Aspire.Dashboard.DashboardWebApplication[0]
      Login to the dashboard at http://localhost:18888/login?t=e4eb3d98ba779ff72614988e1e9455dc. The URL may need changes depending on how network access to the container is configured.

info: Aspire.Dashboard.DashboardWebApplication[0]
      OTLP/gRPC listening on: http://[::]:18889

info: Aspire.Dashboard.DashboardWebApplication[0]
      OTLP/HTTP listening on: http://[::]:18890
```

# [Secure telemetry endpoint](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/security-considerations?tabs=bash#secure-telemetry-endpoint)

API key authentication can be enabled on the telemetry endpoint with some additional configuration:

```PowerShell
docker run --rm -it -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
    -e DASHBOARD__OTLP__AUTHMODE='ApiKey' \
    -e DASHBOARD__OTLP__PRIMARYAPIKEY='{MY_APIKEY}' \
    mcr.microsoft.com/dotnet/aspire-dashboard:9.0
```

The preceding Docker command:

- Starts the .NET Aspire dashboard image and exposes OTLP endpoint as port 4317
- Configures the OTLP endpoint to use `ApiKey` authentication. This requires that incoming telemetry has a valid `x-otlp-api-key` header value.
- Configures the expected API key. `{MY_APIKEY}` in the example value should be replaced with a real API key. The API key can be any text, but a value with at least 128 bits of entropy is recommended.

When API key authentication is configured, the dashboard validates incoming telemetry has a required API key. Apps that send the dashboard telemetry must be configured to send the API key. This can be configured in .NET with `OtlpExporterOptions.Headers`:

```csharp
builder.Services.Configure<OtlpExporterOptions>(
    o => o.Headers = $"x-otlp-api-key={MY_APIKEY}");
```

Other languages have different OpenTelmetry APIs. Passing the [OTEL_EXPORTER_OTLP_HEADERS environment variable](https://opentelemetry.io/docs/specs/otel/protocol/exporter/) to apps is a universal way to configure the header.




# [Configuration](https://hub.docker.com/r/microsoft/dotnet-aspire-dashboard/)
The dashboard must be configured when it is started. The configuration is done via environment variables. The following environment variables are supported:

- `ASPNETCORE_URLS` specifies one or more HTTP endpoints through which the dashboard frontend is served. The frontend endpoint is used to view the dashboard in a browser. Defaults to `http://localhost:18888`.
  
- `DOTNET_DASHBOARD_OTLP_ENDPOINT_URL` specifies the **OTLP/gRPC**⁠ endpoint. This endpoint hosts an OTLP service and receives telemetry using gRPC. When the dashboard is launched by the .NET Aspire app host this address is secured with HTTPS. Securing the dashboard with HTTPS is recommended. Defaults to `http://localhost:18889`.
  
- `DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL` specifies the **OTLP/HTTP**⁠ endpoint. This endpoint hosts an OTLP service and receives telemetry using Protobuf over HTTP. Defaults to `http://localhost:18890`.




