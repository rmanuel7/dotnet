

```csharp
var dashboardConfigSection = Configuration.GetSection("Dashboard");
services.AddOptions<DashboardOptions>()
    .Bind(dashboardConfigSection)
    .ValidateOnStart();
services.AddSingleton<IPostConfigureOptions<DashboardOptions>, PostConfigureDashboardOptions>();
services.AddSingleton<IValidateOptions<DashboardOptions>, ValidateDashboardOptions>();
```

---

```csharp
internal static class DashboardConfigNames
{
    public static readonly ConfigName DashboardFrontendUrlName = new("ASPNETCORE_URLS");
    public static readonly ConfigName DashboardOtlpGrpcUrlName = new("DOTNET_DASHBOARD_OTLP_ENDPOINT_URL");
    public static readonly ConfigName DashboardOtlpHttpUrlName = new("DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL");
    public static readonly ConfigName DashboardUnsecuredAllowAnonymousName = new("DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS");
    public static readonly ConfigName DashboardConfigFilePathName = new("DOTNET_DASHBOARD_CONFIG_FILE_PATH");
    public static readonly ConfigName ResourceServiceUrlName = new("DOTNET_RESOURCE_SERVICE_ENDPOINT_URL");

    public static readonly ConfigName DashboardOtlpAuthModeName = new("Dashboard:Otlp:AuthMode", "DASHBOARD__OTLP__AUTHMODE");
    public static readonly ConfigName DashboardOtlpPrimaryApiKeyName = new("Dashboard:Otlp:PrimaryApiKey", "DASHBOARD__OTLP__PRIMARYAPIKEY");
    public static readonly ConfigName DashboardOtlpSecondaryApiKeyName = new("Dashboard:Otlp:SecondaryApiKey", "DASHBOARD__OTLP__SECONDARYAPIKEY");
    public static readonly ConfigName DashboardOtlpCorsAllowedOriginsKeyName = new("Dashboard:Otlp:Cors:AllowedOrigins", "DASHBOARD__OTLP__CORS__ALLOWEDORIGINS");
    public static readonly ConfigName DashboardOtlpCorsAllowedHeadersKeyName = new("Dashboard:Otlp:Cors:AllowedHeaders", "DASHBOARD__OTLP__CORS__ALLOWEDHEADERS");
    public static readonly ConfigName DashboardOtlpAllowedCertificatesName = new("Dashboard:Otlp:AllowedCertificates", "DASHBOARD__OTLP__ALLOWEDCERTIFICATES");
    public static readonly ConfigName DashboardFrontendAuthModeName = new("Dashboard:Frontend:AuthMode", "DASHBOARD__FRONTEND__AUTHMODE");
    public static readonly ConfigName DashboardFrontendBrowserTokenName = new("Dashboard:Frontend:BrowserToken", "DASHBOARD__FRONTEND__BROWSERTOKEN");
    public static readonly ConfigName DashboardFrontendMaxConsoleLogCountName = new("Dashboard:Frontend:MaxConsoleLogCount", "DASHBOARD__FRONTEND__MAXCONSOLELOGCOUNT");
    public static readonly ConfigName ResourceServiceClientAuthModeName = new("Dashboard:ResourceServiceClient:AuthMode", "DASHBOARD__RESOURCESERVICECLIENT__AUTHMODE");
    public static readonly ConfigName ResourceServiceClientCertificateSourceName = new("Dashboard:ResourceServiceClient:ClientCertificate:Source", "DASHBOARD__RESOURCESERVICECLIENT__CLIENTCERTIFICATE__SOURCE");
    public static readonly ConfigName ResourceServiceClientCertificateFilePathName = new("Dashboard:ResourceServiceClient:ClientCertificate:FilePath", "DASHBOARD__RESOURCESERVICECLIENT__CLIENTCERTIFICATE__FILEPATH");
    public static readonly ConfigName ResourceServiceClientCertificateSubjectName = new("Dashboard:ResourceServiceClient:ClientCertificate:Subject", "DASHBOARD__RESOURCESERVICECLIENT__CLIENTCERTIFICATE__SUBJECT");
    public static readonly ConfigName ResourceServiceClientApiKeyName = new("Dashboard:ResourceServiceClient:ApiKey", "DASHBOARD__RESOURCESERVICECLIENT__APIKEY");
}

internal readonly struct ConfigName(string configKey, string? envVarName = null)
{
    public string ConfigKey { get; } = configKey;
    public string EnvVarName { get; } = envVarName ?? configKey;
}
```

---
## [Dashboard configuration](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash)

The dashboard is configured when it starts up. Configuration includes frontend and OpenTelemetry Protocol (OTLP) addresses, the resource service endpoint, authentication, telemetry limits, and more.

