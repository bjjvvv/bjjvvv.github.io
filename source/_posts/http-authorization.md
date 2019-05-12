title: http api 认证授权技术, OAuth (Open Authorization)
date: 2019-05-12 20:22:55
categories:
tags: http 认证授权
---

> Ref 酷壳 [HTTP API 认证授权术](https://coolshell.cn/articles/19395.html)

主要的几种 HTTP 认证方式:
- HTTP Basic
- Digest Assess
- App Secret Key + HMAC
- JWT - JSON WEB TOKEN
- OAuth 1.0 - 3 legged & 2 legged
- OAuth 2.0 - Authentication Code & Client Credential

<!--more-->
### HTTP Basic
将 username:password 用 base64 编码

传输时带上 HTTP 头, **Authorization: Basic aGFvZW86Y29vbHNoZWxsCg**

服务端进行认证，失败返回 401

### Digest Access (HTTP 摘要认证_)
主要思路，将账号密码进行散列（MD5），并通过服务端和客户端生成随机数来增强散列

优点: 密码没有明文传输，只传输了 MD5

问题: 简短密码仍存在暴力破解问题

### App Secret Key + HMAC
MAC – Message Authentication Code, 是一种给消息签名的技术

HMAC - HMAC – Hash-based Authenticsation Code, 使用 HASH 的签名技术, 如 SHA-256

App ID - 标识 API 的调用方. 用来映射加密的密钥, 服务端可以生成多个密钥对 (AppID, APPSecret)

优点: 根据不同的 APPSecret 可以控制权限的粒度。

问题: 没有统一的规范，各家实现都不相同

例子: AWS

### JWT - JSON Web Tokens
JWT 一个比较标准的认证解决方案, 也是一种 MAC (Message Authentication Code) 的方法。

JWT 认证流程:
1. 用户发送账号密码到服务端请求认证
2. 服务器认证账号密码后，生成 JWT Token  
   Token 生成过程：
     - 认证服务器会生成 Secret Key (密钥)
     - 对 JWT Header 和 JWT Playload 分别求 Base64. 在 playload 可能包括了用户的抽象 ID 和过期时间
     - 用密钥对 JWT 签名, 生成 signature  
       ```
       HMAC-SHA256(SecertKey, Base64UrlEncode(JWT-Header)+'.'+Base64UrlEncode(JWT-Payload));
       ```
3. 然后把 **base64(header).base64(payload).signature** 作为 JWT token 返回给客户端
4. 客户端每次请求都带上 token
   
服务器收到请求后:
1. 检查 JWT Token, 确认是有效的
2. 因为只有认证服务器有用户的 Secret Key, 所以应用服务器需要将 JWT Token 传给认证服务器
3. 认证服务器通过 JWT Payload 解出用户的抽象 ID, 找到对应的 Secret Key, 检查签名
4. 认证服务器确认后, 应用服务器就知道是合法请求了

JWT 除了支持 HMAC-SHA256 的算法外, 还支持 RSA 的非对称加密算法

使用 RSA, 认证服务器用私钥加密，应用服务器用公钥解密。因为 RSA 是一个比较慢的算法, 并且对于越大的数据花费的时间越久. 所以可以进行一次简单的 SHA256 散列, 再对哈希码进行一次 RSA 签名

最后需要定期更换公钥私钥对

> 其实一般项目, 不会单独设立认证服务器, 甚至 Secret Key 也不是每个用户都单独生成 

### OAuth 1.0
协议的三个角色:
- User 用户
- Consumer 第三方服务
- Service Provider 服务提供方

用户需要使用第三方服务，第三方服务需要从服务提供方获取用户的资源

授权过程:
1. Consumer 先上 Service Provider 注册, 获取开发的 Consumer Key 和 Consumer Secret
2. 当 User 访问 Consumer 时, Consumer 想 Service Provider 发起请求, 获取 Request Token (用于对 HTTP 请求签名)
3. Serivce Provider 验明 Consumer 是注册过的第三方服务商后, 返回 Request Token (**oauth_token**) 和 
   Request Token Secret (**oauth_token_secret**)
4. Consumer 收到 Request Token 后, 使用 HTTP GET 请求把 User 重定向到 Service Provider 的认证页 (其中带上 
   Reqquest Token)， 让用户输入 账号和口令
5. Service Provider 认证 User 后, 跳回 Consumer, 并返回 Request Token (**oauth_token**) 和 Verification Code 
   (**oauth_verifer**)
6. 接下来就是签名请求, 用 Request Token 和 Verifiction Code 换取 Access Token (oauth_token) 和 Access Token 
   Secret (**oauth_token_secret**)
7. 最后使用 Access Token 访问用户授权访问的资源

这种三方参与的叫做 3-legged flow, 也有用户不参与的 2-legged flow.

OAuth 中的签名:
- 其中有两个密钥, 一个是在 Serivice Provider 中注册颁发的 Consumer Secret, 另一个是 Token Secret
- 签名密钥就是由这两个密钥拼接而城的, 其中用 & 作连接符号. 假设 Consumer Secret 为 dd2j3j3ke, Token
  Secret 为 s2k2j32kk, 最终的签名密钥就是 dd2j3j3ke&s2k2j32kk
- 在请求 Reqeust/Access Token 的时候需要对整个 HTTP 请求进行签名 (使用 HMAC-SHA1 和 HMAC-RSA1 签名算法), 其中
  请求头中需要包括一些 OAuth 需要的字段, 如:
  - Consumer Key: 也就是所谓的 AppID
  - Token: Request Token 或 Access Token
  - Signature Method: 签名算法, 比如: HMAC-SHA1
  - Timestamp: 过期时间
  - Nonce: 随机字符串
  - Call Back: 回调 URL

原理: 签名同时使用了用户的密钥和第三方服务的密钥, 所以可以同时对用户和第三方进行认证. Service Provider 根据 Consumer Key
和 Request Token 查找对应的 Secret, 同样拼接后进行签名认证.

## OAuth 2.0
以前的 HTTP 请求都需要用 HASH 或 RSA 进行签名. 主要原因是以前 HTTP 都是明文传输, 容易被篡改, 所以需要用签名来保证安全.

使用签名的方式是比较复杂的, 对开发者也是很不友好. 在组织 HTTP 报文的时候需要各种 URLEncode 和 Base64, 还要对 query 的参数进行排序, 有的方法还要层层签名, 非常容易出错. 另外这种安全力度比较低, 授权也比较单一, 对于有终端用户参与的移动端有点不够.

OAuth 2.0 依赖于 TLS/SSL 的链路加密技术 (HTTPS), 完全放弃了签名的方式. 所以 1.0 和 2.0 是完全不同, 不兼容的.

OAuto 2.0 的两个主要的 Flow:
- OAuthorization Code Flow, 这是 3 legged 的
- Client Credential Flow, 这个是 2 legged 的

### OAuthorization Code Flow
Authorization Code Flow 是最常使用的 OAuth 2.0 的授权许可类型, 适用于用户给第三方应用授权访问自己信息的场景.

流程:
1. 当用户 (Resource Owner) 访问第三方应用时 (Client) 的时候, 第三方应用会把用户带到认证服务器 (Authorization Server)
    上去, 主要请求的是 \authorize API, 其中请求方式如下
    ```
    https://login.authorization-server.com/authorize?
            client_id=6731de76-14a6-49ae-97bc-6eba6914391e
            &response_type=code
            &redirect_uri=http%3A%2F%2Fexample-client.com%2Fcallback%2F
            &scope=read
            &state=xcoiv98CoolShell3kch
    ```
    其中: 
    - client_id 为第三方应用的 AppID
    - response_type=code, 告诉认证服务器, 走的是 OAuthorization Code Flow
    - redirect_uri, 意思是我跳转回第三方应用的 URL
    - scope 是相关的权限
    - state 是一个随机字符串, 防止 CSRF 攻击

2. 当 Authorization Server 收到请求后, 会通过 client_id 来检查 redirect_uri 和 scope 是否合法, 如果合法, 则让用户进行登录授权
3. 当用户授权同意后, Authorization Server 会跳转回 Client, 并加上 Authorization Code.
   如:
    ```
    https://example-client.com/callback?
        code=Yzk5ZDczMzRlNDEwYlrEqdFSBzjqfTG
        &state=xcoiv98CoolShell3kch
    ```
    其中
    - 请求的链接为 1) 中的 redirect_uri.
    - state 为 1) 中的 state

