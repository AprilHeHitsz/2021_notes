## 零散知识积累

### JWT

- JSON web token，是为了在网络应用环境间传递声明而执行的一种**基于JSON**的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)).该token被设计为紧凑且安全的，特别适用于**分布式站点**的单点登录（SSO）场景。

  JWT的声明一般被用来在**身份提供者**和**服务提供者**间传递**被认证的用户身份信息**，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

  JWT一般在请求头里加入Authorization，并加上Bearer标注。

- 构成：head+payload+signature。

  - 将这三段信息文本用`.`链接一起就构成了Jwt字符串。

    e.g.

    - 编码的JWT

    ```
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    ```

    - 将以上JWT解码后如下

    ```
    #header:
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    #payload
    {
      "sub": "1234567890",
      "name": "John Doe",
      "iat": 1516239022
    }
    #signature
    HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      your-256-bit-secret
    
    )
    ```

    

  - 其中，在isgniture中的secret是保存在服务端用来进行jwt的签发和jwt的验证，这是服务端的私钥，不应该被客户知道（否则客户就可以自己签发了）。

  对于其构成的详细介绍，查阅[参考博客](https://www.jianshu.com/p/576dbf44b2ae)

  

  

- 应用：（Oauth2的jwt流程）

  ![img](https://upload-images.jianshu.io/upload_images/1821058-2e28fe6c997a60c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 特点

  因为json的通用性，所以JWT是可以进行跨语言支持的

  有payload部分，JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息

  jwt的构成非常简单，便于传输

  易于应用的扩展