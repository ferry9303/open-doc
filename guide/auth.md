# 授权

苏打开放平台的授权协议基于 [RFC 6749](http://tools.ietf.org/html/rfc6749) 的 OAuth 2.0 框架 。

## 概念

### 角色

系统中存在以下角色：

| 角色                 | 说明                                         |
| -------------------- | -------------------------------------------- |
| User                 | 终端用户，即 OAuth 2.0 中的 Resource Owner。 |
| Client               | 请求授权者，接入苏打开放平台的某一具体应用。 |
| Authorization Server | 提供授权能力的苏打开放平台认证服务。         |
| Resource Server      | 提供读写资源能力的苏打开放平台 API Server。  |

### 授权类型

我们为每个应用提供两种授权类型：

- 应用授权

  由应用直接询问 Authorization Server 派发凭证 access_token (grant_type = **client_credentials**)，因此每个凭证均代表着一个应用。

  流程对应 OAuth 2.0 内建的 **Client Credentials Grant Type Flow**。

- 用户授权

  由用户 _(user)_ 授权 Authorization Server 给应用派发凭证 access_token (grant_type = **authorization_code**)，因此每个凭证均代表着一个应用内的一个具体用户（授权人）。

  流程对应 OAuth 2.0 内建的 **Authorization Code Grant Type Flow**。

?> _其余两种未被提及的 OAuth 2.0 内建授权流程为 Implicit Grant Type Flow 和 Resource Owner Password Credentials Grant Type Flow。_

### 授权范围

**由应用授权的范围**:

| scope 值            | 说明               |
| ------------------- | ----------------- |
| iot:public          | 控制 IoT 设备公有权限 |
| iot:control         | 控制 IoT 设备操作权限 |
| iot:premier         | 控制 IoT 设置高级权限 |

**由用户授权的范围**:

| scope 值               | 说明                                                 |
| ---------------------- | ---------------------------------------------------- |
| user:public (no scope) | 读取授权人的公共信息，包含 id、用户名、头像。        |
| user:sex               | 读取授权人的性别信息                                 |
| user:mobile            | 读取授权人的手机号                                   |
| user:wechat_oa         | 读取授权人在缺省微信公众号（苏打生活）下的账户       |
| user:wechat_union      | 读取授权人在缺省微信开放平台（苏打生活）下的唯一账户 |

!> 用户授权范围始终包含 `user:public`，因此在实际请求授权时可以省略该 scope 值。

## 应用授权

> 由应用直接询问 Authorization Server 派发凭证 access_token (grant_type = **client_credentials**)；每个凭证均代表着一个应用。
>
> 流程基于 OAuth 2.0 内建的 **Client Credentials Grant Type Flow**。

在调用平台部分接口时，应用需出示应用授权凭证 **client_credentials access_token** 。

该凭证受以下安全策略保护：

- 有效期目前为 7200 秒（两小时）。
- 在每个应用中均全局唯一，重复获取会使旧凭证作废。
- 获取凭证的接口存在一定频率限制。

!> 我们建议开发者统一管理并缓存该凭证，以避免出现多系统间相互竞争的问题。

### 如何获取应用授权凭证

获取应用授权凭证的流程与 OAuth 2.0 的 Client Credentials Flow 一致，但具体的接口实现有所差异。

**Client Credentials Flow:**

```
     +---------+                                  +---------------+
     |         |                                  |               |
     |         |>--(A)- Client Authentication --->| Authorization |
     | Client  |                                  |     Server    |
     |         |<--(B)---- Access Token ---------<|               |
     |         |                                  |               |
     +---------+                                  +---------------+
```

> _摘自 RFC6749，Figure 6: Client Credentials Flow_

**流程：**

1. 请求获取凭证接口，使用 client_id 和 client_secret 换取 access_token；grant_type 固定为 `client_credentials`。

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d \
  '{
    "grant_type": "client_credentials",
    "client_id": "CLIENT_ID",
    "client_secret": "CLIENT_SECRET",
    "scope": "iot:public,iot:control"
  }' \
  https://open.sodalife.xyz/oauth2/access_token

# { "access_token": "ACCESS_TOKEN", expires_in: 7200 }
```

### API

- [获取授权凭证](api/open.md?id=获取授权凭证)

## 用户授权

> 由用户 _(user)_ 授权 Authorization Server 给应用派发凭证 access_token (grant_type = **authorization_code**)；每个凭证均代表着一个应用内的一个具体用户（授权人）。
>
> 流程基于 OAuth 2.0 内建的 **Authorization Code Grant Type Flow**。

当应用期望直接操作某个用户的资源的时候，应先得到用户的授权，然后在调用平台接口时，出示用户授权凭证 **authorization_code access_token**。

该凭证受以下安全策略保护：

- 有效期目前为 7200 秒（两小时）。
- 获取凭证的接口存在一定频率限制（较宽松）。

### 如何获取用户授权凭证

获取应用授权凭证的流程与 OAuth 2.0 的 Authorization Code Flow 一致，但具体的接口实现有所差异。

**Authorization Code Flow:**

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
```

> _摘自 RFC6749，Figure 3: Authorization Code Flow_

**流程：**

1. 引导用户进入授权页

```http
GET: https://m.sodalife.xyz/oauth2/authorize?client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&response_type=code&scope=&state=STATE
```

2. 如果用户同意授权，或者申请的授权范围均被[允许静默通过](guide/client.md)，页面则跳转至 REDIRECT_URI?code=CODE
3. 请求获取凭证接口，使用 code 换取 access_token

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d \
  '{
    "client_id": "CLIENT_ID",
    "client_secret": "SECRET",
    "code": "CODE",
    "grant_type": "authorization_code"
  }' \
  https://open.sodalife.xyz/oauth2/access_token

# { "access_token": "ACCESS_TOKEN", expires_in: 7200 }
```

### API

- [用户授权页](api/open.md?id=用户授权页)
- [获取授权凭证](api/open.md?id=获取授权凭证)

## 外部文献参考

- OAuth 2.0
  - [RFC 6749](https://tools.ietf.org/html/rfc6749)
- Client credentials grant type flow
  - [RFC 6749 - section 4.4](https://tools.ietf.org/html/rfc6749#section-4.4)
  - [微信公众平台 - 获取 access_token](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183)
  - [微博开放平台 API - oauth2/access_token](http://open.weibo.com/wiki/Oauth2/access_token)
- Authorization code grant type flow
  - [RFC 6749 - section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)
  - [微信公众平台 - 微信网页授权](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)
  - [微博开放平台 API - 微博登录 / 授权机制](http://open.weibo.com/wiki/%E6%8E%88%E6%9D%83%E6%9C%BA%E5%88%B6)