4. Client 就可以使用 Authorization Code 获得 Access Token. 其需要想 Authorization Server 发出的请求如下:
    ```
    POST /oauth/token HTTP/1.1
    Host: authorization-server.com
    
    code=Yzk5ZDczMzRlNDEwYlrEqdFSBzjqfTG
    &grant_type=code
    &redirect_uri=https%3A%2F%2Fexample-client.com%2Fcallback%2F
    &client_id=6731de76-14a6-49ae-97bc-6eba6914391e
    &client_secret=JqQX2PNo9bpM0uEihUPzyrh
    ```
5. 如果没有问题, Authorization Server 会返回如下:
    ```
    {
        "access_token": "iJKV1QiLCJhbGciOiJSUzI1NiI",
        "refresh_token": "1KaPlrEqdFSBzjqfTGAMxZGU",
        "token_type": "bearer",
        "expires": 3600,
        "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciO.eyJhdWQiOiIyZDRkM..."
    }
    ```
    其中
    - access_token 就是访问令牌
    - refresh_token 用于刷新 access_token
    - id_token 就是 JWT 的 token, 其中会包含用户的 OpenID

6. 用 Access Token 请求用户的资源
    ```
    GET /v1/user/pictures
    Host: https://example.resource.com

    Authorization: Bearer iJKV1QiLCJhbGciOiJSUzI1NiI
    ```

