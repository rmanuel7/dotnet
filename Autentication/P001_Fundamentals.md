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


## Authentication service
> [!NOTE]
> In ASP.NET Core, authentication is handled by the authentication service, `IAuthenticationService`, which is used by authentication middleware.

> [!IMPORTANT]
>  The authentication service uses registered authentication handlers **to complete authentication-related actions**.



## [Create an authentication cookie](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/cookie?view=aspnetcore-9.0#create-an-authentication-cookie)
> [!IMPORTANT]
> To create a cookie holding user information, construct a `ClaimsPrincipal`.
> -  The user information is serialized and stored in the cookie.

> [!TIP]
> Create a `ClaimsIdentity` with any required `Claims` and call `SignInAsync` to sign in the user.
> - `SignInAsync` creates an encrypted cookie and **adds it to the current response**. If AuthenticationScheme isn't specified, the default scheme is used.

```csharp
await HttpContext.SignInAsync(
            CookieAuthenticationDefaults.AuthenticationScheme, 
            new ClaimsPrincipal(claimsIdentity), 
            new AuthenticationProperties
            {
                IsPersistent = true,
                ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
            });
```

> [!TIP]
> To sign out the current user and delete their cookie, call `SignOutAsync`:

```csharp
// Clear the existing external cookie
    await HttpContext.SignOutAsync(
        CookieAuthenticationDefaults.AuthenticationScheme);
```


> [!CAUTION]
> System.InvalidOperationException:
> - The authentication handler registered for scheme 'PasswordScheme' is 'PasswordFlowAuthenticationHandler' which cannot be used for SignInAsync.
> - Did you forget to call AddAuthentication().AddCookie("Cookies") and SignInAsync("Cookies",...)?

<br />
<br />
<br />

---

<br />
<br />
<br />

Claro, vamos a desglosar qué significa `Context.SignInAsync(Scheme.Name, principal)` y sus implicaciones con las cookies y el almacenamiento de información de inicio de sesión.

**¿Qué significa `Context.SignInAsync(Scheme.Name, principal)`?**

Este método, `SignInAsync`, es parte del sistema de autenticación de ASP.NET Core. Su función principal es establecer el principal de identidad del usuario (el `ClaimsPrincipal`) en el contexto de la solicitud actual. En términos más sencillos, le dice a la aplicación que el usuario ha sido autenticado y quién es ese usuario.

* **`Context`**: Representa el contexto HTTP actual, que incluye información sobre la solicitud y la respuesta.
* **`Scheme.Name`**: Es el nombre del esquema de autenticación que estás utilizando (en tu caso, "PasswordFlow").
* **`principal`**: Es el objeto `ClaimsPrincipal` que contiene la identidad del usuario, incluyendo sus reclamaciones (claims) y roles.

**Implicaciones con las cookies**

Por defecto, ASP.NET Core utiliza cookies para almacenar la información de autenticación cuando llamas a `SignInAsync`. Esto significa que, después de que el usuario se autentica, se crea una cookie que contiene una representación cifrada del `ClaimsPrincipal`. En solicitudes subsiguientes, ASP.NET Core lee esta cookie y restaura el `ClaimsPrincipal` en el contexto, permitiendo que la aplicación sepa que el usuario está autenticado.

**Almacenamiento de la información de inicio de sesión sin Bearer**

Dado que no estás utilizando tokens Bearer, las cookies son la opción más común y recomendada para mantener la sesión del usuario. Aquí tienes algunas consideraciones:

* **Cookies**:
    * **Ventajas**: Son la forma estándar de manejar la autenticación basada en sesiones en aplicaciones web. ASP.NET Core proporciona una excelente integración con cookies de autenticación.
    * **Desventajas**: Debes tener cuidado con la seguridad de las cookies (por ejemplo, configurarlas como `HttpOnly` y `Secure`).
* **Base de datos**:
    * **Ventajas**: Puedes almacenar información de sesión más compleja en una base de datos.
    * **Desventajas**: Requiere una infraestructura adicional y puede ser más lento que usar cookies. También debes implementar tu propia lógica para limpiar sesiones antiguas.

**¿Deberías crear una cookie o guardarla en una base de datos?**

* En la mayoría de los casos, usar cookies es la mejor opción. ASP.NET Core está diseñado para funcionar bien con cookies de autenticación.
* Solo deberías considerar una base de datos si tienes requisitos muy específicos, como almacenar grandes cantidades de datos de sesión o si necesitas un control muy granular sobre las sesiones de usuario.