When the dashboard is launched with the .NET Aspire app host project, it's automatically configured to display the app's resources and telemetry. Configuration is provided when launching the dashboard in standalone mode.

There are many ways to provide configuration:

- Command line arguments.
- Environment variables. The `:` delimiter should be replaced with double underscore (`__`) in environment variable names.
- Optional JSON configuration file. The `DOTNET_DASHBOARD_CONFIG_FILE_PATH` setting can be used to specify a JSON configuration file.
  
  ```json
    {
      "Dashboard": {
        "ApplicationName": "MyDashboardApp",
        "Otlp": {
          "AuthMode": "ApiKey",
          "PrimaryApiKey": "your-primary-api-key",
          "SecondaryApiKey": "your-secondary-api-key",
          "Cors": {
            "AllowedOrigins": ["http://localhost", "https://yourdomain.com"],
            "AllowedHeaders": ["Content-Type", "Authorization"]
          }
        },
        "Frontend": {
          "AuthMode": "OpenIdConnect",
          "BrowserToken": "your-browser-token",
          "OpenIdConnect": {
            "NameClaimType": "name",
            "UsernameClaimType": "preferred_username",
            "RequiredClaimType": "role",
            "RequiredClaimValue": "admin"
          }
        },
        "ResourceServiceClient": {
          "Url": "https://resource-service.com",
          "AuthMode": "ApiKey",
          "ApiKey": "your-resource-service-api-key",
          "ClientCertificate": {
            "Source": "File",
            "FilePath": "path/to/certificate.pfx",
            "Password": "your-certificate-password",
            "Subject": "your-certificate-subject",
            "Store": "My",
            "Location": "LocalMachine"
          }
        },
        "TelemetryLimits": {
          "MaxLogCount": 1000,
          "MaxTraceCount": 1000,
          "MaxMetricsCount": 1000,
          "MaxAttributeCount": 100,
          "MaxAttributeLength": 256,
          "MaxSpanEventCount": 50
        }
      },
      "Authentication": {
        "Schemes": {
          "OpenIdConnect": {
            "Authority": "https://your-identity-provider.com",
            "ClientId": "your-client-id",
            "ClientSecret": "your-client-secret"
          }
        }
      }
    }
    ```

