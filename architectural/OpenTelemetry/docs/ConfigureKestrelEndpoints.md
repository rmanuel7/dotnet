# [Configure endpoints in appsettings.json](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#configure-endpoints-in-appsettingsjson)

Kestrel can load endpoints from an [IConfiguration](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.iconfiguration) instance. By default, Kestrel configuration is loaded from the `Kestrel` section and endpoints are configured in `Kestrel:Endpoints`:

```json
{
  "Kestrel": {
    "Endpoints": {
      "MyHttpEndpoint": {
        "Url": "http://localhost:8080"
      }
    }
  }
}
```

The preceding example:

- Uses `appsettings.json` as the configuration source. However, any `IConfiguration` source can be used.
- Adds an endpoint named `MyHttpEndpoint` on port `8080`.

For more information about configuring endpoints with JSON, see later sections in this article that discuss [configuring HTTPS](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#configure-https-in-appsettingsjson) and [configuring HTTP protocols](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-9.0#configure-http-protocols-in-appsettingsjson) in appsettings.json.
