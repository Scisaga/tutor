OIDC = (Identity, Authentication) + OAuth 2.0。它在OAuth2上构建了一个身份层，是一个基于OAuth2协议的身份认证标准协议。OAuth2是一个授权协议，它无法提供完善的身份认证功能，OIDC使用OAuth2的授权服务器来为第三方客户端提供用户的身份认证，并把对应的身份认证信息传递给客户端，且可以适用于各种类型的客户端（比如服务端应用，移动APP，JS应用），且完全兼容OAuth2，也就是说你搭建了一个OIDC的服务后，也可以当作一个OAuth2的服务来用。

OIDC = JWT + OAuth2 + IDP == SSO有效实现方式

## 核心概念
OAuth2提供了Access Token来解决授权第三方客户端访问受保护资源的问题；OIDC在这个基础上提供了ID Token来解决第三方客户端标识用户身份认证的问题。OIDC的核心在于在OAuth2的授权流程中，一并提供用户的身份认证信息（ID Token）给到第三方客户端，ID Token使用JWT格式来包装，得益于JWT（JSON Web Token）的自包含性，紧凑性以及防篡改机制，使得ID Token可以安全的传递给第三方客户端程序并且容易被验证。此外还提供了UserInfo的接口，用户获取用户的更完整的信息。

### 术语
* **EU**：End User：一个人类用户。
* **RP**：Relying Party ,用来代指OAuth2中的受信任的客户端，身份认证和授权信息的消费方；
* **OP**：OpenID Provider，有能力提供EU认证的服务（比如OAuth2中的授权服务），用来为RP提供EU的身份认证信息；
* **ID Token**：JWT格式的数据，包含EU身份认证的信息。
* **UserInfo Endpoint**：用户信息接口（受OAuth2保护），当RP使用Access Token访问时，返回授权用户的信息，此接口必须使用HTTPS；
* **IDP**：身份提供商，如github/微信

### 基本工作流程
# **RP**发送一个认证请求给**OP**；// 通常前端实现
# **OP**对**EU**进行身份认证，然后提供授权；
# **OP**把**ID Token**和**Access Token**（需要的话）返回给**RP** // 前端拿到Access Token，传给后端
# **RP**使用**Access Token**发送一个请求**UserInfo EndPoint**；// 通常后端实现
# **UserInfo EndPoint**返回**EU**的**Claims**。// 通常后端实现

OIDC的AuthN请求中scope参数必须要有一个值为的openid的参数

## ID Token

## 认证

## 总结

- OIDC使得身份认证可以作为一个服务存在。
- OIDC可以很方便的实现SSO（跨顶级域）。
- OIDC兼容OAuth2，可以使用Access Token控制受保护的API资源。
- OIDC可以兼容众多的IDP作为OIDC的OP来使用。
- OIDC的一些敏感接口均强制要求TLS，除此之外，得益于JWT，JWS，JWE家族的安全机制，使得一些敏感信息可以进行数字签名、加密和验证，进一步确保整个认证过程中的安全保障。

## 参考
* [OIDC（OpenId Connect）身份认证（核心部分）](https://www.cnblogs.com/linianhui/p/openid-connect-core.html)
* [读懂 SSO、OAuth 2.0、OpenID Connect](http://jiangew.me/sso-openid-connect/)
* [Diagrams of All The OpenID Connect Flows](https://medium.com/@darutk/diagrams-of-all-the-openid-connect-flows-6968e3990660)