**Recomendaciones adicionales**

* Asegúrate de configurar correctamente la seguridad de las cookies en tu aplicación.
* Considera utilizar la protección de datos de ASP.NET Core para cifrar y proteger tus cookies.
* Recuerda que si no utilizas el sistema de cookies de ASP.NET Core, deberás de crear tu propio sistema de administración de cookies.

Espero que esta explicación te ayude a comprender mejor cómo funciona `SignInAsync` y cómo puedes manejar la autenticación en tu aplicación.

<br />
<br />
<br />

---

<br />
<br />
<br />


¡Exacto! La declaración que mencionas sobre `AuthenticateAsync` es fundamental para entender el propósito y el flujo de un `AuthenticationHandler` en ASP.NET Core. Vamos a desglosar esto y aclarar tu duda.

**Entendiendo `AuthenticateAsync`**

La documentación dice:

> "An authentication scheme's authenticate action is responsible for constructing the user's identity based on request context. It returns an `AuthenticateResult` indicating whether authentication was successful and, if so, the user's identity in an authentication ticket."

Esto significa que `AuthenticateAsync` tiene la responsabilidad de:

1.  **Construir la identidad del usuario**: Basándose en el contexto de la solicitud HTTP (por ejemplo, headers, cookies, claims), debe determinar quién es el usuario.
2.  **Devolver `AuthenticateResult`**: Este resultado indica si la autenticación fue exitosa o no. Si fue exitosa, contiene la identidad del usuario en un `AuthenticationTicket`.

**Tu duda sobre `AuthenticateUser`**

Tu duda surge porque has creado `AuthenticateUser` para autenticar al usuario *fuera* del flujo normal de `AuthenticateAsync`. Esto es válido y útil, pero es importante entender cómo encaja en el flujo general.

**Explicación Detallada**

1.  **`AuthenticateAsync` (Flujo Normal)**:
    * Este método se llama automáticamente por el middleware de autenticación en cada solicitud que requiere autenticación.
    * Su propósito es *reconstruir* la identidad del usuario a partir del contexto de la solicitud (por ejemplo, leer un token de una cookie o header).
    * No debe realizar la autenticación inicial (verificar credenciales contra un IdP). Su trabajo es solo verificar si el usuario *ya* está autenticado.

2.  **`AuthenticateUser` (Autenticación Inicial)**:
    * Este método lo llamas *explícitamente* desde tu controlador o servicio cuando el usuario inicia sesión (por ejemplo, después de enviar sus credenciales en un formulario de login).
    * Su propósito es realizar la autenticación inicial: verificar las credenciales del usuario con el IdP y, si es exitoso, *establecer* la identidad del usuario en el contexto.
    * Después de que `AuthenticateUser` tiene éxito, `AuthenticateAsync` puede *reconstruir* la identidad del usuario en solicitudes subsiguientes.

**En resumen**

* `AuthenticateAsync` es para la autenticación *continua* (verificar si el usuario ya está autenticado).
* `AuthenticateUser` es para la autenticación *inicial* (autenticar al usuario por primera vez).

**Implicaciones**

* Debes asegurarte de que `AuthenticateUser` establezca la identidad del usuario de manera que `AuthenticateAsync` pueda reconstruirla correctamente. Esto generalmente implica guardar la información de autenticación en cookies o claims.
* Al separar la autenticación inicial de la continua, haces que tu código sea más modular y fácil de mantener.

Espero que esta explicación aclare tu duda.



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
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text.Encodings.Web;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.Extensions.Options;

namespace Reincar.Busquedas.WebReact.Services;

