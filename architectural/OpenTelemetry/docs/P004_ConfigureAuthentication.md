# [Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-9.0#authentication-scheme)

The registered authentication handlers and their configuration options are called "schemes".

Authentication schemes are specified by registering authentication services. 

Less commonly, by calling `AuthenticationBuilder.AddScheme` directly.

---

### [DefaultScheme](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-9.0#defaultscheme)

```csharp
var authentication = builder.Services
    // ASP.NET Core authentication needs to have the correct default scheme for the configured frontend auth.
    // This is required for ASP.NET Core/SignalR/Blazor to flow the authenticated user from the request and into the dashboard app.
    .AddAuthentication(configureOptions: o => o.DefaultScheme = dashboardOptions.Frontend.AuthMode switch
    {
        FrontendAuthMode.Unsecured => FrontendAuthenticationDefaults.AuthenticationSchemeUnsecured,
        _ => CookieAuthenticationDefaults.AuthenticationScheme
    })
```

The AddAuthentication parameter `DefaultScheme` is the name of the scheme to use by default based on the authentication option selected in `dashboardOptions.Frontend.AuthMode`:
  * If `Unsecured`, use `FrontendAuthenticationDefaults.AuthenticationSchemeUnsecured`.
  * Otherwise, use cookies (`CookieAuthenticationDefaults.AuthenticationScheme`).

---

### `AuthenticationBuilder.AddScheme`

```csharp
            .AddScheme<FrontendCompositeAuthenticationHandlerOptions, FrontendCompositeAuthenticationHandler>(
                authenticationScheme: FrontendCompositeAuthenticationDefaults.AuthenticationScheme,
                configureOptions: o => { })

            .AddScheme<OtlpCompositeAuthenticationHandlerOptions, OtlpCompositeAuthenticationHandler>(
                authenticationScheme: OtlpCompositeAuthenticationDefaults.AuthenticationScheme,
                configureOptions: o => { })

            .AddScheme<OtlpApiKeyAuthenticationHandlerOptions, OtlpApiKeyAuthenticationHandler>(
                authenticationScheme: OtlpApiKeyAuthenticationDefaults.AuthenticationScheme,
                configureOptions: o => { })

            .AddScheme<ConnectionTypeAuthenticationHandlerOptions, ConnectionTypeAuthenticationHandler>(
                authenticationScheme: ConnectionTypeAuthenticationDefaults.AuthenticationSchemeFrontend,
                configureOptions: o => o.RequiredConnectionType = ConnectionType.Frontend)

            .AddScheme<ConnectionTypeAuthenticationHandlerOptions, ConnectionTypeAuthenticationHandler>(
                authenticationScheme: ConnectionTypeAuthenticationDefaults.AuthenticationSchemeOtlp,
                configureOptions: o => o.RequiredConnectionType = ConnectionType.Otlp);
```

If multiple schemes are used, authorization policies (or authorization attributes) can [specify the authentication scheme (or schemes)](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/limitingidentitybyscheme?view=aspnetcore-9.0) they depend on to authenticate the user.

---

### [Policy schemes](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/policyschemes?view=aspnetcore-9.0#jwt-with-multiple-schemes)

The `AddPolicyScheme` method can define multiple authentication schemes and implement logic to select the appropriate scheme 

```csharp
        switch (dashboardOptions.Frontend.AuthMode)
        {
            case FrontendAuthMode.OpenIdConnect:

                authentication.AddPolicyScheme(
                    authenticationScheme: FrontendAuthenticationDefaults.AuthenticationSchemeOpenIdConnect,
                    displayName: FrontendAuthenticationDefaults.AuthenticationSchemeOpenIdConnect,
                    configureOptions: o =>
                    {
                        // The frontend authentication scheme just redirects to OpenIdConnect and Cookie schemes, as appropriate.
                        o.ForwardDefault = CookieAuthenticationDefaults.AuthenticationScheme;
                        o.ForwardChallenge = OpenIdConnectDefaults.AuthenticationScheme;
                    });

                authentication.AddCookie(configureOptions: options =>
                {
                    options.Cookie.Name = DashboardAuthCookieName;
                });

                authentication.AddOpenIdConnect(configureOptions: options =>
                {
                    // Use authorization code flow so clients don't see access tokens.
                    options.ResponseType = OpenIdConnectResponseType.Code;

                    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;

                    // Scopes "openid" and "profile" are added by default, but need to be re-added
                    // in case configuration exists for Authentication:Schemes:OpenIdConnect:Scope.
                    if (!options.Scope.Contains(OpenIdConnectScope.OpenId))
                    {
                        options.Scope.Add(OpenIdConnectScope.OpenId);
                    }

                    if (!options.Scope.Contains("profile"))
                    {
                        options.Scope.Add("profile");
                    }

                    // Redirect to resources upon sign-in.
                    options.CallbackPath = "/"; // TargetLocationInterceptor.ResourcesPath;

                    // Avoid "message.State is null or empty" due to use of CallbackPath above.
                    options.SkipUnrecognizedRequests = true;
                });

                break;

            case FrontendAuthMode.BrowserToken:

                authentication.AddPolicyScheme(
                    authenticationScheme: FrontendAuthenticationDefaults.AuthenticationSchemeBrowserToken,
                    displayName: FrontendAuthenticationDefaults.AuthenticationSchemeBrowserToken,
                    configureOptions: o =>
                    {
                        o.ForwardDefault = CookieAuthenticationDefaults.AuthenticationScheme;
                    });

                authentication.AddCookie(configureOptions: options =>
                {
                    options.Cookie.Name = DashboardAuthCookieName;
                    options.LoginPath = "/login";
                    options.ReturnUrlParameter = "returnUrl";
                    options.ExpireTimeSpan = TimeSpan.FromDays(3);
                    options.Events.OnSigningIn = context =>
                    {
                        // Add claim when signing in with cookies from browser token.
                        // Authorization requires this claim. This prevents an identity from another auth scheme from being allow.
                        var claimsIdentity = (ClaimsIdentity)context.Principal!.Identity!;

                        claimsIdentity.AddClaim(new Claim(FrontendAuthorizationDefaults.BrowserTokenClaimName, bool.TrueString));

                        return Task.CompletedTask;
                    };
                });

                break;

            case FrontendAuthMode.Unsecured:

                authentication.AddScheme<AuthenticationSchemeOptions, UnsecuredAuthenticationHandler>(
                    authenticationScheme: FrontendAuthenticationDefaults.AuthenticationSchemeUnsecured,
                    configureOptions: o => { });

                break;
        }
```
