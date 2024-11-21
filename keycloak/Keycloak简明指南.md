# Keycloak 简明指南

## 什么是Keycloak

Keycloak 是一个开源的身份和访问管理工具（IAM，Identity and Access Management），为应用程序和服务提供单点登录（SSO）、用户身份验证和授权等功能。它由 Red Hat 开发，旨在简化用户身份管理，并为开发人员提供一套易于集成的安全解决方案。

### 核心功能

1. **单点登录 (SSO)**
   - 用户只需登录一次，即可访问所有集成的应用程序，无需重复输入登录凭据。
2. **身份验证**
   - 支持用户名密码、社交登录（如 Google 和 Facebook）、LDAP、Active Directory，以及双因素认证（2FA）。
3. **细粒度授权策略**
   - **基于属性的权限控制 (ABAC)**：使用属性匹配规则进行授权，避免资源和角色的紧密耦合，适合复杂权限管理场景。
   - **基于角色的权限控制 (RBAC)**：基于用户角色分配权限，适用于大多数传统权限管理需求。
   - **基于用户的权限控制 (UBAC)**：针对个别用户定义专属权限，适用于特定资源的访问控制。
   - **基于上下文的权限控制 (CBAC)**：根据运行时的上下文条件（如设备或位置）动态决定权限。
   - **基于规则的权限控制**：使用 JavaScript 创建复杂的自定义授权规则。
   - **基于时间的权限控制**：设置时间段条件（如工作时间内访问）进行权限控制。
4. **社交登录**
   - 提供 Google、Facebook 和 GitHub 等主流社交媒体账户的快速集成登录功能。
5. **用户管理**
   - 支持用户注册、登录、密码重置、账户管理等，并灵活配置用户组和角色。
   - 提供集中式的资源和权限管理，通过 REST API 和 UI 支持授权决策，快速响应安全需求变化。
6. **协议支持**
   - 兼容 OAuth 2.0、OpenID Connect (OIDC)、SAML 等标准协议，方便与其他系统集成。
7. **可扩展性**
   - 支持插件和自定义扩展，满足多样化的功能需求和界面调整需求。
8. **多租户支持**
   - 在单个 Keycloak 实例中，为多个应用或组织提供独立的身份和访问管理服务。

### 应用场景
1. 企业应用：实现员工对内部系统的单点登录。
2. B2C 应用：为用户提供便捷的注册、登录和认证体验。
3. 微服务架构：在分布式系统中集中管理认证和授权。
4. API 保护：通过 OAuth 2.0 和 JWT 为 API 提供安全访问。

### 为什么选择 Keycloak？
1. 开源且免费：无需额外的授权费用，灵活性高。
2. 易于部署：支持 Docker 和 Kubernetes 等容器化部署方式。
3. 社区支持：拥有活跃的开发者社区，持续更新和维护。
4. 可定制性：允许根据特定需求自定义主题和功能。

### 基础概念

#### Realm（域）
- **定义**：域是 Keycloak 中的核心概念，用于管理一组用户、凭据、角色和组。
- **功能**：用户属于某个域，并能登录到该域。域之间彼此隔离，确保独立管理和验证用户。
- **用途**：在一个 Keycloak 实例中，可以创建多个域，用于支持不同的应用、客户或组织。

#### Client（客户端）
- **定义**：客户端是一个实体，可以请求 Keycloak 对用户进行身份验证。通常，客户端代表一个应用或服务。
- **功能**：
   - 客户端请求 Keycloak 提供安全的单点登录（SSO）机制。
   - 客户端可以获取 Access Token，用于以认证用户的身份调用其他服务。
   - 开启资源保护（Authorization）时，客户端也可以作为资源服务器。
- **两种类型**：
   - **公共客户端**：不需要机密凭据（如浏览器应用）。
   - **机密客户端**：需要机密凭据，用于后端服务。

#### Role（角色）
- **定义**：角色是用户的类型或类别，如管理员、用户、经理等。
- **功能**：
   - 应用通过角色分配权限，而非直接对用户赋权，简化权限管理。
   - 角色分为两种：
      - **Realm 级别角色**：跨客户端共享。
      - **Client 级别角色**：仅限于特定客户端使用。
- **特性**：用户可以同时拥有 Realm 角色和不同客户端的 Client 级别角色。

#### Group（组）
- **定义**：组是用户集合，用于管理一组具有相同属性或角色的用户。
- **功能**：
   - 为组定义属性，组成员会继承这些属性。
   - 可以将角色映射到组，组成员会自动获得该角色。
- **用途**：通过组实现对用户的批量管理，提高管理效率。

#### Users（用户）
- **定义**：用户是能够登录系统的实体。
- **属性**：用户可以拥有与自己相关的属性，如电子邮件、用户名、地址、电话和出生日期等。
- **功能**：
   - 用户可以加入一个或多个组。
   - 用户可以分配特定的角色，获得访问权限。

#### Client Scope（客户端作用域）
- **定义**：Client Scope 定义了协议映射关系，是 Keycloak 用于简化客户端配置的重要概念。
- **类型**：
   - **Default Scope**：默认生效的作用域。
   - **Optional Scope**：需要客户端显式指定才会生效。
- **功能**：
   - 在 Realm 级别定义共享的 Scope 配置，简化多个客户端的配置工作。
   - 支持 OAuth 2 的 Scope 参数，允许客户端在 Access Token 中获取相关信息。

#### Credentials（凭证）
- **定义**：凭证是用来验证用户身份的数据片段。
- **类型**：
   - 密码
   - 一次性密码（OTP）
   - 数字证书
- **功能**：凭证是用户登录或访问资源时的核心认证依据。

#### Resource Server（资源服务器）
- **定义**：资源服务器是保护资源的应用或服务。
- **功能**：
   - 通过安全令牌决定是否授予对受保护资源的访问权限。
   - 对于 RESTful 服务，从安全令牌中解析用户信息。
   - 对于基于会话的 Web 应用，从用户会话中检索认证信息。
- **特点**：资源服务器依赖 Keycloak 的令牌和授权服务进行访问控制。

### 认证和授权的比较

| 认证                                      | 授权                          |
|-----------------------------------------|-----------------------------|
| 验证确认身份以授予对系统的访问权限                       | 授权确定你是否有权访问资源               |
| 验证用户凭据以获得用户访问权限的过程                      | 验证是否允许访问的过程                 |
| 决定用户是否是他声称的用户                           | 确定用户可以访问和不访问的内容             |
| 身份验证通常需要用户名和密码                          | 授权所需的身份验证因素可能有所不同，具体取决于安全级别 |
| 身份验证是授权的第一步，因此始终是第一步                    | 授权在成功验证后完成                  |
| 特定大学的学生在访问大学官方网站的学生链接之前需要进行身份验证。这称为身份验证 | 授权确定成功验证后学生有权在大学网站上访问哪些信息   |

## 安装部署

