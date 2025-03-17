# [Authentication fundamentals](https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-9.0#cookie-based-authentication)

> [!NOTE]
> **Cookie-based authentication** is a popular form of authentication.

> [!NOTE]
> **Token-based authentication systems** are growing in popularity, especially for Single Page Applications (SPAs).

---

## [Cookie-based authentication](https://learn.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-9.0#cookie-based-authentication)
> [!NOTE]
> When a user authenticates using their **username and password** they're **issued a token** containing an **authentication ticket**.

> [!NOTE]
> The token can be used for authentication and authorization.

> [!IMPORTANT]
> The token is **stored as a cookie** that's sent with **every request** the client makes.

> [!IMPORTANT]
> **Generating and validating** this cookie is performed with the `Cookie Authentication Middleware`.

> [!IMPORTANT]
> The middleware serializes a **user principal** into an encrypted cookie.

> [!TIP]
> **On subsequent requests**, the middleware validates the cookie, **recreates the principal**, and assigns the principal to the `HttpContext.User` property.


## [Authentication concepts](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-9.0#authentication-concepts)
> [!IMPORTANT]
> Authentication is responsible for providing the `ClaimsPrincipal` for **authorization** to make permission decisions against.

> [!NOTE]
> The registered **authentication handlers** and their configuration options are called **schemes**.

> [!NOTE]
> The **authentication scheme** can **select** which authentication handler is **responsible** for generating the correct set of claims.


## [Authentication handler](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/?view=aspnetcore-9.0#authentication-handler)
> [!NOTE]
> Has the primary responsibility to authenticate users.

> [!NOTE]
> Is derived from `IAuthenticationHandler` or `AuthenticationHandler<TOptions>`.

> [!NOTE]
> Is a type that implements the **behavior of a scheme**.
> - Construct `AuthenticationTicket` objects representing the user's identity if authentication **is successful**.
> - Return 'no result' or 'failure' if authentication **is unsuccessful**.
> - Have methods for challenge and forbid actions for when users attempt to access resources:
>   - They're unauthorized to access (forbid).
>   - When they're unauthenticated (challenge).

<br />
<br />
<br />

---

<br />
<br />
<br />

¡Claro! Entiendo que estás construyendo un `AuthenticationHandler` personalizado para el flujo de concesión de contraseña (Resource Owner Password Credentials - ROPC) en ASP.NET Core, y te preocupa que se esté intentando autenticar en cada solicitud. Tienes razón en que la autenticación solo debe ocurrir cuando el usuario inicia sesión, no en cada solicitud subsiguiente.

El problema principal en tu código es que estás realizando la autenticación (llamando a `_ropcService.SignInIdp`) dentro del método `HandleAuthenticateAsync`. Este método se llama en cada solicitud que requiere autenticación, lo que explica por qué se intenta iniciar sesión en cada llamada.

Aquí te presento una versión corregida y explicada de tu `AuthenticationHandler` para que comprendas mejor el flujo de autenticación en ASP.NET Core y cómo implementar correctamente el flujo de concesión de contraseña:

**1. Corrección del `PasswordFlowAuthenticationHandler`**

