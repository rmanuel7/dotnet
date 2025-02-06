
## [Configure OTLP HTTP with standalone dashboard](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/enable-browser-telemetry?tabs=bash#configure-otlp-http-with-standalone-dashboard)
If the dashboard is used standalone, without the rest of .NET Aspire, the OTLP HTTP endpoint is enabled by default on port `18890`. However, the port must be mapped when the dashboard container is started:

```yml
  monitoring.ui:
    image: mcr.microsoft.com/dotnet/aspire-dashboard:9.0
    container_name: Aspire
    ports:
      - 18888:18888
      - 4317:18889
      - 4318:18890
    environment:
      - Dashboard__ResourceServiceClient__Url=http://192.168.1.2:5100
      - Dashboard__ResourceServiceClient__AuthMode=Unsecured
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
