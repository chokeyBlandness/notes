### 一、什么是JWT

Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（[(RFC 7519](https://link.jianshu.com?t=https://tools.ietf.org/html/rfc7519)).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

#### 背景

互联网服务离不开用户认证。一般流程是下面这样。

> 1、用户向服务器发送用户名和密码。
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

**另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。**

### 二、JWT数据结构

~~~java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
~~~

它是一个很长的字符串，中间用点（`.`）分隔成三个部分。注意，JWT 内部是没有换行的，这里只是为了便于展示，将它写成了几行。

JWT 的三个部分依次如下。

- Header（头部）
- Payload（载荷）
- Signature（签名）

#### 1、Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

 ```json
 {
   "alg": "HS256",
   "typ": "JWT"
 }
 ```

上面代码中，`alg`属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ`属性表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`。

最后，将上面的 JSON 对象使用 Base64URL 算法转成字符串（可逆）。

#### 2、Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。

JWT 规定了7个官方字段，供选用。

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子。

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串（可逆）。

#### 3、Signature

Signature 部分是对前两部分的签名，防止数据篡改。非对称加密。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照公式产生签名。

### 三、过期时间

1、jwt可以设置过期时间

```java
 public static String createJwt(String subject, String payload) {
    return Jwts.builder().setSubject(subject)
            // 设置过期时间
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60))
            .signWith(SignatureAlgorithm.HS256, PRIVATE_KEY)
            .compact();
}
```

2、配合redis实现

### 四、案例

```java
public class JwtUtil {

    private final static String PRIVATE_KEY = "123";

    public static String createJwt(String subject, String payload) {
        return Jwts.builder().setSubject(subject)
                // 设置过期时间
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60))
                .signWith(SignatureAlgorithm.HS256, PRIVATE_KEY)
                .compact();
    }

    public static String parse(String token) {
        Claims claims = Jwts.parser().setSigningKey(PRIVATE_KEY).parseClaimsJws(token).getBody();
        return claims.getSubject();
    }

}
```

maven依赖

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```





