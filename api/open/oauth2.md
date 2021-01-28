### 用户授权页

**URI:**

    https://m.sodalife.xyz/oauth2/authorize

**请求方式：**

    GET（使用浏览器访问）

**请求参数 Query:**

| 参数          | 说明                                                                | 是否必填 |
| ------------- | ------------------------------------------------------------------- | -------- |
| client_id     | [平台为应用分配的 ID](guide/client.md)                              | 必填     |
| redirect_uri  | 授权后重定向的回调地址，受[域名白名单](guide/client.md)保护         | 必填     |
| response_type | 目前固定为 `code`                                                   | 必填     |
| scope         | [授权范围](guide/auth.md?id=授权范围)，多个时使用半角逗号 `,` 分隔  | 可选     |
| state         | 内部状态，将跟随在回调地址返回，支持最大 128 字节的 a-Z0-9 的字符串 | 可选     |

**返回：**

- 正常且 scope 均被允许静默授权时，页面将直接回调至 **redirect_uri?code=CODE&state=STATE**。
- 正常但 scope 不能直接被静默授权时，页面将经用户确认授权后回调至 **redirect_uri?code=CODE&state=STATE**。
- 异常时，页面将停留并展示异常相关的信息。

**返回参数：**

| 参数  | 说明                                                           |
| ----- | -------------------------------------------------------------- |
| code  | 用于获取用户授权凭证的一次性 Authorization Code，有效期 5 分钟 |
| state | 请求时传入的内部状态                                           |

**示例：**

Request:

```http
GET: https://m.sodalife.xyz/oauth2/authorize?client_id=1db3265caf6368a60eb1719b5f34cf15&redirect_uri=http%3A%2F%2Fapp.dev%2Fconnect%2Fsoda%2F&response_type=code&scope=&state=a931586b6a985a69
```

Response:

```http
HTTP/1.1 302 Found
Location: http://app.dev/connect/soda/?code=c13367945d5d4c91047b3b50234aa7ab&state=a931586b6a985a69
```

### 获取授权凭证

**URI:**

    https://open.sodalife.xyz/oauth2/access_token

**请求方式：**

    POST

**请求参数 Body (JSON):**

| 参数          | 说明                                                               | 获取应用凭证时              | 获取用户凭证时                     |
| ------------- | ------------------------------------------------------------------ | --------------------------- | ---------------------------------- |
| grant_type    | 授权类型                                                           | 固定为 `client_credentials` | 固定为 `authorization_code`        |
| client_id     | [平台为应用分配的 ID](guide/client.md)                             | 必填                        | 必填                               |
| client_secret | [平台为应用分配的密钥](guide/client.md)                            | 必填                        | 必填                               |
| scope         | [授权范围](guide/auth.md?id=授权范围)，多个时使用半角逗号 `,` 分隔 | 可选                        | 无效，以 **用户授权页** 提交的为准 |
| code          | 用于获取用户授权凭证的一次性 Authorization Code                    | 无效                        | 必填                               |

**返回：**

- 正常时，接口将返回以下 JSON 数据：

  ```json
  {
    "access_token": "ACCESS_TOKEN",
    "expires_in": 7200
  }
  ```

- 异常时，接口将返回 [异常 JSON 数据](api/error.md?id=异常格式-json)

**返回参数：**

| 参数         | 说明                                                                              |
| ------------ | --------------------------------------------------------------------------------- |
| access_token | 授权凭证                                                                          |
| expires_in   | 凭证有效时间，单位为秒                                                            |
| user_id      | 应用用户 ID，仅当获取用户凭证时返回。可便于开发者获取用户标识，但非 oauth2 标准。 |

**示例：**

- 获取用户授权凭证

  ```bash
  curl -X POST \
    -H "Content-Type: application/json" \
    -d \
    '{
      "grant_type": "authorization_code",
      "client_id": "1db3265caf6368a60eb1719b5f34cf15",
      "client_secret": "5ebe2294ecd0e0f08eab7690d2a6ee69",
      "code": "c13367945d5d4c91047b3b50234aa7ab"
    }' \
    https://open.sodalife.xyz/oauth2/access_token

  # { "access_token": "ACCESS_TOKEN", "expires_in": 7200, "user_id": "1b9efbab658fbd15557cf1c43bb89ff2" }
  ```

- 获取应用授权凭证

  ```bash
  curl -X POST \
    -H "Content-Type: application/json" \
    -d \
    '{
      "grant_type": "client_credentials",
      "client_id": "1db3265caf6368a60eb1719b5f34cf15",
      "client_secret": "5ebe2294ecd0e0f08eab7690d2a6ee69",
      "scope": "iot:public,iot:control"
    }' \
    https://open.sodalife.xyz/oauth2/access_token

  # { "access_token": "ACCESS_TOKEN", "expires_in": 7200 }
  ```
