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