/// <summary>
/// Authentication handler para el flujo de concesión de contraseña (Resource Owner Password Credentials).
/// </summary>
public class PasswordFlowAuthenticationHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    private readonly IPasswordFlowService _ropcService;

    /// <summary>
    /// Constructor del PasswordFlowAuthenticationHandler.
    /// </summary>
    /// <param name="options">Opciones de autenticación.</param>
    /// <param name="logger">Logger para registrar eventos.</param>
    /// <param name="encoder">Encoder de URL.</param>
    /// <param name="ropcService">Servicio para interactuar con el Identity Provider.</param>
    public PasswordFlowAuthenticationHandler(IOptionsMonitor<AuthenticationSchemeOptions> options, ILoggerFactory logger, UrlEncoder encoder, IPasswordFlowService ropcService)
        : base(options, logger, encoder)
    {
        _ropcService = ropcService;
    }

    /// <summary>
    /// Maneja la autenticación de la solicitud actual.
    /// </summary>
    /// <returns>Resultado de la autenticación.</returns>
    protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        // Verifica si ya hay un principal autenticado en el contexto.
        var result = await Context.AuthenticateAsync(scheme: CookieAuthenticationDefaults.AuthenticationScheme);

        if (!result.Succeeded)
        {
            return result;
        }
        // Hay que asignar el principal que retorna la cookie a nuestro contexto de validación.
        // Al parecer las trata como dos instacias diferentes, que en efecto lo son.
        // pero el Scoped debería ser el mismo. ????
        Context.User = result.Principal;

        // Obtener el access_token del ClaimsPrincipal.
        var accessTokenClaim = result.Principal.FindFirst("access_token");

        if (accessTokenClaim == null || string.IsNullOrEmpty(accessTokenClaim.Value))
        {
            return AuthenticateResult.NoResult(); // No hay access_token en las claims.
        }

        var accessToken = accessTokenClaim.Value;

        // Validar el token de acceso.
        return await ValidateAccessToken(accessToken);
    }

    /// <summary>
    /// Valida el token de acceso y realiza el refresh si es necesario.
    /// </summary>
    /// <param name="accessToken">Token de acceso a validar.</param>
    /// <returns>Resultado de la autenticación.</returns>
    private async Task<AuthenticateResult> ValidateAccessToken(string accessToken)
    {
        try
        {
            // 1. Validar y decodificar el access_token.
            var handler = new JwtSecurityTokenHandler();
            var jwtSecurityToken = handler.ReadJwtToken(accessToken);

            // 2. Verificar la expiración del token.
            var expirationTime = jwtSecurityToken.ValidTo;
            if (expirationTime < DateTime.UtcNow.AddMinutes(5)) // Token expira en menos de 5 minutos.
            {
                // 3. Realizar el refresh token.
                var refreshTokenClaim = Context.User.FindFirst("refresh_token");
                if (refreshTokenClaim == null || string.IsNullOrEmpty(refreshTokenClaim.Value))
                {
                    return AuthenticateResult.Fail("Refresh token no encontrado en las claims.");
                }

                var refreshToken = refreshTokenClaim.Value;

                var refreshResult = await _ropcService.RefreshTokenAsync(refreshToken);
                return await refreshResult.Switch(
                    onSuccess: tokenResponse =>
                    {
                        // Actualizar ClaimsPrincipal con nuevos tokens.
                        return Success(tokenResponse);
                    },
                    onFailure: error =>
                    {
                        return Task.FromResult(AuthenticateResult.Fail(error.Message));
                    });
            }

            // 4. Si el token es válido, Context.User ya debería contener las claims del token validado.
            return AuthenticateResult.Success(new AuthenticationTicket(Context.User, Scheme.Name));
        }
        catch (Exception ex)
        {
            return AuthenticateResult.Fail($"Token de acceso inválido: {ex.Message}");
        }
    }

    /// <summary>
    /// Crea y guarda el principal autenticado en el contexto.
    /// </summary>
    /// <param name="token">Respuesta del token.</param>
    /// <returns>Resultado de la autenticación.</returns>
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
Services.AddAuthentication(configureOptions: action =>
{
    action.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    action.DefaultAuthenticateScheme = PasswordFlowAuthenticationDefaults.AuthenticationScheme;
})
    .AddCookie(authenticationScheme: CookieAuthenticationDefaults.AuthenticationScheme, configureOptions: action =>
    {
        action.Cookie.Name = ".Reincar.Web.Busquedas";
        action.LoginPath = "/#/Account/Login";
    })
    .AddScheme<AuthenticationSchemeOptions, PasswordFlowAuthenticationHandler>(
        authenticationScheme: PasswordFlowAuthenticationDefaults.AuthenticationScheme,
        configureOptions: action => { });

builder.Services.AddAuthorization();
```

Con estos cambios, la autenticación solo ocurrirá cuando llames al método `AuthenticateUser` en tu controlador, y las solicitudes subsiguientes utilizarán el token de acceso para autenticar al usuario.









