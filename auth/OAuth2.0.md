OAuth 2.0 是一个行业的标准授权协议。OAuth 2.0 专注于简化客户端开发人员，同时为 Web 应用程序，桌面应用程序，手机和客厅设备提供特定的授权流程。

它的最终目的是为第三方应用颁发一个有时效性的令牌 token。使得第三方应用能够通过该令牌获取相关的资源。常见的场景就是：第三方登录。当你想要登录某个论坛，但没有账号，而这个论坛接入了如 QQ、Facebook 等登录功能，在你使用 QQ 登录的过程中就使用的 OAuth 2.0 协议。

官方网址：https://oauth.net/2/

## 角色
首先需要介绍的是 OAuth 2.0 协议中的一些角色，整个授权协议的流程都将围绕着这些角色：

* **Resource Owner**，资源所有者，能够允许访问受保护资源的实体。如果是个人，被称为 end-user。
* **Resource Server**，资源服务器，托管受保护资源的服务器。
* **Client**，客户端，使用资源所有者的授权代表资源所有者发起对受保护资源的请求的应用程序。如：web网站，移动应用等。
* **Authorization Server**，授权服务器，能够向客户端颁发令牌。
* **User-Agent**，用户代理，帮助资源所有者与客户端沟通的工具，一般为 web 浏览器，移动 APP 等。

假如我想要在 coding.net 这个网站上用 github.com 的账号登录。那么 coding 相对于 github 就是一个客户端。而我们用什么操作的呢？浏览器，这就是一个用户代理。当从 github 的授权服务器获得 token 后，coding 是需要请求 github 账号信息的，从哪请求？从 github 的资源服务器。

## 协议流程
oauth2-roles
* (A) Client 请求 Resource Owner 的授权。授权请求可以直接向 Resource Owner 请求，也可以通过 Authorization Server 间接的进行。
* (B) Client 获得授权许可。
* (C) Client 向 Authorization Server 请求访问令牌。
* (D) Authorization Server 验证授权许可，如果有效则颁发访问令牌。
* (E) Client 通过访问令牌从 Resource Server 请求受保护资源。
* (F) Resource Server 验证访问令牌，有效则响应请求。
```
    +--------+                               +---------------+
    |        |--(A)- Authorization Request ->|   Resource    |
    |        |                               |     Owner     |
    |        |<-(B)-- Authorization Grant ---|               |
    |        |                               +---------------+
    |        |
    |        |                               +---------------+
    |        |--(C)-- Authorization Grant -->| Authorization |
    | Client |                               |     Server    |
    |        |<-(D)----- Access Token -------|               |
    |        |                               +---------------+
    |        |
    |        |                               +---------------+
    |        |--(E)----- Access Token ------>|    Resource   |
    |        |                               |     Server    |
    |        |<-(F)--- Protected Resource ---|               |
    +--------+                               +---------------+
                    Figure 1: Abstract Protocol Flow
```
## 授权
一个客户端想要获得授权，就需要先到服务商那注册你的应用。一般需要你提供下面这些信息：
* 应用名称
* 应用网站
* 重定向 URI 或回调 URL（redirect_uri）
** 重定向 URI 是服务商在用户授权（或拒绝）应用程序之后重定向用户的地址，因此也是用于处理授权代码或访问令牌的应用程序的一部分。在你注册

成功之后，你会从服务商那获取到你的应用相关的信息：
* 客户端标识 client_id
* 客户端密钥 client_secret
** client_id 用来表识客户端（公开），client_secret 用来验证客户端身份（保密）。

### 授权类型
OAuth 2.0 列举了四种授权类型，分别用于不同的场景：
* Authorization Code（授权码 code）：服务器与客户端配合使用。
* Implicit（隐式 token）：用于移动应用程序或 Web 应用程序（在用户设备上运行的应用程序）。
* Resource Owner Password Credentials（资源所有者密码凭证 password）：资源所有者和客户端之间具有高度信任时（例如，客户端是设备的操作系统的一部分，或者是一个高度特权应用程序），以及当其他授权许可类型（例如授权码）不可用时被使用。
* Client Credentials（客户端证书 client_credentials）：当客户端代表自己表演（客户端也是资源所有者）或者基于与授权服务器事先商定的授权请求对受保护资源的访问权限时，客户端凭据被用作为授权许可。

下面来具体说说这四种授权。注意重定向一定要用 302。

