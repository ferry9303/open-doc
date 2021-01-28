# 开通应用

如需开通应用，请向我们的对接人员提供以下信息：

- name, 应用名
- oauth2_callback_origins, 授权登录的白名单回调的[源地址（即协议名 + 域名 + 端口号（如有指定端口）的地址）](faq.md?id=oauth2_callback_origin 中的 origin 具体指什么)
- webhook, 应用接收苏打平台服务器通知的完整地址

如：

    name: my_first_app
    oauth2_callback_origins:
        - https://app.dev:8080
        - https://app.dev
        - htpp://app.dev
    webhook: https://webhook.app.dev/soda/

随后我们将以人工邮件的形式为你发送开发所需要的以下信息，请注意查收：

- client_id，平台为应用分配的 ID
- client_secret，平台为应用分配的密钥
- scopes, 开放可用的全部授权范围列表
- quiet_scopes，开放可用的静默授权范围列表（免用户确认即可通过）