### Client Credential Flow
没有用户参与

Client 用 client_id 和 client_secret 向 Authorization Server 要一个 Access Token, 然后使用 Access Token 访问资源

请求实例:
```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=czZCaGRSa3F0Mzpn
&client_secret=7Fjfp0ZBr1KtDRbnfVdmIw
```


### 小结
- 区分两个概念, Authentication (认证) 和 Authorization (授权). 认证是必须是本人, 通过密码, 短信认证, 人脸识别等等. 
  而授权则不需要证明是本人, token 即授权

- 区分三个概念: 编码 Base64Encode, 签名 HMAC, 加密 RSA. 编码是为了方便传输, 等同明文, 签名是为了保证信息没有被篡改, 
  加密是不要别人看到是什么信息

## 明白一些初衷
- 使用复杂地HMAC哈希签名方式主要是应对当年没有TLS/SSL加密链路的情况。
- JWT把 uid 放在 Token中目的是为了去掉状态，但不能让用户修改，所以需要签名。
- OAuth 1.0区分了两个事，一个是第三方的Client，一个是真正的用户，其先拿Request Token，再换Access Token的方法主要是为了把
  第三方应用和用户区分开来。
- 用户的Password是用户自己设置的，复杂度不可控，服务端颁发的Serect会很复杂，但主要目的是为了容易管理，可以随时注销掉。
- OAuth 协议有比所有认证协议有更为灵活完善的配置，如果使用AppID/AppSecret签名的方式，又需要做到可以有不同的权限和可以随时
  注销，那么你得开发一个像AWS的IAM这样的账号和密钥对管理的系统。

## 相关的注意事项
- 无论是哪种方式，我们都应该遵循HTTP的规范，把认证信息放在 Authorization HTTP 头中。
- 不要使用GET的方式在URL中放入secret之类的东西，因为很多proxy或gateway的软件会把整个URL记在Access Log文件中。
- 密钥Secret相当于Password，但他是用来加密的，最好不要在网络上传输，如果要传输，最好使用TLS/SSL的安全链路。
- HMAC中无论是MD5还是SHA1/SHA2，其计算都是非常快的，RSA的非对称加密是比较耗CPU的，尤其是要加密的字符串很长的时候。
- 最好不要在程序中hard code 你的 Secret，因为在github上有很多黑客的软件在监视各种Secret，千万小心！这类的东西应该放在你的- 配置系统或是部署系统中，在程序启动时设置在配置文件或是环境变量中。
- 使用AppID/AppSecret，还是使用OAuth1.0a，还是OAuth2.0，还是使用JWT，我个人建议使用TLS/SSL下的OAuth 2.0。
- 密钥是需要被管理的，管理就是可以新增可以撤销，可以设置账户和相关的权限。最好密钥是可以被自动更换的。
  认证授权服务器（Authorization Server）和应用服务器（App Server）最好分开。