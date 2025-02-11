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


---

```csharp
private void AddEndpointConfiguration(Dictionary<string, string?> values, string endpointName, string url, HttpProtocols? protocols = null, bool requiredClientCertificate = false)
{
    values[$"Kestrel:Endpoints:{endpointName}:Url"] = url;

    if (protocols != null)
    {
        values[$"Kestrel:Endpoints:{endpointName}:Protocols"] = protocols.ToString();
    }

    if (requiredClientCertificate && IsHttpsOrNull(BindingAddress.Parse(url)))
    {
        values[$"Kestrel:Endpoints:{endpointName}:ClientCertificateMode"] = ClientCertificateMode.RequireCertificate.ToString();
    }
}
```

---

```csharp
public void ConfigureKestrelEndpoints(WebApplicationBuilder builder, DashboardOptions dashboardOptions)
{
    // A single endpoint is configured if URLs are the same and the port isn't dynamic.
    var frontendAddresses = dashboardOptions.Frontend.GetEndpointAddresses();

    var otlpGrpcAddress = dashboardOptions.Otlp.GetGrpcEndpointAddress();

    var otlpHttpAddress = dashboardOptions.Otlp.GetHttpEndpointAddress();

    var hasSingleEndpoint = frontendAddresses.Count == 1 && IsSameOrNull(frontendAddresses[0], otlpGrpcAddress) && IsSameOrNull(frontendAddresses[0], otlpHttpAddress);

    var initialValues = new Dictionary<string, string?>();

    var browserEndpointNames = new List<string>(capacity: frontendAddresses.Count);

    if (!hasSingleEndpoint)
    {
        // Translate high-level config settings such as DOTNET_DASHBOARD_OTLP_ENDPOINT_URL and ASPNETCORE_URLS
        // to Kestrel's schema for loading endpoints from configuration.
        if (otlpGrpcAddress != null)
        {
            AddEndpointConfiguration(initialValues, "OtlpGrpc", otlpGrpcAddress.ToString(), HttpProtocols.Http2, requiredClientCertificate: dashboardOptions.Otlp.AuthMode == OtlpAuthMode.ClientCertificate);
        }
        if (otlpHttpAddress != null)
        {
            AddEndpointConfiguration(initialValues, "OtlpHttp", otlpHttpAddress.ToString(), HttpProtocols.Http1AndHttp2, requiredClientCertificate: dashboardOptions.Otlp.AuthMode == OtlpAuthMode.ClientCertificate);
        }

        if (frontendAddresses.Count == 1)
        {
            browserEndpointNames.Add("Browser");

            AddEndpointConfiguration(initialValues, "Browser", frontendAddresses[0].ToString());
        }
        else
        {
            for (var i = 0; i < frontendAddresses.Count; i++)
            {
                var name = $"Browser{i}";

                browserEndpointNames.Add(name);

                AddEndpointConfiguration(initialValues, name, frontendAddresses[i].ToString());
            }
        }
    }
    else
    {
        // At least one gRPC endpoint must be present.
        var url = otlpGrpcAddress?.ToString() ?? otlpHttpAddress?.ToString();

        AddEndpointConfiguration(initialValues, "OtlpGrpc", url!, HttpProtocols.Http1AndHttp2, requiredClientCertificate: dashboardOptions.Otlp.AuthMode == OtlpAuthMode.ClientCertificate);
    }

    builder.Configuration.AddInMemoryCollection(initialValues);

    // Use ConfigurationLoader to augment the endpoints that Kestrel created from configuration
    // with extra settings. e.g., UseOtlpConnection for the OTLP endpoint.
    builder.WebHost.ConfigureKestrel((context, serverOptions) =>
    {
        var logger = serverOptions.ApplicationServices.GetRequiredService<ILogger<Startup>>();

        var kestrelSection = context.Configuration.GetSection("Kestrel");

        KestrelConfigurationLoader configurationLoader = serverOptions.Configure(config: kestrelSection);

        foreach (var browserEndpointName in browserEndpointNames)
        {
            configurationLoader.Endpoint(name: browserEndpointName, configureOptions: endpointConfiguration =>
            {
                endpointConfiguration.ListenOptions.UseConnectionTypes([ConnectionType.Frontend]);

                // Only the last endpoint is accessible. Tests should only need one but
                // this will need to be improved if that changes.
                _frontendEndPointAccessor.Add(CreateEndPointAccessor(endpointConfiguration));
            });
        }

        configurationLoader.Endpoint(name: "OtlpGrpc", configureOptions: endpointConfiguration =>
        {
            var connectionTypes = new List<ConnectionType> { ConnectionType.Otlp };

            _otlpServiceGrpcEndPointAccessor ??= CreateEndPointAccessor(endpointConfiguration);

            if (hasSingleEndpoint)
            {
                logger.LogDebug("Browser and OTLP accessible on a single endpoint.");

                if (!endpointConfiguration.IsHttps)
                {
                    logger.LogWarning(
                        "The dashboard is configured with a shared endpoint for browser access and the OTLP service. " +
                        "The endpoint doesn't use TLS so browser access is only possible via a TLS terminating proxy.");
                }

                connectionTypes.Add(ConnectionType.Frontend);

                _frontendEndPointAccessor.Add(_otlpServiceGrpcEndPointAccessor);
            }

            endpointConfiguration.ListenOptions.UseConnectionTypes(connectionTypes.ToArray());

            if (endpointConfiguration.HttpsOptions.ClientCertificateMode == ClientCertificateMode.RequireCertificate)
            {
                // Allow invalid certificates when creating the connection. Certificate validation is done in the auth middleware.
                endpointConfiguration.HttpsOptions.ClientCertificateValidation = (certificate, chain, sslPolicyErrors) =>
                {
                    return true;
                };
            }
        });

        configurationLoader.Endpoint(name: "OtlpHttp", configureOptions: endpointConfiguration =>
        {
            var connectionTypes = new List<ConnectionType> { ConnectionType.Otlp };

            _otlpServiceHttpEndPointAccessor ??= CreateEndPointAccessor(endpointConfiguration);
            if (hasSingleEndpoint)
            {
                logger.LogDebug("Browser and OTLP accessible on a single endpoint.");

                if (!endpointConfiguration.IsHttps)
                {
                    logger.LogWarning(
                        "The dashboard is configured with a shared endpoint for browser access and the OTLP service. " +
                        "The endpoint doesn't use TLS so browser access is only possible via a TLS terminating proxy.");
                }

                connectionTypes.Add(ConnectionType.Frontend);
                _frontendEndPointAccessor.Add(_otlpServiceHttpEndPointAccessor);
            }

            endpointConfiguration.ListenOptions.UseConnectionTypes(connectionTypes.ToArray());

            if (endpointConfiguration.HttpsOptions.ClientCertificateMode == ClientCertificateMode.RequireCertificate)
            {
                // Allow invalid certificates when creating the connection. Certificate validation is done in the auth middleware.
                endpointConfiguration.HttpsOptions.ClientCertificateValidation = (certificate, chain, sslPolicyErrors) => { return true; };
            }
        });
    });
}

private Func<EndpointInfo> CreateEndPointAccessor(EndpointConfiguration endpointConfiguration)
{
    // We want to provide a way for someone to get the IP address of an endpoint.
    // However, if a dynamic port is used, the port is not known until the server is started.
    // Instead of returning the ListenOption's endpoint directly, we provide a func that returns the endpoint.
    // The endpoint on ListenOptions is updated after binding, so accessing it via the func after the server
    // has started returns the resolved port.
    var address = BindingAddress.Parse(endpointConfiguration.ConfigSection["Url"]!);

    return () =>
    {
        var endpoint = endpointConfiguration.ListenOptions.IPEndPoint!;

        return new EndpointInfo(address, endpoint, endpointConfiguration.IsHttps);
    };
}

private static bool IsSameOrNull(BindingAddress frontendAddress, BindingAddress? otlpAddress)
{
    return otlpAddress == null || (frontendAddress.Equals(otlpAddress) && otlpAddress.Port != 0);
}

private static bool IsHttpsOrNull(BindingAddress? address)
{
    return address == null || string.Equals(address.Scheme, "https", StringComparison.Ordinal);
}
```
