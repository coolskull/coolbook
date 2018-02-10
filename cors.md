##CORS 原理和解决方案，包括前端解决方案和后端解决方案


### 什么是CORS？
  CORS (Cross-Origin Resource Sharing跨域资源共享)，依附于AJAX，通过添加HTTP Hearder部分字段请求与获取有权限访问的资源。
  CORS对开发者是透明的，因为浏览器会自动根据请求的情况（简单和复杂）做出不同的处理，CORS的关键是服务端的配置支持。


### 同源与跨域
为了保证用户信息的安全，所有的浏览器都遵循同源策略
(协议、域名、端口号完全相同时，才是同源)。

URL | 说明 |	是否允许通信
---- | ----| :---:
http://www.a.com/a.js<br/>http://www.a.com/b.js| 同一域名下| 允许
http://www.a.com/lab/a.js<br/>http://www.a.com/script/b.js | 同一域名下不同文件夹| 允许
http://www.a.com:8000/a.js<br/>http://www.a.com/b.js| 同一域名，不同端口 | 不允许
http://www.a.com/a.js<br/> https://www.a.com/b.js | 同一域名，不同协议| 不允许
http://www.a.com/a.js<br/>http://70.32.92.74/b.js| 域名和域名对应ip| 不允许
http://www.a.com/a.js<br/>http://script.a.com/b.js| 主域相同，子域不同| 不允许
http://www.a.com/a.js<br/>http://a.com/b.js| 同一域名，不同二级域名（同上）| 不允许(cookie这种<br/>情况下也不允许访问）
http://www.cnblogs.com/a.js<br/>http://www.a.com/b.js| 不同域名| 不允许