#### 授权码模式

该方式需要资源服务器的参与，应用场景大概是：

* 资源拥有者（用户）需要登录客户端（APP），他选择了第三方登录。
* 客户端（APP）重定向到第三方授权服务器。此时客户端携带了客户端标识（client_id），那么第三方就知道这是哪个客户端，资源拥有者确定（拒绝）授权后需要重定向到哪里。
* 用户确认授权，客户端（APP）被重定向到注册时给定的 URI，并携带了第三方给定的 code。
* 在重定向的过程中，客户端拿到 code 与 client_id、client_secret 去授权服务器请求令牌，如果成功，直接请求资源服务器获取资源，整个过程，用户代理是不会拿到令牌 token 的。
* 客户端（APP）拿到令牌 token 后就可以向第三方的资源服务器请求资源了。
```
    +----------+
    | Resource |
    |   Owner  |
    |          |
    +----------+
         ^
         |
        (B)
    +----|-----+          Client Identifier      +---------------+
    |         -+----(A)-- & Redirection URI ---->|               |
    |  User-   |                                 | Authorization |
    |  Agent  -+----(B)-- User authenticates --->|     Server    |
    |          |                                 |               |
    |         -+----(C)-- Authorization Code ---<|               |
    +-|----|---+                                 +---------------+
      |    |                                         ^      v
     (A)  (C)                                        |      |
      |    |                                         |      |
      ^    v                                         |      |
    +---------+                                      |      |
    |         |>---(D)-- Authorization Code ---------'      |
    |  Client |          & Redirection URI                  |
    |         |                                             |
    |         |<---(E)----- Access Token -------------------'
    +---------+       (w/ Optional Refresh Token)
   Note: The lines illustrating steps (A), (B), and (C) are broken into
   two parts as they pass through the user-agent.
                  Figure 3: Authorization Code Flow
```

具体说明，这里以 coding 和 github 为例。当我想在 coding 上通过 github 账号登录时：

##### GET 请求 点击登录，重定向到 github 的授权端点

1. 请求
   ```
   https://github.com/login/oauth/authorize?
       response_type=code&
       client_id=a5ce5a6c7e8c39567ca0&
       redirect_uri=https://coding.net/api/oauth/github/callback&
       scope=user:email
   ```
   - response_type 必须，固定为 code，表示这是一个授权码请求。
   - client_id 必须，在 github 注册获得的客户端 ID。
   - redirect_uri 重新导向的 url
   - scope 可选，请求资源范围，多个空格隔开
   - state 可选（推荐），如果存在，原样返回给客户端。
2. 回调
   ```
   https://coding.net/api/oauth/github/callback?code=fb6a88dc09e843b33f
   ```
   - code 必须。授权码
   - state 如果出现在请求中，必须包含。

授权错误：
1. 第一种，客户端没有被识别或错误的重定向 URI，授权服务器没有必要重定向资源拥有者到重定向URI，而是通知资源拥有者发生了错误。
2. 第二种，客户端被正确地授权了，但是其他某些事情失败了。这种情况下下面地错误响应会被发送到客户端，包括在重定向 URI 中。
   ```shell
   https://coding.net/api/oauth/github/callback?
       error=redirect_uri_mismatch&
       error_description=The+redirect_uri+MUST+match+the+registered+callback+URL+for+this+application.&
       error_uri=https%3A%2F%2Fdeveloper.github.com%2Fapps%2Fmanaging-oauth-apps%2Ftroubleshooting-authorization-request-errors%2F%23redirect-uri-mismatch
   ```
      - error：必须，必须是预先定义的错误码。
      - error_description：可选，错误描述
      - error_uri：可选，指向可解读错误的 URI
      - state：如果出现在授权请求中，必须

##### POST 请求 获取令牌 token

1. 当获取到授权码 code 后，客户端需要用它获取访问令牌：
   ```shell
   https://github.com/login/oauth/access_token?client_id=a5ce5a6c7e8c39567ca0&client_secret=xxxx&grant_type=authorization_code&code=fb6a88dc09e843b33f&redirect_uri=https://coding.net/api/oauth/github/callback
   ```
      * client_id	必须，客户端标识。
      * client_secret	必须，客户端密钥。
      * grant_type	必须，固定为 authorization_code／refresh_token。
      * code	必须，上一步获取到的授权码。
      * redirect_uri	必须，完成授权后的回调地址，与注册时一致。
