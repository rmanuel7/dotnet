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




