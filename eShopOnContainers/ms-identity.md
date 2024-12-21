# [Make secure .NET Microservices and Web Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/secure-net-microservices-web-applications/)

Hay tantos aspectos sobre la seguridad en microservicios y aplicaciones web que el tema fácilmente podría ocupar varios libros como este.


## Implementación de la autenticación en microservicios y aplicaciones web de .NET

Cuando los servicios pueden ser acedidos directamente, se puede usar un microservicio de autenticación dedicado que actúe como un servicio de token de seguridad (STS) para autenticar a los usuarios.

# Autenticación
La autenticación es el proceso de determinar la identidad de un usuario. El mecanismo principal de ASP.NET Core para identificar a los usuarios de una aplicación es [ASP.NET Core Identity](https://docs.microsoft.com/aspnet/core/security/authentication/identity).

ASP .NET Core Identity se configura normalmente mediante una base de datos de SQL Server para almacenar nombres de usuario, contraseñas y datos de perfil. Como alternativa, se pueden utilizar otros almacenamientos de persistencia.

La autenticación con ASP.NET Core Identity funciona bien para muchos escenarios de aplicaciones web en los que es adecuado almacenar información de usuario (incluida la información de inicio de sesión, los roles y las notificaciones) en una cookie. Sin embargo, en escenarios como una API web de ASP.NET que expone puntos de conexión RESTful accesibles por aplicaciones SPA (Single-Page Application) o incluso otras APIs web, el uso de cookies no es adecuado. En estos casos, para habilitar una autenticación segura y escalable, ASP.NET Core ofrece soporte para [OAuth 2.0 y OpenID Connect](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols), que permiten la autenticación y autorización sin necesidad de gestionar cookies, facilitando la integración con aplicaciones distribuidas y servicios externos.

OpenID Connect (OIDC) amplía el `protocolo de autorización` OAuth 2.0 para su uso como otro protocolo de autenticación. Puede utilizar OIDC para habilitar el inicio de sesión único (SSO) entre sus aplicaciones habilitadas para OAuth mediante un token de seguridad denominado *ID token*.

> [!NOTE]
> Tanto el cliente de la aplicación como el usuario de la aplicación se autentican en el flujo confidencial. El cliente de la aplicación utiliza un secreto de cliente o una aserción de cliente para autenticarse.

Mientras que `ASP.NET Core Identity` maneja la administración de usuarios (como registro, inicio de sesión y roles dentro de una sola aplicación), `OAuth 2.0 y OpenID Connect` permiten al cliente autenticarse una vez y usar los tokens para acceder a los recursos, lo que garantiza que el flujo de autenticación y autorización se pueda compartir entre diferentes aplicaciones o servicios en un sistema Single Sign-On (SSO).

Si prefiere emitir tokens de forma local en lugar de usar un proveedor de identidades externo, puede aprovechar algunas bibliotecas de terceros que son buenas. `IdentityServer4` y `OpenIddict` son proveedores de OpenID Connect que se integran fácilmente con `ASP.NET Core Identity` para permitirle emitir tokens de seguridad desde un servicio ASP.NET Core.

Así es como funciona en el contexto de ASP.NET Core y SSO:

1. **Autenticación en el cliente:** el usuario inicia sesión a través de un proveedor de identidad (por ejemplo, via OpenID Connect u OAuth 2.0) en la aplicación cliente. Una vez autenticado, el cliente recibe un token (como un **ID token** o un **access token**).

2. **Autorización para acceder a los recursos:** el token de acceso, que a menudo se transmite junto con cada solicitud, demuestra que el cliente está autorizado para acceder a recursos protegidos (por ejemplo, API). Este token lo emite el proveedor de identidad (IdP) después de que el usuario se autentica y autoriza al cliente a realizar acciones en nombre del usuario.

3. **Integración de ASP.NET Core:** si bien ASP.NET Core Identity se encarga de la administración de usuarios (como el registro, el inicio de sesión y los roles dentro de una sola aplicación), OAuth 2.0 y OpenID Connect permiten que el cliente se autentique una vez y use los tokens para acceder a los recursos, lo que garantiza que el flujo de autenticación y autorización se pueda compartir entre diferentes aplicaciones o servicios en un sistema SSO.

![image](./openid-oauth.svg)

> [!IMPORTANT]
> Los `Identity Providers (IdPs)`, como `IdentityServer4`, `OpenIddict` o `Azure AD`, implementan los protocolos `OpenID Connect y OAuth 2.0` para gestionar la autenticación y la emisión de tokens.


## Roles en OAuth 2.0

En general, hay cuatro partes involucradas en un intercambio de autenticación y autorización de OAuth 2.0 y OpenID Connect. Estos intercambios suelen denominarse *authentication flows* o *auth flows*.

### Servidor de autorización
También llamado proveedor de identidad o IdP, maneja de forma segura la información del usuario final, su acceso y las relaciones de confianza entre las partes en el flujo de autenticación. Estas soluciones ofrecen características y capacidades para administrar la autenticación, autorización e identidad del usuario dentro de una aplicación.

  - **Identidad**
    <br/>Las identidades se utilizan para autenticar y autorizar el acceso a los recursos
    
    - **ASP.NET Core Identity**
      <br/>Proporciona un marco para administrar y almacenar cuentas de usuario en aplicaciones ASP.NET Core
      
      ```csharp
      using Microsoft.AspNetCore.Identity;
      using System.ComponentModel.DataAnnotations;

      // Add profile data for application users by adding properties to the ApplicationUser class
      public class ApplicationUser : IdentityUser
      {
              [Required]
              public string CardNumber { get; set; }
              [Required]
              public string SecurityNumber { get; set; }
              [Required]
              [RegularExpression(@"(0[1-9]|1[0-2])\/[0-9]{2}", ErrorMessage = "Expiration should match a valid MM/YY value")]
              public string Expiration { get; set; }
              [Required]
              public string CardHolderName { get; set; }
              public int CardType { get; set; }
              [Required]
              public string Street { get; set; }
              [Required]
              public string City { get; set; }
              [Required]
              public string State { get; set; }
              [Required]
              public string Country { get; set; }
              [Required]
              public string ZipCode { get; set; }
              [Required]
              public string Name { get; set; }
              [Required]
              public string LastName { get; set; }
      }
      ```
    - **OpenIddict.AspNetCore**
      <br/>Proporciona un marco para registrar y administrar aplicaciones cliente que solicitarán autenticación y autorización para acceder a recursos en nombre de los usuarios.
      
      ```csharp
      using Microsoft.Extensions.Configuration;
      using OpenIddict.Abstractions;
      using static OpenIddict.Abstractions.OpenIddictConstants;
      
      // Confidential Client
      new OpenIddictApplicationDescriptor
      {
          ClientId = "mvc",
          DisplayName = "MVC Client",
          ApplicationType = "Web",
          ClientType = ClientTypes.Confidential,
          ClientSecret = "secret", // Confidential apps require a secret
          RedirectUris =
          {
              new Uri($"{clientsUrl["Mvc"]}/signin-oidc")
          },
          PostLogoutRedirectUris =
          {
              new Uri($"{clientsUrl["Mvc"]}/signout-callback-oidc")
          },
          Permissions =
          {
              Permissions.Endpoints.Authorization,
              Permissions.Endpoints.Token,

              Permissions.GrantTypes.AuthorizationCode,
              Permissions.GrantTypes.ClientCredentials,
              Permissions.GrantTypes.RefreshToken,

              Permissions.ResponseTypes.Code,

              Permissions.Prefixes.Scope + Scopes.Profile, // Ensure 'profile' scope is listed
              Permissions.Prefixes.Scope + Scopes.OpenId,
              Permissions.Prefixes.Scope + Scopes.OfflineAccess,
              Permissions.Prefixes.Scope + "orders",
              Permissions.Prefixes.Scope + "basket",
              Permissions.Prefixes.Scope + "locations",
              Permissions.Prefixes.Scope + "marketing",
              Permissions.Prefixes.Scope + "webshoppingagg",
              Permissions.Prefixes.Scope + "orders.signalrhub"
          }
      },
      ```
      
      > **NOTA**
      > <br/>[ClientType](https://learn.microsoft.com/en-us/entra/msal/dotnet/getting-started/scenarios)
      > <br/>Los diferentes tipos de aplicaciones cliente que interactúan con un servidor de autorización se dividen en dos categorías:
      > * [ClientTypes.Public](https://learn.microsoft.com/en-us/entra/identity-platform/v2-app-types#mobile-and-native-apps) Las aplicaciones de cliente públicas (de escritorio y móviles), son las que realizan funciones en nombre de un usuario. Estas aplicaciones pueden agregar inicio de sesión y autorización a los servicios de back-end mediante el [OAuth 2.0 authorization code grant type, o auth code flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow).
      >   - El authorization code flow comienza cuando el cliente dirige al usuario a el `/authorize` endpoint.
      >   - Ahora que ha adquirido un código de autorización debe enviar una solicitud POST al `/token` endpoint para obtener el `access_token`.
      >   - Tiene sentido habilitar los siguentes permisos:
      >     - `Permissions.Endpoints.Authorization`
      >     - `Permissions.Endpoints.Token`
      >     - `Permissions.GrantTypes.AuthorizationCode`
      >     - `Permissions.ResponseTypes.Code`
      > * [ClientTypes.Confidential](https://learn.microsoft.com/en-us/entra/identity-platform/v2-app-types#server-daemons-and-scripts) Aplicaciones de cliente confidenciales (aplicaciones web, API web y aplicaciones demonio, de escritorio o web). Las aplicaciones que tienen procesos de larga duración o que funcionan sin interacción con un usuario también necesitan una forma de acceder a recursos seguros, como las API web. Estas aplicaciones pueden autenticarse y obtener tokens mediante el uso de un `ClientId`, en lugar de la identidad delegada de un usuario, con el [OAuth 2.0 client credentials flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-client-creds-grant-flow). Puede probar la identidad de la aplicación mediante `ClientSecret` o un certificado.
      >   - En este flujo, la aplicación interactúa directamente con el `/token` endpoint para obtener acceso.
      >   - Tiene sentido habilitar los siguentes permisos:
      >     - `Permissions.Endpoints.Token`,
      >     - `Permissions.GrantTypes.ClientCredentials`
   
      
      > **NOTA**
      > <br/>[Permissions.GrantTypes.RefreshToken](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#refresh-the-access-token)
      > <br/>Los `access_token` tienen una vida útil corta. La aplicación puede usar `refresh_token` para adquirir otros access_token después de que caduque el access_token actual. Puede hacerlo enviando otra solicitud POST al `/token` endpoint.
      > * Solo se proporciona si se habilito los siguentes permisos:
      >   - `Permissions.Prefixes.Scope + Scopes.OfflineAccess`
      
      > **NOTA**
      > <br/>[ID token](https://learn.microsoft.com/en-us/entra/identity-platform/id-tokens)
      > <br/>Los ID token son un tipo de token de seguridad que sirve como prueba de autenticación y confirma que un usuario se ha autenticado correctamente.
      > * Un JSON Web Token.
      > * La aplicación puede decodificar los segmentos de este token para solicitar información sobre el usuario que inició sesión.
      > * Este enfoque se denomina *hybrid flow* porque combina OIDC con el OAuth2 authorization code flow.
      > * Solo se proporciona si se habilito los siguentes permisos:
      >   - `Permissions.Prefixes.Scope + Scopes.OpenId`  Para emitir el token
      >   - `Permissions.Prefixes.Scope + Scopes.Profile`  Para incluir información del usuario en el token

      > **NOTA**
      > <br/>[RedirectUris](https://learn.microsoft.com/en-us/aspnet/core/blazor/security/blazor-web-app-with-oidc?view=aspnetcore-9.0&pivots=without-bff-pattern#configure-the-app)
      > <br/>El proveedor OpendID Connect establece estos valores por defecto `/signin-oidc` y `/signout-callback-oidc`.
      > <br/>Al utilizar el middleware ASP.NET Core OpenID Connect, el `/signin-oidc` endpoint se maneja automáticamente.






      
El servidor de autorización emite los tokens de seguridad que usan sus aplicaciones y API para otorgar, denegar o revocar el acceso a los recursos (autorización) después de que el usuario haya iniciado sesión (autenticado).










## Referencias
1. Endpoints
   * [authorization endpoint and token endpoint](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols#endpoints)
   * [Request an authorization code](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#request-an-authorization-code)
   * [Request an access token with a client_secret](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#request-an-access-token-with-a-client_secret)
2. Refresh Token
   * [Refresh the access token](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#refresh-the-access-token)
   * [Successful response](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#successful-response-2)
3. ID Token
   * [Successful response](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#successful-response-2)
   * [Request an ID token as well or hybrid flow](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow#request-an-id-token-as-well-or-hybrid-flow)





