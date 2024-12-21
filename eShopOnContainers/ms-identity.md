# [Make secure .NET Microservices and Web Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/secure-net-microservices-web-applications/)

Hay tantos aspectos sobre la seguridad en microservicios y aplicaciones web que el tema fácilmente podría ocupar varios libros como este.


## Implementación de la autenticación en microservicios y aplicaciones web de .NET

Cuando los servicios pueden ser acedidos directamente, se puede usar un microservicio de autenticación dedicado que actúe como un servicio de token de seguridad (STS) para autenticar a los usuarios.

# Autenticación
La autenticación es el proceso de determinar la identidad de un usuario. El mecanismo principal de ASP.NET Core para identificar a los usuarios de una aplicación es [ASP.NET Core Identity](https://docs.microsoft.com/aspnet/core/security/authentication/identity).

ASP .NET Core Identity se configura normalmente mediante una base de datos de SQL Server para almacenar nombres de usuario, contraseñas y datos de perfil. Como alternativa, se pueden utilizar otros almacenamientos de persistencia.

La autenticación con ASP.NET Core Identity funciona bien para muchos escenarios de aplicaciones web en los que es adecuado almacenar información de usuario (incluida la información de inicio de sesión, los roles y las notificaciones) en una cookie. Sin embargo, en escenarios como una API web de ASP.NET que expone puntos de conexión RESTful accesibles por aplicaciones SPA (Single-Page Application) o incluso otras APIs web, el uso de cookies no es adecuado. En estos casos, para habilitar una autenticación segura y escalable, ASP.NET Core ofrece soporte para [OAuth 2.0 y OpenID Connect](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols), que permiten la autenticación y autorización sin necesidad de gestionar cookies, facilitando la integración con aplicaciones distribuidas y servicios externos.

OpenID Connect (OIDC) amplía el `protocolo de autorización` OAuth 2.0 para su uso como otro protocolo de autenticación. Puede utilizar OIDC para habilitar el inicio de sesión único (SSO) entre sus aplicaciones habilitadas para OAuth mediante un token de seguridad denominado *ID token*.

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





      
El servidor de autorización emite los tokens de seguridad que usan sus aplicaciones y API para otorgar, denegar o revocar el acceso a los recursos (autorización) después de que el usuario haya iniciado sesión (autenticado).































