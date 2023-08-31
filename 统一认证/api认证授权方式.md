RESTful 接口的授权方式有多种，以下是一些常见的方式：

1. HTTP Basic Authentication（HTTP 基本认证）：
   客户端在请求头中使用 Base64 编码的用户名和密码进行认证。例如：
   ```
   Authorization: Basic base64_encoded(username:password)
   ```

2. API Key（密钥认证）：
   客户端将预先生成的 API 密钥（通常是长字符串）作为请求的一部分发送到服务器。例如：
   ```
   Authorization: ApiKey YOUR_API_KEY
   ```

3. Bearer Token（令牌认证）：
   客户端在请求头中使用令牌进行认证。令牌通常是由认证服务器颁发的，表示用户或客户端的身份。例如：
   ```
   Authorization: Bearer YOUR_ACCESS_TOKEN
   ```

4. OAuth 2.0：
   OAuth 2.0 是一种开放标准的授权框架，允许用户授权第三方应用访问他们在另一个服务提供商上的资源。OAuth 2.0 定义了多种授权方式，如授权码模式、隐式授权模式、密码授权模式和客户端凭证授权模式等。

5. JWT（JSON Web Token）：
   JWT 是一种用于安全传输信息的开放标准，通常用于在客户端和服务器之间传递声明信息。JWT 可以用作身份验证和授权的一种方式。

6. Digest Authentication（摘要认证）：
   类似于 HTTP 基本认证，但在发送密码之前会进行加密处理，提供更高的安全性。

7. AWS Signature（AWS 签名认证）：
   用于对亚马逊网络服务 (AWS) 的请求进行认证和授权。

请注意，每种授权方式都有其适用的场景和安全考虑。选择合适的授权方式取决于你的应用程序需求和安全需求。