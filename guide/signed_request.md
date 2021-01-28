# 使用已签名请求

当苏打平台向应用 Webhook 发送请求时，会将请求的部分数据放入 **signed_request** 参数中，应用必须对其进行解析和验证，以确保数据和来源可靠，然后才能使用。

## 解析和验证

**signed_request** 使用 base64url 编码，并使用应用密钥 __client_secret__ 做 HMAC-SHA256 签名。

**解析和验证的过程：**

1. 用 `.` 字符将 **signed_request** 分为两个部分（例如：238fsdfsd.oijdoifjsidf899）。
2. 用 base64url 解码第一部分，获得编码后的签名。
3. 以 client_secret 作为密钥，用 HMAC-SHA256 对第二部分加密，如结果与上一步获得的签名值一致则验证通过。
4. 用 base64url 解码第二部分，获得 JSON 字符串，最后解码获得有效的 JSON 对象数据。

## 示例

### PHP

```php
function parse_signed_request($signed_request) {
  list($encoded_sig, $payload) = explode('.', $signed_request, 2);

  $secret = "appsecret"; // Use your client_secret here

  // decode the data
  $sig = base64_url_decode($encoded_sig);
  $data = json_decode(base64_url_decode($payload), true);

  // confirm the signature
  $expected_sig = hash_hmac('sha256', $payload, $secret, $raw = true);
  if ($sig !== $expected_sig) {
    error_log('Bad Signed JSON signature!');
    return null;
  }

  return $data;
}

function base64_url_decode($input) {
  return base64_decode(strtr($input, '-_', '+/'));
}
```

> _摘自 [facebook for developer - 使用已签名请求](https://developers.facebook.com/docs/games/gamesonfacebook/login#usingsignedrequest)_

### Node.js

```javascript
var crypto = require("crypto");
var base64url = require("base64-url");

function sign(text, secret) {
  return base64url.escape(
    crypto
      .createHmac("sha256", secret)
      .update(text)
      .digest("base64")
  );
}

exports.parse = function parse(request, secret) {
  var fragments = request.split(".", 2);

  // confirm the signature
  if (sign(fragments[1], secret) !== fragments[0]) {
    throw new Error("Bad Signed JSON signature!");
  }

  return JSON.parse(base64url.decode(fragments[1]));
};
```

> _您也可以直接使用我们封装好的 [@sodalife/signed-request](https://www.npmjs.com/package/@sodalife/signed-request)。_

### Golang

```go
import (
  "bytes"
  "crypto/hmac"
  "crypto/sha256"
  "encoding/base64"
  "encoding/json"
  "fmt"
  "strings"
)

func Parse(request string, secret string) (data map[string]interface{}, err error) {
  fragments := strings.SplitN(request, ".", 2)
  if len(fragments) != 2 {
    err = fmt.Errorf("invalid format of signed request")
    return
  }

  signature, err := base64.RawURLEncoding.DecodeString(fragments[0])
  if err != nil {
    return
  }

  hash := hmac.New(sha256.New, []byte(secret))
  hash.Write([]byte(fragments[1]))
  expected := hash.Sum(nil)

  if bytes.Compare(signature, expected) != 0 {
    err = fmt.Errorf("invalid signature of signed request")
    return
  }

  payload, err := base64.RawURLEncoding.DecodeString(fragments[1])
  if err != nil {
    return
  }

  err = json.Unmarshal(payload, &data)
  return
}
```

## 外部文献参考

- [facebook for developer - 使用已签名请求](https://developers.facebook.com/docs/games/gamesonfacebook/login#usingsignedrequest)
- [facebook for developer - Echo](https://developers.facebook.com/tools/echo)
- [Github - nick / signed-request-example](https://github.com/nick/signed-request-example/)
- [Github - mkdynamic / omniauth-facebook](https://github.com/mkdynamic/omniauth-facebook)
- [Github - ptarjan / signed-request](https://github.com/ptarjan/signed-request)
