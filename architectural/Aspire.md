
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
