# [Enable CORS](https://learn.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-9.0#enable-cors)


### In middleware using a named policy or default policy.

```csharp
public static class OtlpHttpEndpointsBuilder
{
    //...
    public const string CorsPolicyName = "OtlpHttpCors";
    //...
}
```

```csharp
if (dashboardOptions.Otlp.Cors.IsCorsEnabled)
{
    builder.Services.AddCors(setupAction: options =>
    {
        options.AddPolicy(name: OtlpHttpEndpointsBuilder.CorsPolicyName, configurePolicy: policy =>
        {
            var corsOptions = dashboardOptions.Otlp.Cors;

            policy.WithOrigins(origins: corsOptions.AllowedOrigins.Split(',', StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries));

            policy.SetIsOriginAllowedToAllowWildcardSubdomains();

            // By default, allow headers in the implicit safelist and X-Requested-With. This matches OTLP collector CORS behavior.
            // Implicit safelist: https://developer.mozilla.org/en-US/docs/Glossary/CORS-safelisted_request_header
            // OTLP collector: https://github.com/open-telemetry/opentelemetry-collector/blob/685625abb4703cb2e45a397f008127bbe2ba4c0e/config/confighttp/README.md#server-configuration
            var allowedHeaders = !string.IsNullOrEmpty(corsOptions.AllowedHeaders)
                ? corsOptions.AllowedHeaders.Split(',', StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries)
                : ["X-Requested-With"];

            policy.WithHeaders(allowedHeaders);

            // Hardcode to allow only POST methods. OTLP is always sent in POST request bodies.
            policy.WithMethods(HttpMethods.Post);
        });
    });
}
```

The preceding code:

- Sets the policy name to `OtlpHttpEndpointsBuilder.CorsPolicyName`.
- Calls `AddCors` with a lambda expression. The lambda takes a CorsPolicyBuilder object. Configuration options, such as `WithOrigins`.
- Enables the `OtlpHttpEndpointsBuilder.CorsPolicyName` CORS policy for all endpoints. 

---

### [Enable Cors with endpoint routing](https://learn.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-9.0#enable-cors-with-endpoint-routing)

```csharp
    public static void MapHttpOtlpApi(this IEndpointRouteBuilder endpoints/*, OtlpOptions options*/)
    {
        //var httpEndpoint = options.GetHttpEndpointAddress();
        //if (httpEndpoint == null)
        //{
        //    // Don't map OTLP HTTP route endpoints if there isn't a Kestrel endpoint to access them with.
        //    return;
        //}

        var group = endpoints
            .MapGroup("/v1")
            .AddOtlpHttpMetadata();

        //if (!string.IsNullOrEmpty(options.Cors.AllowedOrigins))
        //{
        //    group = group.RequireCors(CorsPolicyName);
        //}

        group.MapPost("logs", static (MessageBindable<ExportLogsServiceRequest> request, OtlpLogsService service) =>
        {
            if (request.Message == null)
            {
                return Results.Empty;
            }
            return OtlpResult.Response(service.Export(request.Message));
        });

        group.MapPost("traces", static (MessageBindable<ExportTraceServiceRequest> request, OtlpTraceService service) =>
        {
            if (request.Message == null)
            {
                return Results.Empty;
            }
            return OtlpResult.Response(service.Export(request.Message));
        });

        group.MapPost("metrics", (MessageBindable<ExportMetricsServiceRequest> request, OtlpMetricsService service) =>
        {
            if (request.Message == null)
            {
                return Results.Empty;
            }
            return OtlpResult.Response(service.Export(request.Message));
        });
    }
```

---
### 
- With the [EnableCors] attribute.