### CORS的是使用场景
1. 跨域调用XMLHttpRequest或Fetch API。<br/>
2. Web Fonts（CSS 中通过 @font-face 使用跨站字体资源）。<br/>
3. WebGL textures（纹理）。<br/>
4. canvas上使用drawImage绘制帧的图像/视频。<br/>
5. 样式表（用于CSSOM访问）。[CSSOM视图模式(CSSOM View Module)相关整理](http://www.zhangxinxu.com/wordpress/2011/09/cssom%E8%A7%86%E5%9B%BE%E6%A8%A1%E5%BC%8Fcssom-view-module%E7%9B%B8%E5%85%B3%E6%95%B4%E7%90%86%E4%B8%8E%E4%BB%8B%E7%BB%8D/)<br/>
6. Scripts (for unmuted exceptions).<br/>

### CORS的兼容性
  目前，所有浏览器基本都支持该功能，但是IE8、IE9浏览器支持不够好，所以要兼容ie低版本的时候还是要选择jsonp。
<img src="/imgs/caniuseCORS.png"/>

### CORS请求原理
<img src="/imgs/CORS2.png"/>


#### 如何判断是否是简单请求?
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

###### 只要同时满足以下两大条件，就属于简单请求:
* 请求方法是以下三种方法之一：HEAD,GET,POST
* HTTP的头信息不超出以下几种字段：
    * Accept
    * Accept-Language
    * Content-Language
    * Last-Event-ID
    * Content-Type(只限于三个值application/x-www-form-urlencoded、 multipart/form-data、text/plain)


### 如何分析ajax跨域错误
##### 一般思路：
1. 判断服务端Access-Control-Allow-Origin的设置和客户端的Origin是否一致。<br/>
2. 判断request method是否包含在服务端Access-Control-Allow-Methods列表里。<br/>
3. 判断accept的方法是否一致<br/>
<img src="/imgs/http_header.jpg"/><br/>

##### 简单请求跨域问题:
跨域服务端没有配置,Status Code 200，但是会控制台会报错
<img src="/imgs/服务端没有配置跨域.png"/>

##### 非简单请求跨域问题：
 参考：[ajax跨域，这应该是最全的解决方案了--ajax跨域的表现](https://segmentfault.com/a/1190000012469713#articleHeader5)

### nodejs后台配置（以koa2框架为例）
```js
app.use(async (ctx, next)=>{
    // 允许来自所有域名请求
    ctx.set("Access-Control-Allow-Origin", "*");
    // 这样就能只允许 http://localhost:8080 这个域名的请求了
    // ctx.set("Access-Control-Allow-Origin", "http://localhost:8080");

    // 设置所允许的HTTP请求方法
    ctx.set("Access-Control-Allow-Methods","OPTIONS, POST, DELETE");

    // 字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段.
    ctx.set("Access-Control-Allow-Headers", "x-requested-with, accept, origin, content-type");

    // 服务器收到请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应。
    // Content-Type表示具体请求中的媒体类型信息
    ctx.set("Content-Type", "application/json;charset=utf-8");

    // 该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。
    // 当设置成允许请求携带cookie时，需要保证"Access-Control-Allow-Origin"是服务器有的域名，而不能是"*";
    ctx.set("Access-Control-Allow-Credentials", true);

    // 该字段可选，用来指定本次预检请求的有效期，单位为秒。
    // 当请求方法是PUT或DELETE等特殊方法或者Content-Type字段的类型是application/json时，服务器会提前发送一次请求进行验证
    // 下面的的设置只本次验证的有效时间，即在该时间段内服务端可以不用进行验证
    // OPTIONS预检的优化: 非常有用，可以大幅优化请求次数
    ctx.set("Access-Control-Max-Age", 300);

    /*
    CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：
        Cache-Control、
        Content-Language、
        Content-Type、
        Expires、
        Last-Modified、
        Pragma。
    */
    // 需要获取其他字段时，使用Access-Control-Expose-Headers，
    // getResponseHeader('myData')可以返回我们所需的值
    ctx.set("Access-Control-Expose-Headers", "myData");
    await next();
});

```


### 配置CORS规则
###### nginx上的CORS配置
[Nginx通过CORS实现跨域](http://www.yunweipai.com/archives/9381.html)
###### OSS上的CORS配置
[跨域资源共享（CORS）](https://help.aliyun.com/document_detail/31928.html)
###### CDN上的CORS配置
[CDN支持cors（跨域）配置的步骤与注意事项](https://help.aliyun.com/knowledge_detail/40183.html)
<br/><br/>


### 其他跨域方案的比较
##### jsonp
jsonp解决跨域问题是一个比较古老的方案(实际中不推荐使用).
###### 实现原理
<img src="/imgs/jsonp.png"/>
###### 实现流程

1.客户端网页网页通过添加一个 `<script>` 元素，向服务器请求JSON数据，这种做法不受同源政策限制

```js
function jsonp(req){
    var script = document.createElement('script');
    var url = req.url + '?callback=' + req.callback.name;
    script.src = url;
    document.getElementsByTagName('head')[0].appendChild(script);
}
```
2.callback定义了一个函数名fn，而远程服务端通过调用指定的函数并传入参数来实现传递参数，将fn(response)传递回客户端
```js
var http = require('http');
var urllib = require('url');

var port = 8080;
var data = {'data':'world'};

http.createServer(function(req,res){
    var params = urllib.parse(req.url,true);
    if(params.query.callback){
        console.log(params.query.callback);
        //jsonp
        var str = params.query.callback + '(' + JSON.stringify(data) + ')';
        res.end(str);
    } else {
        res.end();
    }

}).listen(port,function(){
    console.log('jsonp server is on');
});
```
3.客户端接收到返回的js脚本，开始解析和执行fn(response)
```js
function hello(res){
    alert('hello ' + res.data);
}
jsonp({
    url : '',
    callback : hello
});
```


### 扩展笔记：
vuejs怎么实现跨域 vue-resource:Supports latest Firefox, Chrome, Safari, Opera and IE9+<br/>


[js中 [].slice 与 Array.prototype.slice 有什么区别?](https://www.zhihu.com/question/46724226)<br/>
两种调用的不同之处，就是this的指向不同。<br/>
在[].slice中，this指向的是该实例[]。而在Array.prototype.slice中，this指向的是Array.prototype.<br/>

[plainObject的作用](http://kodango.com/javascript-oop-type-system)<br/>


### 参考：
[ajax跨域，这应该是最全的解决方案了](https://segmentfault.com/a/1190000012469713)<br/>
[jsonp的原理与实现](https://segmentfault.com/a/1190000007665361)<br/>
[koa cors的实现](https://www.jianshu.com/p/5b3acded5182)<br/>