2. 返回值：
   ```json
   {
      "access_token":"a14afef0f66fcffce3e0fcd2e34f6ff4",
       "token_type":"bearer",
       "expires_in":3920,
       "refresh_token":"5d633d136b6d56a41829b73a424803ec"
   }
   ```
      * access_token	这个就是最终获取到的令牌。
      * token_type	令牌类型，常见有 bearer/mac/token（可自定义）。
      * expires_in	失效时间。
      * refresh_token	刷新令牌，用来刷新 access_token。

##### 获取资源服务器资源

拿着 access_token 就可以获取账号的相关信息了：
```shell
curl -H "Authorization: token a14afef0f66fcffce3e0fcd2e34f6ff4" https://api.github.com/user
```

##### POST 请求 刷新令牌

1. access_token 是有时效性的，当在获取 github 用户信息时，如果返回 token 过期：
   ```shell
   https://github.com/login/oauth/access_token?
       client_id=a5ce5a6c7e8c39567ca0&
       client_secret=xxxx&
       redirect_uri=https://coding.net/api/oauth/github/callback&
       grant_type=refresh_token&
       efresh_token=5d633d136b6d56a41829b73a424803ec
   ```
      * client_id	必须
      * client_secret	必须
      * redirect_uri	必须
      * grant_type	必须，固定为 refresh_token
      * refresh_token	必须，上面获取到的 refresh_token
2. 返回值：
   ```json
   {
       "access_token":"a14afef0f66fcffce3e0fcd2e34f6ee4",
       "token_type":"bearer",
       "expires_in":3920,
       "refresh_token":"4a633d136b6d56a41829b73a424803vd"
   }
   ```

refresh_token 只有在 access_token 过期时才能使用，并且只能使用一次。当换取到的 access_token 再次过期时，使用新的 refresh_token 来换取 access_token

```
  +--------+                                           +---------------+
  |        |--(A)------- Authorization Grant --------->|               |
  |        |                                           |               |
  |        |<-(B)----------- Access Token -------------|               |
  |        |               & Refresh Token             |               |
  |        |                                           |               |
  |        |                            +----------+   |               |
  |        |--(C)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(D)- Protected Resource --| Resource |   | Authorization |
  | Client |                            |  Server  |   |     Server    |
  |        |--(E)---- Access Token ---->|          |   |               |
  |        |                            |          |   |               |
  |        |<-(F)- Invalid Token Error -|          |   |               |
  |        |                            +----------+   |               |
  |        |                                           |               |
  |        |--(G)----------- Refresh Token ----------->|               |
  |        |                                           |               |
  |        |<-(H)----------- Access Token -------------|               |
  +--------+           & Optional Refresh Token        +---------------+
               Figure 2: Refreshing an Expired Access Token
```

#### 隐式模式

该方式一般用于移动客户端或网页客户端。隐式授权类似于授权码授权，但 token 被返回给用户代理再转发到客户端（APP），因此它可能会暴露给用户和用户设备上的其它客户端（APP）。此外，此流程不会对客户端（APP）的身份进行身份验证，并且依赖重定向 URI（已在服务商中注册）来实现此目的。

基本原理：要求用户授权应用程序，然后授权服务器将访问令牌传回给用户代理，用户代理将其传递给客户端。
```
    +----------+
    | Resource |
    |  Owner   |
    |          |
    +----------+
         ^
         |
        (B)
    +----|-----+          Client Identifier     +---------------+
    |         -+----(A)-- & Redirection URI --->|               |
    |  User-   |                                | Authorization |
    |  Agent  -|----(B)-- User authenticates -->|     Server    |
    |          |                                |               |
    |          |<---(C)--- Redirection URI ----<|               |
    |          |          with Access Token     +---------------+
    |          |            in Fragment
    |          |                                +---------------+
    |          |----(D)--- Redirection URI ---->|   Web-Hosted  |
    |          |          without Fragment      |     Client    |
    |          |                                |    Resource   |
    |     (F)  |<---(E)------- Script ---------<|               |
    |          |                                +---------------+
    +-|--------+
      |    |
     (A)  (G) Access Token
      |    |
      ^    v
    +---------+
    |         |
    |  Client |
    |         |
    +---------+
  Note: The lines illustrating steps (A) and (B) are broken into two
  parts as they pass through the user-agent.
                      Figure 4: Implicit Grant Flow
```

