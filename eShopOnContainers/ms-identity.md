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



