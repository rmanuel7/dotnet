# [Protocol Details](https://github.com/open-telemetry/opentelemetry-proto/blob/main/docs/specification.md#protocol-details)

OTLP defines the encoding of telemetry data and the protocol used to exchange data between the client and the server.

This specification defines how OTLP is implemented over [gRPC](https://grpc.io/) and HTTP transports and specifies [Protocol Buffers schema](https://developers.google.com/protocol-buffers/docs/overview) that is used for the payloads.

OTLP is a request/response style protocol: the clients send requests, and the server replies with corresponding responses. This document defines one request and response type: `Export`.

All server components MUST support the following transport compression options:

- No compression, denoted by `none`.
- Gzip compression, denoted by `gzip`.

```csharp
public class Startup
{
    public IConfiguration Configuration { get; }

    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        // Add services to the container.
        services.AddGrpc();

        services.AddRazorComponents()
            .AddInteractiveServerComponents();

        // Enable the Response Compression Middleware
        //services.AddResponseCompression(options =>
        //{
        //    options.EnableForHttps = true;
        //});

        services.AddTransient<OtlpLogsService>();
        services.AddTransient<OtlpTraceService>();
        services.AddTransient<OtlpMetricsService>();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // Configure the HTTP request pipeline.
        if (!env.IsDevelopment())
        {
            app.UseExceptionHandler("/Error");
            // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
            app.UseHsts();
        }

        app.UseHttpsRedirection();

        app.UseRouting();

        app.UseAntiforgery();

        // app.UseResponseCompression();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapStaticAssets();
            endpoints.MapRazorComponents<App>()
                .AddInteractiveServerRenderMode();

            // OTLP HTTP services.
            endpoints.MapHttpOtlpApi(/*dashboardOptions.Otlp*/);

            // OTLP gRPC services.
            endpoints.MapGrpcService<OtlpGrpcTraceService>();
            endpoints.MapGrpcService<OtlpGrpcMetricsService>();
            endpoints.MapGrpcService<OtlpGrpcLogsService>();
        });
    }
}
```

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