```csharp
using System;
using System.Security.Claims;
using System.Text.Encodings.Web;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Options;
using System.IdentityModel.Tokens.Jwt;
using Microsoft.AspNetCore.Http; // Para HttpContext

public class PasswordFlowAuthenticationHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    private readonly IRopcService _ropcService;

    public PasswordFlowAuthenticationHandler(IOptionsMonitor<AuthenticationSchemeOptions> options, ILoggerFactory logger, UrlEncoder encoder, IRopcService ropcService)
        : base(options, logger, encoder)
    {
        _ropcService = ropcService;
    }

    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Verifica si ya hay un principal autenticado en el contexto.
        if (Context.User.Identity?.IsAuthenticated == true)
        {
            return AuthenticateResult.Success(new AuthenticationTicket(Context.User, Scheme.Name));
        }

        // Verifica si la solicitud tiene un token de acceso en el encabezado de autorización.
        if (!Request.Headers.ContainsKey("Authorization"))
        {
            return AuthenticateResult.NoResult(); // No se requiere autenticación, o no hay token.
        }

        var authorizationHeader = Request.Headers["Authorization"].ToString();
        if (!authorizationHeader.StartsWith("Bearer ", StringComparison.OrdinalIgnoreCase))
        {
            return AuthenticateResult.NoResult(); // No es un token Bearer.
        }

        var accessToken = authorizationHeader.Substring("Bearer ".Length).Trim();

        try
        {
            // Validar el token de acceso (aquí puedes usar tu lógica de validación).
            var handler = new JwtSecurityTokenHandler();
            var jwtSecurityToken = handler.ReadJwtToken(accessToken);

            // Crear principal autenticado.
            var identity = new ClaimsIdentity(jwtSecurityToken.Claims, Scheme.Name);
            var principal = new ClaimsPrincipal(identity);
            var ticket = new AuthenticationTicket(principal, Scheme.Name);

            return AuthenticateResult.Success(ticket);
        }
        catch (Exception ex)
        {
            return AuthenticateResult.Fail($"Token de acceso inválido: {ex.Message}");
        }
    }

    public async Task<AuthenticateResult> AuthenticateUser(string username, string password)
    {
        var result = await _ropcService.SignInIdp(username, password);

        return await result.Switch(
            onSuccess: async tokenResponse =>
            {
                return await Success(token: tokenResponse);
            },
            onFailure: error =>
            {
                return Task.FromResult(result: AuthenticateResult.Fail(error.Message));
            });
    }

    private async Task<AuthenticateResult> Success(TokenResponse token)
    {
        if (token == null)
        {
            return AuthenticateResult.Fail("Credenciales inválidas.");
        }

        var handler = new JwtSecurityTokenHandler();
        var jwtSecurityToken = handler.ReadJwtToken(token.IdToken);

        // Crear principal autenticado.
        var identity = new ClaimsIdentity(jwtSecurityToken.Claims, Scheme.Name);

        identity.AddClaim(claim: new Claim("access_token", token.AccessToken));
        identity.AddClaim(claim: new Claim("refresh_token", token.RefreshToken));

        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);

        // Guardar el principal autenticado en el contexto.
        await Context.SignInAsync(Scheme.Name, principal);

        return AuthenticateResult.Success(ticket);
    }
}
```

**2. Explicación de los Cambios**

* **`HandleAuthenticateAsync` Corregido:**
    * Primero, verifica si el usuario ya está autenticado (`Context.User.Identity?.IsAuthenticated == true`). Si lo está, devuelve `AuthenticateResult.Success` con el principal autenticado existente.
    * Luego, verifica si el encabezado de autorización contiene un token Bearer. Si no lo encuentra, devuelve `AuthenticateResult.NoResult()`.
    * Si encuentra un token Bearer, lo extrae y lo valida.
    * Si el token es válido, crea un `ClaimsPrincipal` a partir de las reclamaciones del token y devuelve `AuthenticateResult.Success`.
    * Si el token no es valido retorna un error.
* **Método `AuthenticateUser`:**
    * Este método encapsula la lógica de autenticación (la llamada a `_ropcService.SignInIdp`).
    * Lo puedes llamar desde tu controlador o servicio cuando el usuario envía sus credenciales para iniciar sesión.
    * Este metodo guarda el principal autenticado en el contexto usando `Context.SignInAsync`.
* **Validación de Token:**
    * En el `HandleAuthenticateAsync`, debes implementar tu lógica de validación de token. Esto puede incluir verificar la firma del token, la fecha de expiración, y otras reclamaciones.

**3. Implementación en tu Controlador**

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly PasswordFlowAuthenticationHandler _authenticationHandler;

    public AuthController(PasswordFlowAuthenticationHandler authenticationHandler)
    {
        _authenticationHandler = authenticationHandler;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login(string username, string password)
    {
        var result = await _authenticationHandler.AuthenticateUser(username, password);

        if (result.Succeeded)
        {
            return Ok("Inicio de sesión exitoso.");
        }

        return BadRequest(result.Failure.Message);
    }

    [HttpGet("protected")]
    [Authorize(AuthenticationSchemes = "PasswordFlow")] // Usa el esquema de autenticación personalizado.
    public IActionResult ProtectedResource()
    {
        return Ok("Recurso protegido.");
    }
}
```

**4. Configuración en `Startup.cs` o `Program.cs`**

```csharp
// Program.cs
builder.Services.AddAuthentication("PasswordFlow")
    .AddScheme<AuthenticationSchemeOptions, PasswordFlowAuthenticationHandler>("PasswordFlow", null);

builder.Services.AddAuthorization();
```

Con estos cambios, la autenticación solo ocurrirá cuando llames al método `AuthenticateUser` en tu controlador, y las solicitudes subsiguientes utilizarán el token de acceso para autenticar al usuario.









