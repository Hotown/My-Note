# OAuth 2.0 理解

> An open protocol to allow secure API authorization in a simple and standard method from web, mobile and desktop applications.

OAuth是一种开放的协议，为桌面、手机或web应用提供了一种简单的，标准的方式去访问需要用户授权的API服务。

OAuth协议的特点是：简单，安全，开放。

本文主要对OAuth协议的流程做一个简单的总结，主要参考材料为_[理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)_ 、_[RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.1)_
## 一、应用场景

这里举一个简单的例子来理解OAuth的使用场景：

用户A（User A）想要调用两项服务：服务A（Server A）是一个云存储平台，用来存储照片，服务B（Server B）是一个用来打印照片的平台。

现在，**用户需要用服务B来打印存储在服务A上的照片，然而服务A于服务B属于两个不同的平台，用户需要在两个服务平台上注册不用的用户，而且用户名和密码都不相同**，这种情况下需要如何处理呢？

方法一：用户从服务A上将照片下载下来，然后上传到服务B进行打印。

方法二：用户将服务A上的用户名和密码提供给服务B，由服务B登录这个账号，再从服务A上下载照片进行打印处理。

这两种方法的缺陷显然都很明显。

```
方法一：
    繁琐，虽然相对而言安全性较高，但用户体验较差。
    
方法二：
    (1) 服务B必须创建相应的数据库表来储存用户在服务A上的信息，在不考虑空间损耗的问题下，更重要的是安全性很低。
    (2) 服务B会拥有用户在服务A上的所有权限，查看甚至篡改资源。
    (3) 用户想要剥夺服务B权力的途径只有修改用户在服务A上的密码，但这将会造成所有被服务A授权的服务失效。
```

OAuth的出现则很好地解决了这个安全与繁琐之间的矛盾。

## 二、相关名词定义

```
(1) Third-party Application：第三方应用，又称客户端（client），即例子中的服务B。

(2) HTTP Service：HTTP服务提供商，简称服务提供商，即例子中的服务A。

(3) Resource Owner：资源拥有者，即用户。

(4) User Agent：用户代理，在此指浏览器。

(5) Authorization Server：认证服务器，即服务提供商用来处理认证的服务器。

(6) Resource Server：资源服务器，即服务提供商用来存放用户生成的资源的服务器。它和认证服务器可以为同一台，也可分开。
```

## 三、OAuth工作思路

OAuth在“客户端”和“服务提供者”之间，设置了一个新的层，叫做授权层(authorization layer)。

“客户端”不能直接登录“服务提供者”，而是登录授权层，以此起到了隔离“客户端”和“服务提供者”的作用。同时，授权层区分了用户和客户端的概念，“客户端”登录授权层，所需要的就是我们所熟知的令牌（token），这与用户密码不同，我们可以为token指定使用范围，以及有效期。

## 四、OAuth工作流程

