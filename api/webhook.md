# Webhook

**Webhook** 是 **应用** 负责接收 **苏打开放平台** 消息的服务器接口。

## 处理 Webhook 消息

### 使用已签名请求

为保证消息来源可靠，在发送消息的时候苏打开放平台将对关键数据做签名，应用 Webhook 必须根据签名规则解析并校验已签名请求，具体规则请查看 [开发指南 - 使用已签名请求](guide/signed_request.md) 章节。

### 请求和响应

消息将以 **POST** 的形式发送，并把数据以 **JSON 格式** 放置在 **请求体 Body** 中。当应用成功地解析并校验消息数据，应响应 **Http 状态 200** 以及字符串内容 **success** 给苏打开放平台。

#### 示例

Request:

```http
POST: https://MY_APP_SERVER/webhook/

signed_request=BF0BjqmK_YTCaShX23I6Tn3RZuMKyDx2YlrXU4DnEhU.eyJhbGdvcml0aG0iOiJITUFDLVNIQTI1NiIsIm9yZGVyIjp7ImlkIjoiT1JERVJfSUQuT1JERVJfSUQuT1JERVJfSUQuT1JERVJfSUQiLCJwcmljZSI6NDMuMX19
```

Response:

```http
HTTP/1.1 200 OK

success
```

### 重试机制

若发出消息后 Webhook 未响应成功状态，苏打开放平台将认为消息通知失败，并触发重试机制：

**平台将在 8 秒后重发消息；若再次失败，间隔时间加倍，以此循环。最多重试 15 次（累积约 72 小时）。**

> _第 N 次发送距离上次间隔的时长 (毫秒) = 初始间隔 × Math.pow(2, N - 1)_
>
> _最长累积时长 (小时) = 初始间隔 × (Math.pow(2, 最多重试次数) - 1) / 60 / 60_

### 事件通知完整流程

```plantuml
autonumber

苏打开放平台 -> 苏打开放平台 : 触发了一起事件
苏打开放平台 -> 苏打开放平台 : 对事件数据进行签名
苏打开放平台 -> 应用_Webhook : 发送已签名请求（消息）

loop 当应用未能响应成功状态，最多重试 15 次
苏打开放平台 --> 苏打开放平台 : 等待重试（间隔逐渐加长）
苏打开放平台 -> 应用_Webhook : 重新推送消息
end

应用_Webhook -> 应用_Webhook : 校验通知的签名，并解析数据
应用_Webhook -> 苏打开放平台 : 返回已校验解析成功
应用_Webhook --> 应用_Webhook : 使用通知的数据
```
