---
title: JWT
date: 2024-04-02 11:04:19
tags:
  - JWT
  - 身份认证
---
# JWT

JSON Web Token（缩写 JWT）是目前最流行的跨域认证解决方案

JWT（JSON Web Token）是一种开放标准（RFC 7519），用于在不同实体之间安全地传输信息。它是一种基于 JSON 的轻量级令牌，通常用于身份验证和授权机制

## 跨域认证问题

互联网服务离不开用户认证。一般流程是下面这样

```markdown
1. 用户向服务器发送用户名和密码

2. 服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等

3. 服务器向用户返回一个 session_id，写入用户的 Cookie

4. 用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器

5. 服务器收到 session_id，找到前期保存的数据，由此得知用户的身份
```

![图片](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202404021146609.png)

这种方案有一些缺点：

- 需要从内存或数据库里存取 session 数据
- 扩展性差，对于分布式应用，需要实现 session 数据共享

为了解决上面的问题，可以将所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表

![78fa0f2ef25b9527abcf0507d1bc695b](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202404021148349.png)

该方案将需要使用到的用户数据等信息放入 JWT 里面，每次请求都会携带上，只要保证密钥不泄露，JWT 就无法伪造。返回信息示例：

```json
{
  "姓名": "张三",
  "角色": "管理员",
  "到期时间": "2018年7月1日0点0分"
}
```

## 数据格式

一个典型的 JWT 就像这样

```
eyJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiemhhbmdzYW4iLCJpZCI6MjB9.FFt1xncPjUu-R_Dt0SenhPpwcZscH8RaybRt9oLV0Ss
```

将上面的数据放到 [jwt.io](https://jwt.io/) 中进行解析，结果如下

![image-20240402152753908](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202404021527098.png)

可以看出 JWT 以不同颜色区分，两个小数点隔开，分为了**三部分**：

- Header（头部）
- Payload（负载）
- Signature（签名）

### Header

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子

```java
{
  "alg": "HS256",
  "typ": "JWT"
}
```

上面代码中，`alg` 属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；`typ` 属性表示这个令牌（token）的类型（type），JWT 令牌统一写为 `JWT`

### Payload

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用

```markdown
- iss (issuer)：签发人

- exp (expiration time)：过期时间

- sub (subject)：主题

- aud (audience)：受众

- nbf (Not Before)：生效时间

- iat (Issued At)：签发时间

- jti (JWT ID)：编号
```

除了官方字段，还可以在这个部分定义私有字段，下面就是一个例子

```json
{
  "name": "zhangsan",
  "id": 20
}
```

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分

### Signature

Signature 部分是对前两部分的签名，防止数据篡改

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，就可以返回给用户

## 使用方式

客户端收到服务器返回的 JWT，可以储存在 Cookie 里面，也可以储存在 localStorage。

此后，客户端每次与服务器通信，都要带上这个 JWT。你可以把它放在 Cookie 里面自动发送，但是这样不能跨域，所以更好的做法是放在 HTTP 请求的头信息 `Authorization` 字段里面。

```javascript
Authorization: Bearer <token>
```

另一种做法是，跨域的时候，JWT 就放在 POST 请求的数据体里面

## 特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输

## Java 实战

可以使用Java的JWT库（如jjwt）来生成和验证JWT

```java
public class JjwtTest {
    private static final String SECRET_KEY = "4E04F2FD4A822A823C88598E5F059A96AB"; // 请替换为你的密钥

    // 生成 JWT Token
    public static String generateToken(String id, String subject, long ttlMillis) {
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);

        // 设置过期时间
        long expMillis = nowMillis + ttlMillis;
        Date exp = new Date(expMillis);

        // 自定义负载
        Map<String,Object> claims = new HashMap<>();
        claims.put("name","zhangsan");
        claims.put("id",20);

        // 使用 HS256 算法和密钥生成 JWT
        return Jwts.builder()
                .setId(id) // 设置 JWT 的唯一标识
                .setSubject(subject) // 设置主题
                .setIssuedAt(now) // 设置签发时间
                .setExpiration(exp) // 设置过期时间
                .addClaims(claims)
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY) // 签名算法以及密钥
                .compact();
    }

    // 验证 JWT Token
    public static Claims verifyToken(String token) {
        // 解析 Token 的 Claims
        return Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody();
    }

    // 测试方法
    public static void main(String[] args) {
        String jwt = generateToken("123", "user1", 3600000); // 生成 Token
        System.out.println("JWT Token: " + jwt);

        Claims claims = verifyToken(jwt); // 验证 Token
        System.out.println("Token Claims: " + claims);
    }
}
```

