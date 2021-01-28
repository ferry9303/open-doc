# 异常

## 异常格式 (JSON)

接口在遇到异常时，如预期响应格式为 JSON，将返回以下数据：

```json
{
  "error": "ERROR_NAME",
  "error_description": "ERROR_DESCRIPTION",
  "error_uri": "https://ERROR_URI"
}
```

**返回参数：**

| 参数              | 说明                   |
| ----------------- | ---------------------- |
| error             | 异常的机器码           |
| error_description | 异常的描述             |
| error_uri         | 阐述异常详细信息的网页 |

## 异常码

### 通用异常码

| 异常机器码 (error)      | 异常描述 (error_description)                        |
| ----------------------- | --------------------------------------------------- |
| INTERNAL_SERVER_ERROR   | 服务内部异常                                        |
| TEMPORARILY_UNAVAILABLE | 服务暂时无法访问                                    |
| TOO_MANY_REQUESTS       | 请求频率过高                                        |
| NOT_FOUND               | 找不到服务                                          |
| INVALID_RESOURCE        | 资源无效（如找不到资源）                            |
| INVALID_REQUEST         | 请求不合法（如参数错误）                            |
| UNAUTHORIZED_CLIENT     | 应用未有权限做该操作 (如应用未有获取授权凭证的权限) |
| ACCESS_DENIED           | 用户或授权服务器拒绝授予数据访问权限                |
| LOCKED                  | 资源已被锁定                                        |
| GONE                    | 资源已失效                                          |
| CANCELLED               | 操作已被取消                                        |
| COMPLETED               | 操作已被完成                                        |
| UNAUTHORIZED            | 当前会话无效（如已过期）                            |
| UNAUTHORIZED_USER       | 当前会话用户无效（如未登录）                        |
| EXTERNAL_SERVER_ERROR   | 外部服务异常                                      |

### 授权异常码

| 异常机器码 (error)        | 异常描述 (error_description)                                   |
| ------------------------- | -------------------------------------------------------------- |
| INVALID_CLIENT            | client_id 或 client_secret 参数无效                            |
| INVALID_GRANT             | 提供的 Access Grant (code or token) 是无效的、过期的或已撤销的 |
| INVALID_SCOPE             | 请求的 scope 无效                                              |
| INVALID_REDIRECT_URI      | 提供的重定向地址与白名单不匹配                                 |
| UNSUPPORTED_RESPONSE_TYPE | 不支持的 Response Type                                         |
| UNSUPPORTED_GRANT_TYPE    | 不支持的 Grant Type                                            |

### IoT 相关异常码

| 异常机器码 (error)         | 异常描述 (error_description) |