---

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:ApplicationName` | `Aspire` | The application name to be displayed in the UI. This applies only when no resource service URL is specified. When a resource service exists, the service specifies the application name. |

---

### [Common configuration](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#common-configuration)

| Option | Default value | Description |
| --- | --- | --- |
| `ASPNETCORE_URLS` | `http://localhost:18888` | One or more HTTP endpoints through which the dashboard frontend is served. The frontend endpoint is used to view the dashboard in a browser. When the dashboard is launched by the .NET Aspire app host this address is secured with HTTPS. Securing the dashboard with HTTPS is recommended. |
| `DOTNET_DASHBOARD_OTLP_ENDPOINT_URL` | `http://localhost:18889` | The [OTLP/gRPC](https://opentelemetry.io/docs/specs/otlp/#otlpgrpc) endpoint. This endpoint hosts an OTLP service and receives telemetry using gRPC. When the dashboard is launched by the .NET Aspire app host this address is secured with HTTPS. Securing the dashboard with HTTPS is recommended. |
| `DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL` | `http://localhost:18890` | The [OTLP/HTTP](https://opentelemetry.io/docs/specs/otlp/#otlphttp) endpoint. This endpoint hosts an OTLP service and receives telemetry using Protobuf over HTTP. When the dashboard is launched by the .NET Aspire app host the OTLP/HTTP endpoint isn't configured by default. To configure an OTLP/HTTP endpoint with the app host, set an `DOTNET_DASHBOARD_OTLP_HTTP_ENDPOINT_URL` env var value in _launchSettings.json_. Securing the dashboard with HTTPS is recommended. |
| `DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS` | `false` | Configures the dashboard to not use authentication and accepts anonymous access. This setting is a shortcut to configuring `Dashboard:Frontend:AuthMode` and `Dashboard:Otlp:AuthMode` to `Unsecured`. |
| `DOTNET_DASHBOARD_CONFIG_FILE_PATH` | `null` | The path for a JSON configuration file. If the dashboard is being run in a Docker container, then this is the path to the configuration file in a mounted volume. This value is optional. |
| `DOTNET_RESOURCE_SERVICE_ENDPOINT_URL` | `null` | The gRPC endpoint to which the dashboard connects for its data. If this value is unspecified, the dashboard shows telemetry data but no resource list or console logs. This setting is a shortcut to `Dashboard:ResourceServiceClient:Url`. |
| `DOTNET_DASHBOARD_RESOURCESERVICE_APIKEY` | Automatically generated 128-bit entropy token. | The API key used to authenticate requests made to the app host's resource service. The API key is required if the app host is in run mode, the dashboard isn't disabled, and the dashboard isn't configured to allow anonymous access with `DOTNET_DASHBOARD_UNSECURED_ALLOW_ANONYMOUS`. |
| `DOTNET_DASHBOARD_FRONTEND_BROWSERTOKEN` | Automatically generated 128-bit entropy token. | Configures the frontend browser token. This is the value that must be entered to access the dashboard when the auth mode is BrowserToken. If no browser token is specified then a new token is generated each time the app host is launched. |


---

### [OTLP authentication](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#otlp-authentication)
The OTLP endpoint authentication is configured with `Dashboard:Otlp:AuthMode`. The OTLP endpoint can be secured with an API key or [client certificate](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/certauth) authentication.

API key authentication works by requiring each OTLP request to have a valid `x-otlp-api-key` header value. It must match either the primary or secondary key.

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:Otlp:AuthMode` | `Unsecured` | Can be set to `ApiKey`, `Certificate` or `Unsecured`. `Unsecured` should only be used during local development. It's not recommended when hosting the dashboard publicly or in other settings. |
| `Dashboard:Otlp:PrimaryApiKey` | `null` | Specifies the primary API key. The API key can be any text, but a value with at least 128 bits of entropy is recommended. This value is required if auth mode is API key. |
| `Dashboard:Otlp:SecondaryApiKey` | `null` | Specifies the secondary API key. The API key can be any text, but a value with at least 128 bits of entropy is recommended. This value is optional. If a second API key is specified, then the incoming `x-otlp-api-key` header value can match either the primary or secondary key. |

### [OTLP CORS](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#otlp-cors)
Cross-origin resource sharing (CORS) can be configured to allow browser apps to send telemetry to the dashboard.

By default, browser apps are restricted from making cross domain API calls. This impacts sending telemetry to the dashboard because the dashboard and the browser app are always on different domains. To configure CORS, use the Dashboard:Otlp:Cors section and specify the allowed origins and headers:

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:Otlp:Cors:AllowedOrigins` | `null` | Specifies the allowed origins for CORS. It's a comma-delimited string and can include the `*` wildcard to allow any domain. This option is optional and can be set using the `DASHBOARD__OTLP__CORS__ALLOWEDORIGINS` environment variable. |
| `Dashboard:Otlp:Cors:AllowedHeaders` | `null` | A comma-delimited string representing the allowed headers for CORS. This setting is optional and can be set using the `DASHBOARD__OTLP__CORS__ALLOWEDHEADERS` environment variable. |




---

### [Frontend authentication](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#frontend-authentication)
The dashboard frontend endpoint authentication is configured with `Dashboard:Frontend:AuthMode`. The frontend can be secured with OpenID Connect (OIDC) or browser token authentication.

Browser token authentication works by the frontend asking for a token. The token can either be entered in the UI or provided as a query string value to the login page. For example, `https://localhost:1234/login?t=TheToken`. When the token is successfully authenticated an auth cookie is persisted to the browser, and the browser is redirected to the app.

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:Frontend:AuthMode` | `BrowserToken` | Can be set to `BrowserToken`, `OpenIdConnect` or `Unsecured`. `Unsecured` should only be used during local development. It's not recommended when hosting the dashboard publicly or in other settings. |
| `Dashboard:Frontend:BrowserToken` | `null` | Specifies the browser token. If the browser token isn't specified, then the dashboard generates one. Tooling that wants to automate logging in with browser token authentication can specify a token and open a browser with the token in the query string. A new token should be generated each time the dashboard is launched. |
| `Dashboard:Frontend:OpenIdConnect:NameClaimType` | `name` | Specifies one or more claim types that should be used to display the authenticated user's full name. Can be a single claim type or a comma-delimited list of claim types. |
| `Dashboard:Frontend:OpenIdConnect:UsernameClaimType` | `preferred_username` | Specifies one or more claim types that should be used to display the authenticated user's username. Can be a single claim type or a comma-delimited list of claim types. |
| `Dashboard:Frontend:OpenIdConnect:RequiredClaimType` | `null` | Specifies the claim that must be present for authorized users. Authorization fails without this claim. This value is optional. |
| `Dashboard:Frontend:OpenIdConnect:RequiredClaimValue` | `null` | Specifies the value of the required claim. Only used if `Dashboard:Frontend:OpenIdConnect:RequireClaimType` is also specified. This value is optional. |
| `Authentication:Schemes:OpenIdConnect:Authority` | `null` | URL to the identity provider (IdP). |
| `Authentication:Schemes:OpenIdConnect:ClientId` | `null` | Identity of the relying party (RP). |
| `Authentication:Schemes:OpenIdConnect:ClientSecret` | `null` | A secret that only the real RP would know. |
| Other properties of [OpenIdConnectOptions](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.openidconnectoptions) | `null` | Values inside configuration section `Authentication:Schemes:OpenIdConnect:*` are bound to `OpenIdConnectOptions`, such as `Scope`. |




---

### [Resources](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#resources)
The dashboard connects to a resource service to load and display resource information. The client is configured in the dashboard for how to connect to the service.

The resource service client authentication is configured with `Dashboard:ResourceServiceClient:AuthMode`. The client can be configured to support API key or client certificate authentication.

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:ResourceServiceClient:Url` | `null` | The gRPC endpoint to which the dashboard connects for its data. If this value is unspecified, the dashboard shows telemetry data but no resource list or console logs. |
| `Dashboard:ResourceServiceClient:AuthMode` | `null` | Can be set to `ApiKey`, `Certificate` or `Unsecured`. `Unsecured` should only be used during local development. It's not recommended when hosting the dashboard publicly or in other settings. This value is required if a resource service URL is specified. |
| `Dashboard:ResourceServiceClient:ApiKey` | `null` | The API to send to the resource service in the `x-resource-service-api-key` header. This value is required if auth mode is API key. |
| `Dashboard:ResourceServiceClient:ClientCertificate:Source` | `null` | Can be set to `File` or `KeyStore`. This value is required if auth mode is client certificate. |
| `Dashboard:ResourceServiceClient:ClientCertificate:FilePath` | `null` | The certificate file path. This value is required if source is `File`. |
| `Dashboard:ResourceServiceClient:ClientCertificate:Password` | `null` | The password for the certificate file. This value is optional. |
| `Dashboard:ResourceServiceClient:ClientCertificate:Subject` | `null` | The certificate subject. This value is required if source is `KeyStore`. |
| `Dashboard:ResourceServiceClient:ClientCertificate:Store` | `My` | The certificate [StoreName](/en-us/dotnet/api/system.security.cryptography.x509certificates.storename). |
| `Dashboard:ResourceServiceClient:ClientCertificate:Location` | `CurrentUser` | The certificate [StoreLocation](/en-us/dotnet/api/system.security.cryptography.x509certificates.storelocation). |

---

### [Telemetry limits](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/dashboard/configuration?tabs=bash#telemetry-limits)
Telemetry is stored in memory. To avoid excessive memory usage, the dashboard has limits on the count and size of stored telemetry. When a count limit is reached, new telemetry is added, and the oldest telemetry is removed. When a size limit is reached, data is truncated to the limit.

Telemetry limits have different scopes depending upon the telemetry type:

- `MaxLogCount` and `MaxTraceCount` are shared across resources. For example, a `MaxLogCount` value of 5,000 configures the dashboard to store up to 5,000 total log entries for all resources.
- `MaxMetricsCount` is per-resource. For example, a `MaxMetricsCount` value of 10,000 configures the dashboard to store up to 10,000 metrics data points per-resource.

| Option | Default value | Description |
| --- | --- | --- |
| `Dashboard:TelemetryLimits:MaxLogCount` | 10,000 | The maximum number of log entries. Limit is shared across resources. |
| `Dashboard:TelemetryLimits:MaxTraceCount` | 10,000 | The maximum number of log traces. Limit is shared across resources. |
| `Dashboard:TelemetryLimits:MaxMetricsCount` | 50,000 | The maximum number of metric data points. Limit is per-resource. |
| `Dashboard:TelemetryLimits:MaxAttributeCount` | 128 | The maximum number of attributes on telemetry. |
| `Dashboard:TelemetryLimits:MaxAttributeLength` | `null` | The maximum length of attributes. |
| `Dashboard:TelemetryLimits:MaxSpanEventCount` | `null` | The maximum number of events on span attributes. |


---

### [Connection string and endpoint references](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/app-host-overview?tabs=docker#connection-string-and-endpoint-references)
It's common to express dependencies between project resources.

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var cache = builder.AddRedis("cache");

var apiservice = builder.AddProject<Projects.AspireApp_ApiService>("apiservice");

builder.AddProject<Projects.AspireApp_Web>("webfrontend")
       .WithReference(cache)
       .WithReference(apiservice);
```

| Method | Environment variable |
| --- | --- |
| `WithReference(cache)` | `ConnectionStrings__cache="localhost:62354"` |
| `WithReference(apiservice)` | `services__apiservice__http__0="http://localhost:5455"`  <br>`services__apiservice__https__0="https://localhost:7356"` |

Adding a reference to the "apiservice" project results in service discovery environment variables being added to the frontend. This is because typically, project-to-project communication occurs over HTTP/gRPC.