同样以 coding.net 和 github 为例：

1. 提交请求
   ```shell
   https://github.com/login/oauth/authorize?
       response_type=token&
       client_id=a5ce5a6c7e8c39567ca0&
       redirect_uri=https://coding.net/api/oauth/github/callback&
       scope=user:email
   ```
      * response_type 必须，固定为 token。
      * client_id 必须。当客户端被注册时，有授权服务器分配的客户端标识。
      * redirect_uri 可选。由客户端注册的重定向URI。
      * scope 可选。请求可能的作用域。
      * state 可选(推荐)。任何需要被传递到客户端请求的URI客户端的状态。
   回调接口
   ```shell
   https://coding.net/api/oauth/github/callback#
       access_token=a14afef0f66fcffce3e0fcd2e34f6ff4&
       token_type=token&
       expires_in=3600
   ```
      * access_token	必须。授权服务器分配的访问令牌。
      * token_type	必须。令牌类型。
      * expires_in	推荐，访问令牌过期的秒数。
      * scope	可选，访问令牌的作用域。
      * state	必须，如果出现在授权请求期间，和请求中的 state 参数一样。
   授权错误和上面一样
2. 用户代理提取令牌 token 并提交给 coding
3. coding.net 拿到 token 就可以获取用户信息了
   ```shell
   curl -H "Authorization: token a14afef0f66fcffce3e0fcd2e34f6ff4" https://api.github.com/user
   ```

#### 资源所有者密码模式

用户将其服务凭证（用户名和密码）直接提供给客户端，该客户端使用凭据从服务获取访问令牌。如果其它方式不可行，则只应在授权服务器上启用该授权类型。此外，只有在客户端受到用户信任时才能使用它（例如，它由服务商自有，或用户的桌面操作系统）。

```
    +----------+
    | Resource |
    |  Owner   |
    |          |
    +----------+
         v
         |    Resource Owner
        (A) Password Credentials
         |
         v
    +---------+                                  +---------------+
    |         |>--(B)---- Resource Owner ------->|               |
    |         |         Password Credentials     | Authorization |
    | Client  |                                  |     Server    |
    |         |<--(C)---- Access Token ---------<|               |
    |         |    (w/ Optional Refresh Token)   |               |
    +---------+                                  +---------------+
           Figure 5: Resource Owner Password Credentials Flow
```

- POST 请求 密码凭证流程
   ```
   https://oauth.example.com/token?grant_type=password&username=USERNAME&password=PASSWORD&client_id=CLIENT_ID
   ```
   * grant_type 必须，固定为 password。
   * username 必须，UTF-8 编码的资源拥有者用户名。
   * password 必须，UTF-8 编码的资源拥有者密码。
   * scope 可选，授权范围。
- 返回值：
   ```json
   {
       "access_token"  : "...",
       "token_type"    : "...",
       "expires_in"    : "...",
       "refresh_token" : "...",
   }
   ```
如果授权服务器验证成功，那么将直接返回令牌 token，改客户端已被授权。

#### 客户端模式

这种模式只需要提供 `client_id` 和 `client_secret` 即可获取授权。一般用于后端 API 的相关操作。
```
    +---------+                                  +---------------+
    |         |                                  |               |
    |         |>--(A)- Client Authentication --->| Authorization |
    | Client  |                                  |     Server    |
    |         |<--(B)---- Access Token ---------<|               |
    |         |                                  |               |
    +---------+                                  +---------------+
                    Figure 6: Client Credentials Flow
```

- POST 请求 客户端凭证流程：
   ```shell
   https://oauth.example.com/token?grant_type=client_credentials&client_id=CLIENT_ID&client_secret=CLIENT_SECRET
   ```
   - grant_type 必须。必须设置到客户端证书中。
   - scope 可选。授权的作用域。
- 返回值
   ```
   {
      "access_token"  : "...",
      "token_type"    : "...",
      "expires_in"    : "...",
   }
   ```

如果授权服务器验证成功，那么将直接返回令牌`token`，该客户端已被授权。

## 参考
* https://developers.douban.com/wiki/?title=oauth2
* https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2
* https://tools.ietf.org/html/rfc6749
* https://oauth.net/2/
* [OAuth2授权](https://www.cnblogs.com/linianhui/p/oauth2-authorization.html)