如果需要微信登录需要准备以下三个文件
- keycloak-services-social-weixin-0.1.1.jar
- realm-identity-provider-weixin.html
- realm-identity-provider-weixin-ext.html
相关项目地址：
- [GitHub:keycloak-services-social-weixin](https://github.com/halobug/keycloak-services-social-weixin)
- [io.github.jeff-tian/keycloak-services-social-weixin@0.5.40](https://central.sonatype.com/artifact/io.github.jeff-tian/keycloak-services-social-weixin)

docker-compose配置文件 `keycloak.yaml`
```shell
services:
  keycloak:
    image: quay.io/keycloak/keycloak:26.0.5
    container_name: keycloak
    hostname: keycloak
    restart: always
    volumes:
      - ./keywind/theme/keywind:/opt/keycloak/themes/keywind
      - ./weixin/keycloak-services-social-weixin-0.1.1.jar:/opt/keycloak/providers/keycloak-services-social-weixin-0.1.1.jar
      - ./weixin/realm-identity-provider-weixin.html:/opt/keycloak/themes/base/admin/resources/partials/realm-identity-provider-weixin.html
      - ./weixin/realm-identity-provider-weixin-ext.html:/opt/keycloak/themes/base/admin/resources/partials/realm-identity-provider-weixin-ext.html
    ports:
      - 8080:8080
    environment:
      - KC_BOOTSTRAP_ADMIN_USERNAME=admin
      - KC_BOOTSTRAP_ADMIN_PASSWORD=admin
      - PROXY_ADDRESS_FORWARDING=true
      - KC_FEATURES=docker,admin-fine-grained-authz,token-exchange,scripts
      - KC_PROXY=edge
      - KC_HTTP_RELATIVE_PATH=/auth
      - KC_HOSTNAME_STRICT=false
    command: [
      "--spi-login-protocol-openid-connect-legacy-logout-redirect-uri=true",
      "start-dev"
    ]
    entrypoint: [ "/opt/keycloak/bin/kc.sh" ]
```

启动keycloak容器
```shell
docker-compose -f keycloak.yaml up -d
```

## 配置Realm

第一次配置Keycloak是新建并设置Realm，设置好后可以导出Realm的json文件，之后配置新的keycloak时就可以直接通过导入json直接设置realm，无需手动设置。

### 创建并配置Realm

Keycloak默认的Realm是Master，在使用时需要新建Realm，点击左边栏的`Create Realm`，填写Realm名称，可选择是否需要导入已有realm的配置文件。

![](.Keycloak简明指南_images/create_realm.png)

创建后单击Realm Setting选项配置realm属性，需要修改的配置包括：
1. General
   - 可配置当前realm的显示名称
   - User-managed access设置为ON，允许用户使用账户管理控制台管理他们的资源和权限
   - 开发/测试环境，没有配置HTTPS服务，且需要外网访问时，需要将 Require SSL 的值设为 None
2. Login
   - 配置是否允许用户注册等相关内容
   - 推荐 Remember me 设置为 On
3. Email
   - 可开启SMTP服务，配置Keycloak发送邮件的邮箱。
4. Theme
   - 配置所选的皮肤主题以及使用语言。
5. Tokens
   - 配置Token和session的存活时间
   - 建议：Access Token Lifespan --> 15 Minutes

### 创建Client

#### 创建前台应用客户端
点击左侧导航栏Clients，进入客户端列表，点击Create client按钮
![](.Keycloak简明指南_images/clients-list.png)

`Client Type`选择`OpenID Connect`，设置Client ID，点击`Next`
![](.Keycloak简明指南_images/create-client.png)

`Client authentication` 设置为 `Off`，`Authentication flow` 中选中 `Standard flow` 和 `Direct access grants`，点击`Next`
![](.Keycloak简明指南_images/create-client-2.png)

##### 设置客户端安全性行为
![](.Keycloak简明指南_images/create-client-3.png)

- **Root URL**
    - 定义：客户端应用程序的基本 URL，用于构建其他 URL（如回调和重定向 URL）。
    - 作用：Keycloak 根据 Root URL 自动生成默认的重定向和登出 URL（如 Valid redirect URIs 和 Valid post logout redirect URIs）。
    - 示例：
        - 如果设置为 `https://example.com`，Keycloak 会基于此生成其他 URL，例如 `https://example.com/callback`。

- **Home URL**
    - 定义：客户端的默认主页地址。
    - 作用：完成登录或注销操作后，用户可能被重定向到此地址作为默认主页。
    - 示例：
        - 设置为 `https://example.com/home` 后，用户登录或注销后将默认重定向到该地址。

- **Valid Redirect URIs**
    - 定义：允许用户被重定向到的合法 URL 列表。
    - 作用：防止重定向攻击，仅允许列出的地址进行重定向。
    - 支持通配符：
        - 例如：`https://example.com/*` 表示所有以 `https://example.com/` 开头的路径都被允许。
    - 示例：
        - `https://example.com/callback`
        - `https://example.com/*`
    - 注意：只配置可信的 URL，以避免安全风险。

- **Valid Post Logout Redirect URIs**
    - 定义：用户注销后可以被重定向的合法 URL 列表。
    - 作用：Keycloak 完成登出流程后将用户重定向到这些地址之一。
    - 支持通配符：
        - 例如：`https://example.com/logout/*` 表示所有以 `https://example.com/logout/` 开头的路径都被允许。
    - 示例：
        - `https://example.com/logout-success`
        - `https://example.com/logout/*`

- **Web Origins**
    - 定义：允许的来源（origin），即客户端可以从中发起跨源资源共享（CORS）请求的合法域名。
    - 作用：控制浏览器在跨域情况下（如 JavaScript 调用 Keycloak API）允许哪些来源访问资源。
    - 默认值：未设置时，Keycloak 默认允许所有有效重定向 URL 的来源。
    - 示例：
        - `https://example.com`
        - `https://sub.example.com`
        - `*`（允许所有来源，但不推荐，因为存在安全风险）

> 配置示例
> 如果应用的 URL 结构如下：
> - **主页**：`https://example.com/home`
> - **登录回调**：`https://example.com/callback`
> - **注销成功页**：`https://example.com/logout/success`
>> 则配置为：
>> - **Root URL**：`https://example.com`
>> - **Home URL**：`https://example.com/home`
>> - **Valid redirect URIs**：`https://example.com/callback, https://example.com/*`
>> - **Valid post logout redirect URIs**：`https://example.com/logout/success, https://example.com/logout/*`
>> - **Web origins**：`https://example.com, https://sub.example.com`

配置完成后，点击`Save`保存设置。

#### 创建后台应用客户端
1. 点击左侧导航栏Clients，点击Create client按钮
2. Client Type 选择 OpenID Connect，设置Client ID，点击 `Next`
3. `Client authentication` 设置为 **On**，`Authorization` 设置为 **On**（开启授权服务，使Keycloak作为资源服务器管理用户受保护资源，Service accounts role会默认选中，即允许通过 Client credentials 换取 Client Access oken），Authentication flow 中选中 Standard flow 和 Direct access grants，点击 `Next`
4. Valid redirect URIs设置为`*`，点击`Save`

### 创建Role

Role分为两种，一种是Realm Roles，它的作用范围是整个域；另一种是Client Roles，它作用于当前域内的某个特定的Client，需要在Client选项栏里创建。参考：[How are Keycloak roles managed?](https://stackoverflow.com/questions/47837613/how-are-keycloak-roles-managed)

#### Realm Role创建
在UI管理页面点击左边栏的Realm Roles选项，并点击Create Role。输入Role名称，并点击`Save`
![](.Keycloak简明指南_images/create-role.png)

设置当前Role的属性，拥有这个Role的用户将会拥有此属性。
![](.Keycloak简明指南_images/create-role-2.png)

#### Client Role创建
点击左边栏的Clients，选择需要使用的Client，并点击Roles一栏。
![](.Keycloak简明指南_images/create-client-role.png)
点击Create role，输入名称，并设置Role属性等

### User和Group
#### 创建User
点击左边栏 Users → Add user
![](.Keycloak简明指南_images/add-user-1.png)

输入用户的基本信息，点击`Create`按钮
![](.Keycloak简明指南_images/add-user-2.png)

在Credentials一栏可配置用户的登录密码
![](.Keycloak简明指南_images/add-user-3.png)

在Role mapping栏可以设置用户的角色
![](.Keycloak简明指南_images/add-user-4.png)

浏览器访问`http://<host>:<port>/realms/<realm>/account`验证创建的用户是否可以正常登录

#### 创建Group

点击左边栏Groups选项，点击Create group，并输入名称
![](.Keycloak简明指南_images/add-group-1.png)

在Attributes一栏可以设置此Group内部成员共有的属性
![](.Keycloak简明指南_images/add-group-2.png)

在Role Mappings一栏可以设置此Group成员共同属于的Role或Client Role
![](.Keycloak简明指南_images/add-group-3.png)

#### 添加User到Group
点击左边栏Users，并选择需要操作的用户，选择Groups选项卡，选择需要加入的Group
![](.Keycloak简明指南_images/add-user-to-group.png)

#### 给User添加Role
- 用户配置页面选择Role Mappings一栏，点击Assign role ，点击要分配给用户的角色
- 默认添加Client role，添加Realm role需要点击Filter by realm roles
![](.Keycloak简明指南_images/add-user-role.png)

#### 给新用户添加默认的Role
- 点击左边栏的Realm Setting，选择User registration一栏，在这里你可以选择给用户添加默认的Realm Role和Client Role。
- 注意：
   - 用户是属于Realm的管理下，一个Realm对应一组User，而非是一个Client对应一组User，所以不存在给某个Client下面的用户添加默认的Client Role。
   - 用户添加的Client Role表示一个用户在这个Client下默认拥有的角色。
![](.Keycloak简明指南_images/add-user-default-role.png)

### 配置用户属性

#### 用户配置文件

在 Keycloak 中，用户与一组属性相关联。这些属性用于更好地描述和识别 Keycloak 中的用户，以及将有关用户的附加信息传递给应用程序。用户配置文件使管理员能够：
- 为用户属性定义模式
- 定义查看和编辑用户属性的特定权限，在第三方（包括管理员）无法查看或更改某些属性的情况下，可以遵守严格的隐私要求
- 动态强制执行用户配置文件合规性，以便始终更新用户信息并符合与属性关联的元数据和规则
- 通过利用内置验证器或编写自定义验证器，在每个属性的基础上定义验证规则
- 根据属性定义动态呈现用户交互的表单，例如在帐户控制台中的注册、更新配置文件、代理和个人信息，而无需手动更改主题

从管理的角度来看，用户详细信息页面上的“属性”选项卡将仅显示用户配置文件配置中定义的属性。

##### 管理用户配置文件
管理控制台的 Realm settings，选择User Profile选项卡，进入如下配置页面
![](.Keycloak简明指南_images/user-profile-1.png)

点击Create attribute创新新的用户配置文件关联属性
![](.Keycloak简明指南_images/user-profile-attr-1.png)
![](.Keycloak简明指南_images/user-profile-attr-2.png)

属性的详细配置：
- Name：属性的名称
- Display Name：属性的用户友好名称，面向用户的表单呈现
- Multivalued：多值属性
- Attribute Group：属性组
- Enabled when scopes are requested：根据定义的scope动态启用属性，默认设置是属性始终处于启用状态
- Required field：如启用则该属性必须由用户和管理员提供，同时可设置属性仅供用户或者管理员使用
- Permission：定义该属性是否可以被用户或者管理员修改或查看
- Validation：定义验证器，验证属性值是否符合预定规则
   - keycloak官方提供了一些开箱即用的验证器，如设置正则匹配等
- Annotation：将注释关联到属性，主要用于将额外的元数据传到前端进行渲染
   - 如在input输入框给与用户提示信息

更多参考：[Server Administration Guide](https://www.keycloak.org/docs/latest/server_admin/#defining-a-user-profile)

##### 主题中用户属性相关的模板
主题模板中对应的文件说明
- login/register.ftl: 注册页
- login/login-update-profile.ftl：更新用户配置，通过第三方登录用户查看/更新用户配置

例子1：注册页面添加phone字段
```html
<!-- login/register.ftl -->
...
<div class="${properties.kcFormGroupClass!} ${messagesPerField.printIfExists('phone',properties.kcFormGroupErrorClass!)}">
    <div class="${properties.kcLabelWrapperClass!}">
        <label for="user.attributes.phone" class="${properties.kcLabelClass!}">${msg("phone")}</label>
    </div>
    <div class="${properties.kcInputWrapperClass!}">
        <input type="text"
               id="user.attributes.phone"
               class="${properties.kcInputClass!}"
               name="user.attributes.phone"
               value="${(register.formData['user.attributes.phone']!'')}"
        />
    </div>
</div>

...
```
用户填写信息后会自动增加phone字段，填写完毕注册后就会在这个用户下增加phone属性。

例子2：第三方登录用户增加自定义字段
```html
<!-- login/login-update-profile.ftl -->
...
<div class="${properties.kcFormGroupClass!} ${messagesPerField.printIfExists('phone',properties.kcFormGroupErrorClass!)}">
    <div class="${properties.kcLabelWrapperClass!}">
        <label for="user.attributes.phone" class="${properties.kcLabelClass!}">${msg("phone")}</label>
    </div>
    <div class="${properties.kcInputWrapperClass!}">
        <input type="text" 
               id="user.attributes.phone" 
               name="user.attributes.phone" 
               value="${(user.attributes.phone!'')}" 
               class="${properties.kcInputClass!}" />
    </div>
</div>

...
```
此外，还在登录页面增加隐藏控件设置需要进行初始化的属性，这样也就不需要通过group的方式来添加用户的属性了。同样在register.ftl下增加上面的代码，设置div的style="display:none"并将input的value设置为默认的值即可。更多参考：[How to add a custom field to the Keycloak registration page](https://keycloakthemes.com/blog/how-to-add-custom-field-keycloak-registration-page)

#### 在Token中增加用户信息
用户的属性或Role等信息在Keycloak中保存，通过access token获取userinfo默认不带有这些信息，可以通过设置客户端作用域的方式增加token中携带的用户信息。

选择一个client，在Client Scopes选项卡中，找到名为 ${client-name}-dedicated 的Scope，进入详情配置界面
![](.Keycloak简明指南_images/client-dedicated-scope-1.png)

添加Mapper，选择`From Predefined Mapper`，选择`realm roles`，进入详细配置界面
![](.Keycloak简明指南_images/client-dedicated-scope-2.png)

如果添加自定义用户属性，此步骤选择`By configuration`，然后选择`User Attribute`
![](.Keycloak简明指南_images/client-dedicated-scope-2.1.png)

将`Add to access token`、`Add to ID token`和`Add to userinfo`设为`ON`
![](.Keycloak简明指南_images/client-dedicated-scope-3.png)

注意：如果在前端/后台程序中设置了从Access/Id Token中读取额外信息的逻辑，而Keycloak中未进行相关设置，可能会导致多种问题：
- 连接无法创建成功，反复重试
- 无法验证登录
- 遇到以上情况，可以从客户端作用域设置处开始排查

### 配置GitHub登录

#### User Profile中添加avatar_url字段
- Attribute: avatar_url
- Display name: ${avatar_url}
- Permission: 全部选中

##### 注册Github OAuth应用
1. 点击帐户设置
2. 点击左边栏最下方的Developer settings
3. 在左侧边栏中点击“OAuth Apps”，并点击New OAuth App
4. 选择输入如下参数
   - Application name: OAuth 应用名称
   - Homepage URL：Keycloak的网址 https://auth.dvclab.com
   - Authorization callback URL：OAuth登录回调地址，https://<host>:<port>/realms/<realm>/broker/github/endpoint
5. 创建完毕后进入OAuth App详情页，点击Generate a new client secret生成一个新的client secret，记录当前的Client ID和Client secrets两个参数，它们会在Keycloak中进行配置
6. 参考：[注册 GitHub 应用](https://docs.github.com/zh/apps/creating-github-apps/registering-a-github-app/registering-a-github-app)

#### 配置Keycloak Identity Providers
在左边栏中点击Identity Providers，点击Github
![](.Keycloak简明指南_images/github-idp-1.png)

填写上一步生成的Client ID和Client Secret参数，点击Add添加Github Provider
![](.Keycloak简明指南_images/github-idp-2.png)

点击Mappers，选择Add Mapper，进行第三方用户信息映射
![](.Keycloak简明指南_images/github-idp-3.png)

配置如下参数，将Github用户的头图信息映射到Keycloak的用户属性当中
- Name：github-avatar
- Sync mode override：第三方用户信息导入方式，选择import
- Mapper type：将信息映射到哪里，选择Attribute Importer
- Social Profile JSON Field Path ：第三方用户信息字段选择，填写avatar_url
- User Attribute Name ：Keycloak属性字段，填写avatar_url
点击Save


### Realm 导出/导入
导出
```shell
docker exec -it keycloak /bin/bash -c "/opt/keycloak/bin/kc.sh export --dir /tmp/export --realm ${REALM}"

docker cp keycloak:/tmp/export .
```

导入
```shell
docker cp ./export keycloak:/tmp/import

docker exec -it keycloak /bin/bash -c "/opt/keycloak/bin/kc.sh import  --dir /tmp/import"

docker restart keycloak
```
注：
- 通过Keycloak UI中新建Realm时，也可直接选择配置文件进行导入

参考：
- [keycloak/keycloak-documentation: Export and Import](https://github.com/keycloak/keycloak-documentation/blob/master/server_admin/topics/export-import.adoc)
- [Keycloak Realm Export/Import](https://www.plexx.digital/keycloak-realm-export-import/)
- [Export and Import](https://www.keycloak.org/docs/latest/server_admin/#_export_import)
- [how to get keycloak to export realm users and then exit](https://stackoverflow.com/questions/60766292/how-to-get-keycloak-to-export-realm-users-and-then-exit)


### Admin REST API
修改用户属性
```shell
curl -X PUT https://${host}:${port}/admin/realms/${realm}/users/${uid} \
--header 'Content-Type: application/json' \
--header "Accept: application/json" \
--header "Authorization: Bearer ${token}" \
-d '{
    "attributes": {
        "avatar_url": [
            ${new_value}
        ]
    }
}'
```

禁用用户
```shell
curl -X PUT https://${host}:${port}/auth/admin/realms/${realm}/users/${uid} \
--header 'Content-Type: application/json' \
--header "Accept: application/json" \
--header "Authorization: Bearer ${token}" \
-d '{"enabled":false}'
```

### 配置HTTPS访问

#### HTTPS与HTTP协议的区别
- HTTP是超文本传输协议，信息是明文传输，HTTPS则是具有安全性的SSL加密传输协议。
- HTTP和HTTPS使用的是完全不同的连接方式，使用的端口也不一样,前者是80,后者是443。
- HTTPS协议需要到证书颁发机构(Certificate Authority，简称CA)申请证书，
HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比HTTP协议安全。
- HTTPS和HTTP协议相比提供了
   - 数据完整性：内容传输经过完整性校验 
   - 数据隐私性：内容经过对称加密，每个连接生成一个唯一的加密密钥 
   - 身份认证：第三方无法伪造服务端(客户端)身份

#### Keycloak的HTTPS配置说明
1. keycloak默认不会去处理SSL/HTTPS请求，应在反向代理服务器上开启SSL。
2. keycloak中的HTTPS有三种模式；分别是external requests、none、all requests。默认为external requests模式，在这个模式下keycloak的私有IP地址才能不使用SSL方式登录。私有IP如：127.0.0.1，192.168.x.x。 当通过公网访问时就需要SSL(HTTPS)的方式进行登录。
3. 可以通过反向代理的方式来启动HTTPS/SSL

使用Nginx做keycloak代理，方便统一设置HTTPS访问
- 在容器部署环境，后端服务和前端无法通过统一地址访问Keycloak，导致token认证失败，通过设置统一的逆向代理可以解决这个问题，详见：
   - [keycloak token introspection always fails with {"active":false}](https://stackoverflow.com/questions/53721588/keycloak-token-introspection-always-fails-with-activefalse)
   - 在典型的2客户端场景中，后端服务通过keycloak的容器名称访问，前端则运行在用户侧，只能通过宿主机IP或DNS域名访问，此时一个变通方法是配置Realm的 Frontend URL
      - http://{宿主机IP/域名}:{服务端口}/auth
- 在keycloak容器中添加 PROXY_ADDRESS_FORWARDING=true环境变量可以解决keycloak.js mix content的问题，参考：
   - [keycloak in docker behind proxy](https://keycloak.discourse.group/t/keycloak-in-docker-behind-reverse-proxy/1195)
- 其他参考：
   - [keycloak官方文档Setting up HTTPS/SSL](https://www.keycloak.org/docs/latest/server_installation/index.html#setting-up-https-ssl)
   - [keycloak ssl-required报错问题处理](https://blog.csdn.net/dblrxy417894/article/details/101590062)
   - [docker镜像中的Setting up TLS(SSL)](https://hub.docker.com/r/jboss/keycloak/)

#### nginx配置
```nginx configuration
server {
    listen 80;
    server_name {domain/ip};
    rewrite ^(.*)$  https://{domain/ip}$1 permanent;
}

server {
    listen          443 ssl;
    server_name     {domain/ip};
    
    # ssl证书key位置
    ssl_certificate      /etc/nginx/ssl/auth-cert.pem;
    ssl_certificate_key  /etc/nginx/ssl/auth-cert.key;
    ssl_session_timeout  10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers  on;

    location / {
         proxy_set_header  Host  $host;         
         proxy_set_header  X-Forwarded-Proto $scheme;
         proxy_set_header  X-Forwarded-For $host;
         proxy_set_header  Upgrade $http_upgrade;
         proxy_set_header  Connection 'upgrade';
         proxy_set_header  X-Real-IP $remote_addr;

         proxy_pass    http://{keycloak_host}:{keycloak_port}/;
    }
}

```

### 其他

#### 设置用户为Realm管理员
1. 选择用户，转到Role Mappings选项卡
2. 选择 Filter by realm roles，选择Admin，选择Assign

#### 跨Domain验证
- 前端使用服务部署后的外部IP访问Keycloak，获取Token；
- 后端所在容器和Keycloak所在容器在一个主机，只能通过 host.internal.local 或docker-compose中的内部域名方式访问Keycloak；
- 此时需要在realm中增加 `Frontend URL` 设置，如：http://*:8080/
- 这样后端就可以正常验证前端发过来的Token了，否则只会得到 {"active":false}
![](.Keycloak简明指南_images/realm-frontend-url.png)

## Endpoints 端点

### OpenID配置端点 OpenID Configuration Endpoint

1. `http://<server>/realms/<realm>/.well-known/openid-configuration`
    1. OpenID Configuration Endpoint就像是根目录，它可以返回所有其他可用的Keycloak端点，支持的范围和声明以及签名算法。

### 授权端点 Authorize Endpoint

1. `http://<server>/realms/<realm>/protocol/openid-connect/auth`
    1. 接受scope和redirect\_url作为可选参数

### 令牌端点 Token Endpoint

1. `http://<server>/realms/<realm>/protocol/openid-connect/token`
    1. 令牌端点允许我们获取访问令牌，刷新令牌或ID令牌。OAuth 2.0支持不同的授予类型，例如authorization\_code，refresh\_token或密码。
    2. 每种授权类型都需要一些专用的表单参数。
    3. 通过authorization\_code获取access\_token，必须在请求正文中传递以下表单参数：client\_id，client\_secret，grant\_type，code和redirect\_uri。
    4. 如果要绕过授权码流程，则选择密码授予类型。这里需要用户凭据，因此当网站或应用程序上具有内置登录页面时，可以使用此流程。

### 用户信息端点 User Information Endpoint

1. `http://<server>/realms/<realm>/protocol/openid-connect/userinfo`
    1. 当具有有效的access\_token时，可以从用户信息端点检索用户配置文件数据。

### Token验证端点 Token Introspect Endpoint

1. `https://<server>/realms/<realm>/protocol/openid-connect/token/introspect`
    1. 如果资源服务器需要验证access\_token是否处于active状态或需要更多有关它的数据，尤其是对于不透明的访问令牌，则需要访问Token Introspect Endpoint（token自检端点）。在这种情况下，资源服务器使用安全的配置方法整合了自检过程。
    2. 如果令牌有效则返回token状态信息，若无效则返回token的active为false。

### 参考

1. [Keycloak获取Token示例及遇到的坑](https://blog.csdn.net/u012760435/article/details/82259493)

## 令牌 Token

### Token介绍

#### JSON Web Token

参考：

1. [JWT token generation](https://cloudnativereference.dev/related-repositories/keycloak/)

#### ID Token 身份令牌

ID Token 的格式为 JWT。ID Token 仅适用于认证场景。

1. OIDC (OpenID Connect) 协议对 OAuth 2.0 协议 最主要的一个扩展就是 ID Token 数据结构。ID Token 相当于用户的身份凭证，开发者的前端访问后端接口时可以携带 ID Token，开发者服务器可以校验用户的 ID Token 以确定用户身份。
   - 例如，有一个应用使用了谷歌登录，然后同步用户的日历信息，谷歌会返回 ID Token 给这个应用，ID Token 中包含用户的基本信息（用户名、头像等）。应用可以解析 ID Token 然后利用其中的信息，展示用户名和头像。
   - 在使用 ID Token 之前应该先验证合法性。不推荐使用 ID Token 来进行 API 的访问鉴权。
2. 在S3的AssumeRoleWithWebIdentity中使用的就是ID Token进行验证
   - ID Token的aud 参数是发起认证授权请求的应用的 ID（或编程访问账号的 AK）。
3. 获取ID Token的方法是在scope参数中加入openid

Keycloak中获取id token的api如下：
```shell
curl -X POST \
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=password" \ 
  --data "client_id={client_id}" \  
  --data "client_secret={client_secret}" \  
  --data "username={username}" \ 
  --data "password={password}" \ 
  --data "scope=openid"
```

#### Access Token 访问令牌

Access Toke 是一个用来访问受保护资源的凭证，它是由授权服务器(Authorization Server)颁发给客户端(Client)的，通常是字符串形式，access token 拥有特定的访问范围(scope)，并且有时间限制，访问令牌可以有不同的格式和结构。

* 绝对不要使用 Access Token 做认证。Access Token 本身不能标识用户是否已经认证。
* Access Token 中只包含了用户 id，在 sub 字段。在应用开发中，应该将 Access Token 视为一个随机字符串，不要试图从中解析信息。

Access Token 内容示例：
```shell
{
  "jti": "YEeiX17iDgNwHGmAapjSQ",   
  "sub": "601ad46d0a3d171f611164ce", // subject 的缩写，为用户 ID   
  "iat": 1612415013,   
  "exp": 1613624613,   
  "scope": "openid profile offline\_access",   
  "iss": "https://yelexin-test1.authing.cn/oidc",   
  "aud": "601ad382d02a2ba94cf996c4" // audience 的缩写，为应用 ID 
}
```

Keycloak中获取access token的api如下：
```shell
curl -X POST \ 
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=password" \ 
  --data "client_id={client_id}" \  
  --data "client_secret={client_secret}" \  
  --data "username={username}" \ 
  --data "password={password}"
```

参考：

1. [https://auth0.com/docs/secure/tokens/access-tokens](https://auth0.com/docs/secure/tokens/access-tokens)
2. [Access Token vs ID Token](https://docs.authing.cn/v2/concepts/access-token-vs-id-token.html)

#### Refresh Token 刷新令牌

Refresh Token 是一种常用于 OAuth 2.0 授权框架中的令牌，用于延长访问令牌（Access Token）的生命周期。它的主要作用是在访问令牌过期后，允许客户端通过刷新令牌获取一个新的访问令牌，而不需要用户再次进行身份验证或授权。

Refresh Token 的特点：

1. 生命周期较长：
   - 与访问令牌相比，刷新令牌的有效期通常更长，甚至可以是无限期，具体取决于服务提供商的设计。
2. 安全性：
   - 刷新令牌的使用频率低，通常只在访问令牌过期时才用到，因此它暴露在外的风险较小。为了进一步增强安全性，可以将刷新令牌与 IP 地址或设备绑定。
3. 续期机制：
   - 客户端使用刷新令牌与授权服务器交互时，不需要重新要求用户输入用户名和密码或重新授权。

Refresh Token 的使用流程：

1. 获取 Refresh Token：
   - 当用户完成身份验证并授权后，授权服务器返回一个访问令牌和一个刷新令牌。
2. 使用 Access Token：
    - 客户端通过访问令牌访问受保护的资源。
3. Access Token 过期：
    - 当访问令牌过期后，客户端无法再使用它访问资源。
4. 刷新令牌获取新 Access Token：
    - 客户端向授权服务器发送刷新令牌，服务器验证后返回一个新的访问令牌，可能会返回新的刷新令牌（视具体实现而定）。

Keycloak中刷新access token的api如下：
```shell
curl -X POST \ 
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=refresh_token" \ 
  --data "client_id={client_id}" \
  --data "client_secret={client_secret}" \
  --data "refresh_token={refresh_token}"
```

### Offline Token 离线令牌

1. offline access是OpenID Connect 规范中描述的一项功能。它核心概念是在登录过程中，客户端应用程序将请求offline token而不是传统的refresh token。应用程序可以将此offline token保存在数据库或磁盘上，即使用户已注销，也可以稍后使用它。可以使用offline token在用户已经离线的情况下获取用户的access token，代表用户执行一些离线操作。
2. refresh token和offline token的区别是，offline token在默认情况下永远不会到期，并且不受SSO Session Idle timeout和SSO Session Max lifespan两个参数的影响。即使在用户注销或服务器重新启动后，离线令牌仍然有效。默认情况下，需要至少每30天使用一次offline token进行刷新令牌操作（Offline Session Idle timeout可以在管理控制台中Realm Setting的Tokens选项卡下更改此值）。此外，如果启用该选项Offline Session Max Limited，使用离线令牌进行刷新令牌操作，离线令牌也会在 60 天后过期（Offline Session Max也可以在管理控制台的Realm Setting下的Token选项卡中更改此值）。
3. 此外，如果启用该选项Revoke refresh tokens，那么每个离线令牌只能使用一次。因此，刷新后，始终需要将刷新响应中的新offline token存储到数据库中。

获取Offline token api如下：
```shell
curl -X POST \ 
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=password" \ 
  --data "client_id={client_id}" \ 
  --data "client_secret={client_secret}" \
  --data "username={username}" \ 
  --data "password={password}" \ 
  --data "scope=offline_access"
```

参考：[offline access](https://www.keycloak.org/docs/12.0/server_admin/index.html#_offline-access)

## Client Type与Token关系

1. 在介绍Keycloak前需要先了解Oauth2.0下定义的两种Client 类型；OAuth 2.0 核心规范定义了两种客户端类型, confidential 机密的 和 public 公开的，区分这两种类型的方法是， 判断这个客户端是否有能力维护自己的机密性凭据(client\_secret)。
   - **confidential**：对于一个普通的web站点来说，虽然用户可以访问到前端页面，但是数据都来自服务器的后端api服务， 前端只是获取授权码code， 通过 code 换取access\_token 这一步是在后端的api完成的，由于是内部的服务器， 客户端有能力维护密码或者密钥信息， 这种是机密的的客户端。
   - **public**：对于一个没有后端的纯前端应用来说，数据的展示和操作都是在前端完成的，包括获取令牌和操作令牌， 把一个客户端密钥放在纯前端应用是不安全的, 这种是公开的客户端。
2. 在 OAuth 2.0 授权码模式（Authorization Code）中， 客户端通过授权码code向授权服务器获取访问令牌(access\_token) 时，同时还需要在请求中携带客户端密钥(client\_secret)， 授权服务器对其进行验证， 保证 access\_token 颁发给了合法的客户端。
3.  对于公开的客户端来说， 本身就有密钥泄露的风险， 所以就不能使用常规 OAuth 2.0 的授权码模式， 于是就针对这种不能使用 client\_secret 的场景， 衍生出了 Implicit 隐式模式， 这种模式从一开始就是不安全的。在经过一段时间之后， PKCE 扩展协议推出， 就是为了解决公开客户端的授权安全问题。

参考：[OAuth 2.0 的探险之旅](https://www.cnblogs.com/myshowtime/p/15500050.html)

### 不同Client间Access Token的转化

在Keycloak的使用场景中，用户在前端登录使用的是Oauth的Implicit隐式模式，对应的Client Type是public类型。当用户在前端页面上请求后端服务时需要的是后端client（confidential类型）的token，此时就需要用到不同client间token的互相转化。Keycloak中使用方式如下：

1. 重新启动keycloak，加入-Dkeycloak.profile.feature.token\_exchange=enabled \-Dkeycloak.profile.feature.admin\_fine\_grained\_authz=enabled参数
2. 使用如下api进行将前端token换取为后端token

Keycloak Token请求方式（管理员请求所有token类型）
```shell
curl -X POST \ 
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  --data "grant_type=urn:ietf:params:oauth:grant-type:token-exchange" \ 
  --data "client_id={client_id}" \  
  --data "client_secret={client_secret}" \  
  --data "audience={resource_server_client_id}" \   
  --data "subject_token={access_token}" \  
  --data "requested_token_type=urn:ietf:params:oauth:token-type:access_token"
```

3. 使用前端token换取后端token（设置参数scope=offline\_access获取长期有效的离线refresh\_token）
```shell
curl -X POST \ 
  http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \  
  --data "grant_type=urn:ietf:params:oauth:grant-type:token-exchange" \ 
  --data "client_id={client_id}" \  
  --data "client_secret={client_secret}" \  
  --data "audience={resource_server_client_id}" \   
  --data "subject_token={access_token}" \  
  --data "requested_token_type=urn:ietf:params:oauth:token-type:refresh_token" \ 
  --data "scope=offline_access"
```

## 授权服务
Keycloak的细粒度授权主要有三个必要步骤：
1. 资源管理
   - 定义什么是被保护的对象。
   - 资源服务器可以使用Keycloak管理员控制台来管理，可以将任何已注册的客户端启用为资源服务器，并管理其资源和范围。
   - 资源可以是web页面、Rest资源、文件系统上的一个文件、一个EJB等等。资源既可以是一组资源（如Java中的一个Class），也可以是一个特定的资源。
   ![](.Keycloak简明指南_images/resource-scope.png)
2. 权限及策略管理
   - 策略定义了在资源或者范围上执行某些动作的先决条件，但是它们并不和被保护的资源直接绑定。
   - 策略是通用的，可以通过再次组合来构造更复杂的权限及策略。 
   - 权限与被保护的资源紧耦合，它们由被保护对象及策略组合而成。
3.	执行策略
   - 策略的执行包含了对资源服务器实际实施授权决策的必要步骤。实现方式是在资源服务器上启用策略实施点（PEP），PEP能够与授权服务器通信，请求授权数据，并根据服务器返回的决策来控制对受保护资源的访问。

### 资源管理
通过创建资源服务器，可以创建要保护的资源和作用域（scope）。可以从客户端的Resource和Scope选项卡来管理它们。

#### 资源服务器 Resource Server
任何可信的Keycloak客户端都可以作为资源服务器。这些客户端的资源及范围由一系列的授权策略保护。
- 启用授权服务需要首先在客户端Client Setting中将Authorization Enabled打开。
- 授权服务启动后，相关配置在 Client → Authorization 一栏下，如下图：
![](.Keycloak简明指南_images/resource-server-auth.png)

#### 查看资源
在Resouce页面，可以查看当前资源服务器上的资源列表。资源是应用或者组织的资产。它们可以是一些列的端点、一个典型的HTML页面等等。在授权策略语境中，资源指的就是被保护的对象。
![](.Keycloak简明指南_images/client-resources.png)
1. Keycloak中，Resource定义了一组通用的信息：
   - Name：可读的、全局唯一的字符串
   - Type：唯一标识一组资源类型的字符串，用来对不同资源实例分组。因此可以使用一组通用的权限来保护它们
   - URIs：代表资源地址，对于HTTP资源来说，URIS通常是资源的相对路径
   - Authorization Scopes：授权作用域，一个或多个
      - **通常表示对指定资源可施加的动作**，如查看、编辑、删除等等。
      - 资源的范围扩展了资源的访问界限。
      - 在授权语义中，范围是可以应用在资源上的许多动词之一。
      - 参考：[Scopes explained](https://www.keycloak.org/docs/latest/server_admin/#scopes-explained)
   - Resource attributes：资源可能具有与其关联的属性。 这些属性可用于提供有关资源的附加信息，并在评估与资源关联的权限时为策略提供附加信息。每个属性都是一个键值对，其中值可以是一组一个或多个字符串。 可以通过用逗号分隔每个值来为属性定义多个值。
2. 可以从列表中选择资源点击Create Permission来为其创建权限
    - 创建权限前请确保已定义好需要关联的策略
3. 资源管理API：[Managing resources](https://www.keycloak.org/docs/latest/authorization_services/index.html#_service_protection_resources_api)

#### 创建资源
主要需要考虑的是资源的粒度。Keycloak的Resource概念可以用来表示一组（一个或多个）资源，需要考虑如何进行资源的定义。
1. 点击资源列表上方的`Create`按钮
![](.Keycloak简明指南_images/client-resources-1.png)

### 授权策略

1. 策略定义了授予对象访问权时必须**满足的条件**。
2. 与权限不同，策略不指定受保护的对象，而是指定访问对象（如资源、范围或两者）时必须满足的条件。
3. 策略与用来保护资源的访问控制机制(ACMs)密切相关。使用策略可以实现基于属性的访问控制(ABAC)、基于角色的访问控制(RBAC)、基于上下文的访问控制以及其任何组合。Keycloak还提供了聚合策略，即“策略的策略”。
4. 在构造复杂的访问控制条件时，Keycloak授权服务遵循了分而治之的原则。你可以创建独立的策略，并在不同的权限中使用它们，然后通过组合它们来构造更复杂的策略。
5. 可以看成是访问某个对象需要满足的条件；它通过permission将这个条件和受保护对象结合起来

Keycloak支持的授权策略类型
![](.Keycloak简明指南_images/policy-1.png)

资源服务器需要依据一些信息才能判断权限。对于REST的资源服务器这些信息通常来自于一个加密的token，如在每个访问请求中携带bearer token。对于依赖于session的Web应用来说，这些信息就来自于每个request对应的用户session。

#### 基于用户的策略 User-based policy

1. 基于用户的策略可以限制只有指定的用户能够访问被保护的资源。
2. 配置信息：
   - Name：名称是一个可读且唯一的字符串。建议取一些和业务相关的名字以易于识别。
   - Description：策略的描述信息
   - Users：指定此策略授予哪些用户访问权限。
   - Logic：在评估其他条件后应用此策略的逻辑。POSITIVE（正向）/NEGATIVE（负向）
      - 策略可以配置为正向或负向。简单地说，可以使用这个选项来定义策略结果是应该保持原样，还是被否定。
      - 举例来说，假设创建了策略，其中只有没有授予特定角色的用户才被授权。此时可以使用该角色创建基于角色的策略，并将其逻辑字段设置为极负向（Negative）。

![](.Keycloak简明指南_images/policy-user.png)

#### 基于角色的策略 Role-based policy

1. 此种策略允许只有指定的Role才能获取权限。
2. 默认情况下，如果发起请求的用户已被授予此策略中任意一个角色就可以获得授权。但是，如果要强制执行特定角色，可以根据需要指定特定角色。您还可以组合必需和非必需角色，无论它们是realm role还是client role。
3. 配置信息：
   - Name
   - Description
   - Roles：可以选择Realm role和Client role
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-role.png)

#### 基于组的策略 Group-based policy

1. 此种策略允许只有你指定的组才能获取访问权限。
2. 配置信息：
   - Name
   - Description
   - Groups Claim：在包含组名和/或路径的令牌中指定声明（即令牌key/value中的key）。通常，授权请求是基于ID Token或Access Token处理的。该策略将根据Token的组声明来获取用户所属的组。如果没有定义，则从域配置中获取。
   - Groups：允许您在评估权限时选择应由该策略强制执行的组。添加组后，可以通过标记复选框扩展至子级来扩展对组子级的访问。如果未标记，则访问限制仅适用于所选组。
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-group.png)

#### 基于客户端的策略 Client-based policy
1. 根据权限定义条件，允许一组一个或多个客户端访问某个对象。
2. 配置信息：
   - Name
   - Description
   - Clients：该策略关联的客户端
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-client.png)

#### 基于客户端作用域的策略 Client-scope policy
1. 基于特定客户端作用域的策略
2. 配置信息：
   - Name
   - Description
   - Client scopes：选择对应的客户端作用域
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-client-scope.png)

#### 基于时间的策略 Time-based policy

1. 此种策略允许你制定时间相关的权限条件。
2. 配置信息：
   - Name
   - Description
   - Repeat：非重复 / 重复
      - Month
      - Day
      - Hour
      - Minute
   - Start time
   - Expire time
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-time.png)

#### 聚合策略 Aggregated policy
1. Keycloak支持构建策略的策略，称之为聚合策略。你可以使用策略聚合，通过重用现有的策略来构建更复杂策略，使得权限与策略更解耦。
2. 配置信息：
   - Name
   - Description
   - Apply Policy：定义一组与聚合策略关联的一个或多个策略。这里可以选择已有的策略，也可以创建新策略。
   - Decision Strategy：此权限的决策逻辑
      - Unanimous：全体一致策略
         - 所有相关策略必须同时评估为“允许”才能授予访问权限。
         - 如果任何一个策略评估为“拒绝”，最终结果就是“拒绝”。
         - 如果所有策略均未明确评估为“允许”，最终结果为“拒绝”。
      - Affirmative：肯定优先策略
         - 只要有一个策略评估为“允许”，就授予访问权限。
         - 优先考虑“允许”，如果至少一个策略评估为“允许”，最终结果为“允许”。
         - 只有在所有策略均评估为“拒绝”或未明确允许时，最终结果为“拒绝”。
      - Consensus：共识策略
         - 基于多数决，策略的“允许”票数多于“拒绝”票数时授予访问权限。
         - 如果“允许”的策略多于“拒绝”的策略，则最终结果为“允许”。
         - 如果“拒绝”的策略多于或等于“允许”的策略，则最终结果为“拒绝”。
         - 未明确评估（如“不适用”）的策略不会影响最终结果。
   - Logic：正向或负向

![](.Keycloak简明指南_images/policy-aggregated.png)

### 权限 Permission

**Permissions权限将上文的策略和作用对象结合起来**

1. 权限将被保护的对象与必须评估的策略关联起来，以确定是否允许访问。
2. 例如 X 可以在资源 Z 上施加 Y；
    1. 这里 X 指可以是用户、角色、用户组，或者其组合。在这里你可以使用声明和上下文。
    2. Y 指代一个动作，如写、读等等。
    3. Z 指代的源，如 "/accounts"
3. Keycloak提供了丰富的平台用于构建从简单到复杂的权限策略，如基于规则的权限控制，使用Keycloak可以带来：
    1. 减少代码重构和权限管理成本；
    2. 支持灵活的安全模型，用以应对安全模型可能的变化
    3. 支持运行时变化。应用系统只需要关心资源和范围，Keycloak隐藏了它们被保护的细节。
4. 权限包含两种，分别是基于资源的权限和基于范围的权限。可在Authorization一栏的Permission出创建、管理权限。

#### 基于资源的权限 Resource-based Permission

1. 基于资源的权限定义了一组资源且使用一组策略来保护这些它们。
2. 可以针对一组同类型的资源统一设定权限。在对有着相同访问限制的资源设置权限时非常有用。
3. 比如说在一个金融类的应用，其中每个银行账户属于一个特定的用户。虽然是不同的银行账户，但是可以共享全局银行机构通用的安全性要求。你可以使用分类的资源权限来定义这些通用策略。
   - 例如：仅有账户拥有者才能管理自己的账户仅能在账户拥有者的国家/地区进行访问执行特定的身份认证方法
4. 配置信息：
   - Name
   - Description
   - Apply To Resource Type：指定权限是否应用于具有给定类型的所有资源。 选择此字段时，系统会提示您输入要保护的资源类型。
   - Resources/Resource Type：定义一组要保护的一个或多个资源/资源类型
   - Policies：定义此权限关联的一组策略。这里可以选择已有的策略或者新建
   - Decision Strategy：此权限的决策策略

![](.Keycloak简明指南_images/permission-resource-based.png)

#### 基于范围的权限 Scoped-based Permission

1. 基于范围的权限定义一组范围，通过关联授权策略来保护这些范围。
2. 与基于资源的权限不同，可以使用此权限类型为为资源关联的范围创建权限，从而提供更细的权限控制粒度。
3. 要创建新的基于范围的权限，请在新建权限时选择Scoped-based。
4. 配置信息：
   - Name
   - Description
   - Apply To Resource Type：指定权限是否应用于具有给定类型的所有资源。 选择此字段时，系统会提示您输入要保护的资源类型。
   - Resource/Resource Type：将范围限制为与所选资源关联的范围。 如果没有选择，则所有范围都可用。
   - Authorization scopes：定义一组要保护的一个或多个scope
   - Policies：定义此权限关联的一组策略。这里可以选择已有的策略或者新建
   - Decision Strategy

![](.Keycloak简明指南_images/permission-scope-based.png)

### 决策策略 Decision Strategy

#### 资源服务器 Resource Server 决策策略

当外部请求（如用户）对某个资源进行访问时，Resource Server会根据决策策略评估此资源所有的Permission是否符合判定要求，以决定外部请示是否有权限访问此资源

1. Affirmative：至少一个绑定该资源权限必须评估为一个积极的决定，以便授予对资源及其范围的访问权限。
2. Unanimous：味着绑定此资源的所有权限必须评估为积极的决定，以便最终决定也是积极的。

例如，如果同一资源或范围的两个权限发生冲突（其中一个是授予访问权限，另一个是拒绝访问），如果选择的策略是Affirmative，则将授予对该资源或范围的权限。 否则，任何权限的单个拒绝也将拒绝对资源或范围的访问。

#### 权限 Permission 决策策略

Permission将一个或多个Policy绑定至Resource，以确定Resource的访问规则

1. Affirmative
2. Unanimous
3. Consensus：意味着积极决策的数量必须大于消极决策的数量。如果正面和负面的数量相同，则最终决定是否定的。

#### 决策策略示例

如果有多个权限匹配给定的资源/范围，最终是否授予对资源/范围的访问权限取决于以下：

1. The Resource Server's decision strategy（Affirmative/Unanimous）
2. 每个Permission's decision strategy（Affirmative/Unanimous/Consensus）
3. 每个Policy's logic value（Positive/Negative）

例如：  
一个Resource设置了两个Permission（permission\_A，permission\_B），permission\_A引用了两个policy（policy\_A\_1，policy\_A\_2），permission\_B引用了两个Policy（policy\_B\_1，policy\_B\_2），policy的Logic的均设置为Positive

1. permission\_A的决策策略设置为Affirmative，假设引用的Policy评估为：
    1. policy\_A\_1：true
    2. policy\_A\_2：false
2. permission\_B的决策策略设置为Unanimous，假设引用的Policy评估为：
    1. policy\_B\_1：true
    2. policy\_B\_2：false

对于permission\_A，由于其决策策略设置为Affirmative，所以只要其引用的Policy中有一个评估为true权限permission\_A被认定为true；对于permission\_B，由于其决策策略设置为Unanimous，只有当其引用的所有Policy均被评估为true，其才会被认定为true。  
最终评估：

1. 当资源服务器的决策策略设置为Affirmative，只要有一个Permission被认定为true，则说明对此Resource具有访问权限
2. 当资源服务器的决策策略设置为Unanimous，只有当所有的Permission被认定为true，才能对此Resource具有访问权限

## UMA资源管理

### 名词解释

1. 资源服务器 Resource Server / RS
   - 存放资源并能够分发permission ticket
   - Resource Server是**策略执行点（PEP）**，所以它知道访问API需要的Scope，且能够根据RPT的验证结果返回对应的资源。
   - 判断访问是否含有token，没有Token则请求Permission Ticket，有Token则验证Token
2. 授权服务器 Authorization Server / AS
   - 发布token，验证policy，保护资源。这里指Keycloak
   - 是策略决策点 Policy Decision Point
3. 客户端 Client / C
   - 试图访问受保护资源的应用
   - Client不知道Scope，通过获得的Permission Ticket提供
4. 资源所有者 Resource Owner / RO
   - 拥有并决定是否接受访问资源的用户
5. 请求第三方 Requesting Party / RP
   - 请求访问资源的用户
6. 权限凭据 Permission Ticket / PT
   - 包含了资源信息和资源动作范围的凭据

### UMA资源授权流程

**资源注册**

1. Resource Owner 向 RS 注册资源（例如文件、健康数据）。
2. RS 将资源的元数据（资源标识符、类型、支持的操作等）发送到 AS。
3. 授权策略配置：
4. Resource Owner 在 AS 定义资源的访问策略（如谁可以访问、何时访问）。

**客户端请求资源**

1. Requester 通过 Client 向 RS 请求访问资源（如读取文件）。
2. 如果 Client 没有有效的 RPT（Requesting Party Token） 或权限不足，RS 拒绝请求并返回一个 Permission Ticket。
   - Permission Ticket是RS 生成的临时票据，标识请求的资源和操作，用于后续向 AS 请求授权。
   - Permission Ticket 是匿名的，不包含请求者的身份信息。

**权限请求阶段**

1. Client 携带 Permission Ticket 和自身的认证凭据（如 Access Token）向 AS 请求权限。
   - 请求中包括需要访问的资源标识符和操作权限（如 read、write）。
2. AS 验证 Permission Ticket 和 Access Token 是否有效。
   - 根据资源拥有者定义的访问策略（如基于角色、时间、地点等条件）评估请求。
3. 如果权限请求通过，AS 生成并返回一个 RPT（Requesting Party Token） 给 Client。

**资源访问阶段**

1. Client 携带 RPT 再次向 RS 请求资源访问。
2. RS 验证 RPT 的合法性（如签名、权限范围、有效期等）。
   - 如果验证通过，RS 执行请求操作（如提供资源或完成写入操作）。
   - 如果 RPT 不足以覆盖新的权限请求，RS 返回新的 Permission Ticket，流程重新回到权限请求阶段。

### Keycloak资源授权流程实例

**注意**：在进行Keycloak资源认证时要修改前端Client中Access Token的信息内容。在默认情况下Access Token中会包含Realm Role和Client Role信息，但在使用基于Role的授权策略时，Client Role内容会非常会非常多，导致前端Access Token不能存入Cookie。为解决此问题，需要通过修改Client Scope的内容来定义Access Token中包含的信息：

1. 将Client Role信息从前端Client的Access Token中剔除，后端Token不变
   - 点击Client scopes，点击roles，点击Mappers，从三个已有Mapper中删除client roles
2. 点击Create client scope，设置Name为client_roles，点击Save
   - 在新建的Scope中点击Add mapper → From predefined Mapper → 选择client roles
3. 点击Clients → 选择前端Client → 点击Client中的Client scopes → Add client scope → 勾选client roles → 将client roles的Assign Type设置为Optional
4. 点击Clients → 选择后端Client → 点击Client中的Client scopes → Add client scope → 勾选client roles → 将client roles的Assign Type设置为Default

Keycloak资源验证流程如下:

1. Keycloak中新建资源，Policy，Scope，并通过新建Permission进行绑定。
2. 用户请求受保护资源，Resource Server会带着`Access Token`请求Keycloak，索要对应资源的`Permission Ticket`。
3. Keycloak返回Permission Ticket给资源服务器，资源服务器在拿着Permission Ticket索要RPT token
   - Keycloak如果正确返回RPT则去验证RPT，并验证RPT获取受保护资源信息，验证成功则允许访问JupyterLab页面
   - 如果索要RPT失败则无法访问

整体设想流程如下，与标准的UMA方式略有不同：

#### 角色和访问流程

1. Client：前端应用
   - 访问资源URL，Cookie中带上`Access Token`
2. 资源服务器 Resource Server
   1. 判断Cookie中是否含有`Access Token`，没有则返回登录页面
   2. 查询中台被访问资源的Resource ID和Scope
   3. 请求`Permission Ticket`
   4. 请求`RPT Token`
   5. 请求验证`RPT Token`
3. 授权服务器 Authorization Server：Keycloak
   - 负责分发`Permission Ticket`和`RPT Token`，验证`RPT token`
4. 中台
   - 通过API在Keycloak中创建对应资源和Scope，记录返回的ID信息以供查询
   - 通过API给资源创建绑定Policy，需要Resource Owner的`Access Token`

![](.Keycloak简明指南_images/keycloak-authorization-workflow.svg)

### API接口测试
参考：
- [Keycloak Managing permission requests](https://www.keycloak.org/docs/latest/authorization_services/index.html#_service_protection_permission_api_papi)

#### Permission Ticket获取
通过Permission Ticket的方式访问资源，在用户无权访问受保护资源时，可以发送一条请求给资源的Owner。
```shell
curl -X POST \
http://${host}:${port}/realms/${realm}/authz/protection/permission \
  -H 'Authorization: Bearer '${access_token} \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "resource_id": "{resource_id}",
    "resource_scopes": [
      "view"
    ],
    "claims": {
      "organization": ["acme"]
    }
  }
]'
```
返回信息
```json
{
    "ticket": "${permission_ticket}"
}
```

#### 通过permission ticket获取RPT token
submit_request代表是否发送资源请求
```shell
curl -X POST \ http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \
  -H "Authorization: Bearer ${access_token}" \ 
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \ 
  --data "ticket=${permission_ticket}"
  --data "submit_request=${submit_request}"
```
返回信息
```shell
{
  "upgraded": false,
  "access_token": "${rpt}",
  "expires_in": 1800,
  "refresh_expires_in": 1800,
  "refresh_token": "",
  "token_type": "Bearer",
  "not-before-policy": 0
}
```

#### 直接获取RPT token（不使用Permission Ticket）
这里的Resource可以是Resource Id，也可以是Resource Name
```shell
curl -X POST \ http://${host}:${port}/realms/${realm}/protocol/openid-connect/token \ 
  -H "Authorization: Bearer ${access_token}" \
  --data "grant_type=urn:ietf:params:oauth:grant-type:uma-ticket" \ 
  --data "audience=${resource_server_client_id}" \ 
  --data "permission=Resource_A#Scope A" \ 
  --data "permission=Resource_B#Scope B"
```
返回信息与使用Permission Ticket获取的access token相同。

#### 验证RPT token
- basic_secret是`client_id:client_secret`的base64编码
- 结合直接获取RPT,可以用来获得Resource Name对应的Resource Id
```shell
curl -X POST  "http://${host}:${port}/realms/${realm}/protocol/openid-connect/token/introspect" \
    -H "Authorization: Basic ${basic_secret}" \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d 'token_type_hint=requesting_party_token&token=${rpt}'
```
返回信息
```shell
{
    "exp": 1616847431,
    "nbf": 0,
    "iat": 1616845631,
    "jti": "84079d88-1f34-4a3e-944a-2f233ef1a889",
    "aud": "dvclab",
    "typ": "Bearer",
    "acr": "1",
    "permissions": [
        {
            "scopes": [
                "view"
            ],
            "rsid": "${rsid}",
            "rsname": "${rsname}",
            "resource_scopes": [
                "view"
            ],
            "resource_id": "${resource_id}"
        }
    ],
    "active": true
}
```
如果该用户无权访问资源则返回
```shell
{
    "error": "access_denied",
    "error_description": "request_submitted"
}
```
且此处资源拥有者在资源管理界面会收到资源访问请求，资源拥有者可以决定是否给予此用户对应的Scope访问权限

#### 创建资源
- 这里的 `$pat` 是grant_type为client_credentials，通过client_id和client_secret获取的access_token
- 如果要设置”_id”，必须设置成UUID的形式，8-4-4-4-12，否则无法通过 id 来访问该资源

```shell
curl -v -X POST \ 
  http://${host}:${port}/realms/${realm}/authz/protection/resource_set \ 
  -H 'Authorization: Bearer $pat \ 
  -H 'Content-Type: application/json' \ 
  -d '{
    "name": "test_1",
    "type": "test",
    "owner": "admin",
    "ownerManagedAccess": true,
    "_id": "2898b46e-9a74-4cee-8f19-dccde51de020",
    "resource_scopes":[
      "visit",
      "view"
    ]
  }'
```
返回信息
```json
{
    "name": "test_1",
    "type": "test",
    "owner": {
        "id": "e407b23f-f9f7-4028-8dcd-cb4f58b7f686",
        "name": "admin"
    },
    "ownerManagedAccess": true,
    "_id": "27fad3cf-3ea8-4f65-a43b-5d4323c5241a",
    "resource_scopes": [
        {
            "id": "3f5a408c-17bb-47ae-a805-683b7c5ed28c",
            "name": "view"
        },
        {
            "id": "563ee5af-c6f0-4b1f-8814-6fdca0ec62b7",
            "name": "visit"
        }
    ],
    "scopes": [
        {
            "id": "3f5a408c-17bb-47ae-a805-683b7c5ed28c",
            "name": "view"
        },
        {
            "id": "563ee5af-c6f0-4b1f-8814-6fdca0ec62b7",
            "name": "visit"
        }
    ]
}
```

#### 更新资源信息
资源的owner无法更新
```shell
curl -v -X PUT \
  http://${host}:${port}/realms/${realm_name}/authz/protection/resource_set/{resource_id} \ 
  -H 'Authorization: Bearer '$pat \
  -H 'Content-Type: application/json' \ 
  -d '{ "_id": "Alice Resource", "name":"Alice Resource", "resource_scopes": [ "read" ] }'
```

#### 给资源创建并绑定Policy
access_token是资源拥有者的Access Token，例子：给资源绑定只有特定用户才能访问的策略：

```shell
curl -X POST \
http://${host}:${port}/realms/${realm}/authz/protection/uma-policy/${resource_id} \
  -H 'Authorization: Bearer '$access_token \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
        "name": "only-one-access",
        "description": "Allow access to one user",
        "scopes": ["visit"],
        "users": "test1"
  }'
```
返回信息
```json
{
    "exp": 1617004841,
    "nbf": 0,
    "iat": 1617003041,
    "jti": "84f2c242-fbe5-485a-be0e-44bcce893946",
    "aud": "dvclab",
    "typ": "Bearer",
    "acr": "1",
    "permissions": [
        {
            "scopes": [
                "visit"
            ],
            "rsid": "27fad3cf-3ea8-4f65-a43b-5d4323c5241a",
            "rsname": "test_2",
            "resource_scopes": [
                "visit"
            ],
            "resource_id": "27fad3cf-3ea8-4f65-a43b-5d4323c5241a"
        }
    ],
    "active": true
}
```

#### 更新资源绑定策略
access_token需要是资源owner的Access Token

```shell
curl -X PUT \
  http://${host}:${port}/realms/${realm}/authz/protection/uma-policy/${permission_id} \
  -H 'Authorization: Bearer '$access_token \
  -H 'Content-Type: application/json' \
  -d '{
    "id": "21eb3fed-02d7-4b5a-9102-29f3f09b6de2",
    "name": "Any people manager",
    "description": "Allow access to any people manager",
    "type": "uma",
    "scopes": [
        "album:view"
    ],
    "logic": "POSITIVE",
    "decisionStrategy": "UNANIMOUS",
    "owner": "7e22131a-aa57-4f5f-b1db-6e82babcd322",
    "roles": [
        "user"
    ]
}'
```

#### 删除资源绑定策略
```shell
curl -X DELETE \ 
  http://${host}:${port}/realms/${realm}/authz/protection/uma-policy/${permission_id} \
  -H 'Authorization: Bearer '$access_token
```

### 中台鉴权时序图
![](.Keycloak简明指南_images/authorization-sequence-diagram.svg)
说明：以上为用户成功访问受保护资源的情况，实际操作中还存在访问无需用户Token的接口、用户Token无效、用户未被授权（获取不到Rpt Token）、用户访问无权限管理的接口等情况
1. 无需用户Token：不进入Authenticator，直接进入请求方法对应的路由
2. 需要用户Token：
   - 用户Token无效：halt(401, “Token error”)
   - 用户Token有效未被授权：由获取Rpt Token方法抛出异常，在before中处理，halt(403, “forbidden, access denied”)
   - 用户访问无授权管理的接口：if逻辑判断resourceName为null或者scope为null时，用户Token验证通过后直接放行


<style>
body { counter-reset: h1counter h2counter h3counter h4counter h5counter h6counter; }

h1 { counter-reset: h2counter; }
h2 { counter-reset: h3counter; }
h3 { counter-reset: h4counter; }
h4 { counter-reset: h5counter; }
h5 { counter-reset: h6counter; }
h6 {}

h2:before {
    counter-increment: h2counter;
    content: counter(h2counter) "\0000a0";
}

h3:before {
    counter-increment: h3counter;
    content: counter(h2counter) "." counter(h3counter) "\0000a0";
}

h4:before {
    counter-increment: h4counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "\0000a0";
}

h5:before {
    counter-increment: h5counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "\0000a0";
}

h6:before {
    counter-increment: h6counter;
    content: counter(h2counter) "." counter(h3counter) "." counter(h4counter) "." counter(h5counter) "." counter(h6counter) "\0000a0";
}

pre {
    overflow: auto;
    white-space: pre-wrap !important;
    word-wrap: break-word !important;
    
    margin: .75rem 0;
    padding: .5rem;

    font-size: .875em;
    
    border: 1px solid #666;
    border-radius: 3px;
}
</style>