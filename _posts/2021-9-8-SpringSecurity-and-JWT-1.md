---
layout: post
title: "SpringSecurity和JWT实现认证和授权（一）"
subtitle: "JWT token格式以及认证和授权的流程"
date: 2021-9-8 22:04:13
background: '/img/posts/06.jpg'
---
# JWT token的格式：
## header:
存放签名生成的算法和token的类型
例如：  

```
{
    "alg": "HS512"
    "typ": "JWT"
}
```

## payload:
用于存放一些规定的配置以及自定义的字段（JSON格式）  
其中规定的配置包含:
### iss
Issuer的简写，代表token的颁发者。
### sub
Subject的简写，代表token的主题。  
### aud
Audience的简写，代表token的接收目标。  
### exp
Expiration Time的简写，代表token的过期时间，时间戳格式。  
### nbf
Not Before的简写，代表token在这个时间之前不能被处理，主要是纠正服务器时间偏差。
### iat
Issued At的简写，代表token的颁发时间。
### jti
JWT ID的简写，代表token的id，通常当不同用户认证的时候，他们的token的jti是不同的。  

**Tips：  
注意，除了以上标准定义的字段，用户可以自由添加需要的信息，通常我们会把全局的、经常使用的、安全要求不高的信息写入载荷，比如用户ID、用户名等信息。**

## signature
signature为以header和payload生成的签名，一旦header和payload被篡改，验证将失败。

```
String signature = HMACSHA512(base64UrlEncode(header)+"."+base64UrlEncode(secret);
```

# JWT实现认证和授权的原理
用户调用登录接口，登陆成功后获取到Server发送的JWT的token。  
之后用户每次调用其他接口时，会在http请求的header中添加一个叫做Authorization的头，值为JWT的token。  
之后Server会对Authorization头中的信息解码以及校验获取其中用户的信息从而实现认证和授权。  