![](http://image.beekka.com/blog/2014/bg2014051203.png)

```
(A) 用户打开客户端，客户端向用户请求授权。

(B) 用户同意给客户端授权。

(C) 客户端用上一步中用户的授权，向认证服务器申请token

(D) 认证服务器经过认证后，返回给用户一个token

(E) 客户端使用token，向资源服务器申请获取资源。

(F) 资源服务器验证token，同意客户端获取资源的请求。
```

总体来说，OAuth的概念和中间件类似，通过授权服务器以及token，来隔断客户端和服务提供者之间的直接沟通。

接下来，简单理解一下客户端获取授权的四种模式。

OAuth2.0的核心既是去的access_token，四种授权模式的最终目的也在于此，只是路径不同，参数不同罢了。

## 五、客户端授权模式

OAuth2.0定义了四中授权的方式：

+ 授权码模式（authorization code）
+ 简化模式（implicit）
+ 密码模式（resource owner password credentials）
+ 客户端模式（client credentials）

## 六、授权码模式

授权码模式（authorization code）

![](http://image.beekka.com/blog/2014/bg2014051204.png)

```
(A) 客户端通过将资源拥有者的客户端（例子中的服务B）导向授权端点，即授权服务器，来启动授权流程。客户端中的信息包括：客户端标识符（Client Identifier），请求范围(Requested Scope)，本地状态(Local State)以及一个重定向URI（Redirection URI）。资源服务器将通过这个重定向URI来返回授权码（Authorization Code）。

(B) 授权服务器认证资源所有者（通过用户代理（User-Agent）），并确定资源所有者授予或拒绝客户端的访问请求。

(C) 假设资源所有者给予访问权限，服务器将用户代理重定向回客户端之前提供的重定向URI（在请求中或客户注册之前）。重定向URI包括一个授权代码（Authorization Code）和之前客户端提供的任何本地状态(Local State)。

(D) 客户端将授权码以及之前的重定向URI向认证服务器申请令牌(toekn)，这一步通常在客户端的后台服务器上完成，对用户不可见。

(E) 认证服务器核对授权码和重定向URI，确认后，向客户端发送访问令牌(access-token)和更新令牌(refresh token)(可选)。
```

A步骤中，客户端申请认证的URI，包括以下参数：

+ response_type：必选，表示授权类型，此模式中指定为"code"
+ client_id：必选，表示客户端ID
+ redirect_uri：可选，表示重定向URI
+ scope：可选，表示申请的权限范围
+ state：推荐，一个被客户端所使用的不透明的值，用来维持request和callback的状态。认证服务器将在重定向用户代理（User Agent）到客户端（Client）时带上这个值。这个参数**应当**在防止跨站点请求伪造时被使用。

举例：

```
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

C步骤中，服务器响应客户端的URI，包括以下参数：

+ code：必选，表示授权码。有效期很短，通常为10min，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应的关系。
+ state：如果客户端的请求中包括这个参数，认证服务器的回应也会相同的值。

举例：

```
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
          &state=xyz
```

D中，客户端向认证服务器发起的申请令牌请求，包含以下参数：

+ grant_type：必选，表示使用的授权模式，此模式中固定为"authorization_code"。
+ code：必选，表示上一步获取的授权码。
+ redirect_uri：必选，表示重定向URI，与A步骤中保持一致。
+ client_id：必选，表示客户端ID。

举例：

```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

E中，认证服务器向客户端发起的回复，包含以下参数：

+ access_token：必选，表示访问令牌。
+ token_type：必选，表示令牌类型，大小写不敏感，可以为bearer或mac类型。
+ expires_in：过期时间，单位秒。若省略该参数，则必须以其他方式设置过期时间。
+ refresh_token：可选，表示更新令牌，用于获取下一次的访问令牌。
+ scope：表示权限范围，若与客户端申请的范围一致，此项可忽略。

举例：

```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
    "access_token":"2YotnFZFEjr1zCsicMWpAA",
    "token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
}
```

## 七、简化模式

简化模式(implicit grant type)不通过第三方应用程序的服务器，而是直接在浏览器中向认证服务器申请令牌，从而跳过了”授权码“的步骤。所有步骤都在浏览器完成，令牌对访问者可见，且客户端不需要认证。这种客户端通常是使用脚本语言在浏览器实现的，如JavaScript。

![](http://image.beekka.com/blog/2014/bg2014051205.png)

```
(A) 客户端将用户导向认证服务器。

(B) 用户决定是否授权给客户端。

(C) 若用户给予授权，认证服务器将用户导向客户端指定的“重定向URI”，并在URI的Hash部分包含了访问令牌。

(D) 浏览器向资源服务器请求，不包括上一步收到的Hash值。

(E) 资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。

(F) 浏览器执行上一步获取的脚本，提取令牌。

(G) 浏览器将令牌返回给客户端。
```

参数：

A中，客户端发出的HTTP请求包括

+ response_type：必选，表示授权类型，此模式中为token。
+ client_id：必选，表示客户端ID。
+ redirect_uri：可选，表示重定向URI。
+ scope：可选，表示权限范围。
+ state：推荐，一个被客户端所使用的不透明的值，用来维持request和callback的状态。认证服务器将在重定向用户代理（User Agent）到客户端（Client）时带上这个值。这个参数**应当**在防止跨站点请求伪造时被使用。

举例：

```
    GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com
```

C中，认证服务器回应客户端的URI包括：

+ access_token：必选，表示访问令牌。
+ token_type：必选，大小写不敏感，表示令牌类型。
+ expires_in：过期时间，单位秒。若省略该参数，则必须以其他方式设置过期时间。
+ scope：表示权限范围，若与客户端申请的一致可省略。
+ state：如果客户端的请求中包括这个参数，认证服务器的回应也会相同的值。

举例：

```
     HTTP/1.1 302 Found
     Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
               &state=xyz&token_type=example&expires_in=3600
```

其中，HTTP-header中的Location中包含了重定向的网址，Hash部分包含了令牌，将在E步骤中被解析。

## 八、密码模式

密码模式（Resource Owner Password Credentials Grant）中，用户会直接向客户端提供自己的用户名和密码，客户端使用这些信息向“服务提供商”索要授权。

这种模式的安全性很低，必须建立在用户对于客户端高度信任的前提下。

整个过程中，客户端不得保存用户的密码。

![](http://image.beekka.com/blog/2014/bg2014051206.png)

```
(A) 用户向客户端提供用户名和密码。

(B) 客户端将用户名和密码发给认证服务器来请求令牌。

(C) 认证服务器确认后，向客户端提供访问令牌。
```

参数：

B步骤中，客户端发出的HTTP请求包含：

+ grant_type：必选，表示授权类型，此模式固定为password。
+ username：必选，用户名。
+ password：必选，密码。
+ scope：可选，表示权限范围。

举例：

```
     POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=password&username=johndoe&password=A3ddj3w
```

C中，认证服务器向客户端发送访问令牌，各参数含义与《授权码模式》相同。

举例：

```
     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }
```

## 九、客户端模式

客户端模式（Client Credentials Grant）指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。

![](http://image.beekka.com/blog/2014/bg2014051207.png)

步骤：

```
(A) 客户端向认证服务器进行身份认证，并要求一个访问令牌。

(B) 认证服务器认证无误后，向客户端提供访问令牌。
```

A中，客户端发出的HTTP请求包含：

+ grant_type：必选，表示授权类型，此模式固定为clientcredientials。
+ scope：可选，表示权限范围。

举例：

```
     POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=client_credentials

```

认证服务器以某种形式，验证客户端身份。

B中，认证服务器向客户端发送访问令牌。各参数含义与《授权码模式》相同。

举例：

```
     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "example_parameter":"example_value"
     }
```

## 十、更新令牌

如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。

客户端发出更新令牌的HTTP请求，包含：

+ grant_type：表示使用的授权模式，此处的值固定为"refreshtoken"，必选项。
+ refresh_token：表示早前收到的更新令牌，必选项。
+ scope：表示申请的授权范围，不可以超出上一次申请的范围，如果省略该参数，则表示与上一次一致。

举例：

```
     POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
```

## 十一、总结

总体而言OAuth2.0其实就是一个关于授权协议的规范化，对比以上几种模式：

`授权码模式`和`简化模式`都采用了User-Agent来作为请求的中转，而另外两种模式直接由客户端和认证服务器沟通。

从总体流程来看，授权码模式（authorization code）是其中功能最完整，流程最严密的授权模式，安全系数较高。






