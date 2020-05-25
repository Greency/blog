- #### 1. 定义

XSS全称是Cross Site Scripting，指黑客往HTML文件中或者DOM中注入恶意脚本，从而在用户浏览页面时利用注入的恶意脚本对用户实施攻击的一种手段。

- #### 2. 恶意脚本影响的范围

1. 窃取Cookie信息：恶意JS脚本通过document.cookie获取Cookie信息，然后通过使用了CORS的请求将数据发送给恶意的服务器。
2. 监听用户行为：使用addEventListener接口监听用户的输入。
3. 修改DOM：伪造假的登录窗口。
4. 在页面内生成浮窗广告。

- #### 3. 三种XSS攻击

- ##### 1. 储存型XSS攻击

![image](https://greency.github.io/images-site/youdaoyunbiji/33.png)

1. 利用漏洞将恶意JS代码提交到数据库中。
2. 用户请求包含了恶意JS代码的页面。
3. 当用户浏览该页面时，恶意JS代码就会将用户的Cookie信息等数据上传到恶意的服务器。

- ##### 2. 反射型XSS攻击

在一个反射型XSS攻击中，恶意JS代码属于用户发送给网站请求中的一部分，随后网站又把恶意JS代码返回给用户。

服务端代码如下：
```
var express = require('express');
var router = express.Router();


/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express',xss:req.query.xss });
});

module.exports = router;
```

前端代码如下：

```
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel='stylesheet' href='/stylesheets/style.css' />
</head>
<body>
  <h1><%= title %></h1>
  <p>Welcome to <%= title %></p>
  <div>
      <%- xss %>
  </div>
</body>
</html>
```

当在浏览器输入如下地址时，结果如下：

```
http://localhost:3000/?xss=<script>alert('你被xss攻击了')</script>
```

![image](https://greency.github.io/images-site/youdaoyunbiji/34.png)

注意：Web服务器不会储存反射型XSS攻击的恶意脚本，这是和储存型XSSS攻击不同的地方。

==注意：在现实生活中，黑客经常会通过 QQ 群或者邮件等渠道诱导用户去点击这些恶意链接，所以对于一些链接我们一定要慎之又慎。==

- ##### 3. 基于DOM的XSS攻击

基于DOM的XSS攻击是不牵涉到页面Web服务器的。具体来讲，黑客通过各种手段将恶意脚本注入用户的页面中，比如通过网络劫持在页面传输过程中修改HTML页面的内容，这种劫持类型很多，有通过Wifi路由劫持的，有通过本地恶意软件劫持的，它们的共同点是在Web资源传输过程或者用户使用页面的过程中修改Web页面的数据。

- #### 4. 如何防止XSS攻击

- ##### 1. 服务器对输入的数据进行过滤/转码

```
code:<script>alert('你被xss攻击了')</script>

//过滤后
code:

//转义后：
code:&lt;script&gt;alert(&#39;你被xss攻击了&#39;)&lt;/script&gt;
```

- ##### 2. 充分利用CSP

> CSP有如下功能：

1. 限制加载其他域下的资源文件，这样即使黑客插入了一个JS文件，这个JS文件也无法被加载。
2. 禁止向第三方域提交数据，这样用户数据也不会外泄。
3. 禁止执行内联脚本和未授权的脚本。
4. 提供了上报机制，可以帮助我们尽快的发现有哪些XSS攻击。

- ##### 3. 使用HttpOnly属性

设置了HttpOnly属性的Cookie无法被document.cookie读取。

可以通过HttpOnly属性来保护我们的cookie安全。

将Cookie设置为HttpOnly标志，HttpOnly是服务器通过HTTP响应头来设置的：

```
set-cookie: NID=189=M8q2FtWbsR8RlcldPVt7qkrqR38LmFY9jUxkKo3-4Bi6Qu_ocNOat7nkYZUTzolHjFnwBw0izgsATSI7TZyiiiaV94qGh-BzEYsNVa7TZmjAYTxYTOM9L_-0CN9ipL6cXi8l6-z41asXtm2uEwcOC5oh9djkffOMhWqQrlnCtOI; expires=Sat, 18-Apr-2020 06:52:22 GMT; path=/; domain=.google.com; HttpOnly
```