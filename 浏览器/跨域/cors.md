# CORS

`@time 2020/05/29`

**跨域资源共享CORS**浏览器提供的针对**XMLHTTPRequest/Fetch**跨域问题的统一方案, 其使用**Origin**header来进行跨域资源访问的验证

这篇文章不会介绍关于**Fetch**的**CORS**(因为我对fetch了解的不够多 TODO)

### 应用场景

- **XMLHTTPRequest/Fetch跨域**

- **使用canvas导出外域的图片base64**

### API

**CORS**对前端来说不需要额外的设置(除了设置`xhr.witchCredentials`外), 因为很多请求头都由用户代理(浏览器)自动完成。相对的服务端需要返回一些配置的响应头, 由浏览器对其进行校验

**CORS**分为2种情况
- **简单请求**

- **需预检的请求**

**需预检请求**相较于**简单请求**会预先发送一次请求方法为**OPTIONS**的http请求(tips: 这个请求在`Chrome network`中是看不到的，原因是[chrome改变CORS实现](https://segmentfault.com/a/1190000021809808), 只能使用抓包工具来调试)

有一篇文章提到修改[out-of-blink-cors](chrome://flags/#out-of-blink-cors)为`disabled`可以查看到**OPTIONS**请求，这个目前已失效

#### 简单请求

全部满足以下几种条件则于**简单请求**

1.  使用以下几种请求方法
  - `GET`
  - `POST`
  - `HEAD`

2.  允许脚本设置的请求头为以下几种
  - `Content-Type`
  - `Accept`
  - `Accept-Language`
  - `Content-Language`
  - `DPR`
  - `Downlink`
  - `Save-Data`
  - `Viewport-Width`
  - `Width`

除了`Content-Type`其他的都没什么太大用,不用care

3.  `Content-Type`只能为以下几种value
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`

4.  请求中的`XMLHttpRequestUpload`对象没有绑定任何监听事件, 不是很重要
```JavaScript
xhr.upload.onload = function() {

}
```

##### 简单请求的服务端配置

- **Access-Control-Allow-Origin**

`Access-Control-Allow-Origin`对应了`Origin` 只有`Origin`的值满足`Access-Control-Allow-Origin`的情况下，**CORS**请求才会被允许

`Access-Control-Allow-Origin`的值有2种类型
  - **origin(portocal://domain:port)**
  
  此时只有
  ```JavaScript
  Origin === Access-Control-Allow-Origin
  ```
  请求被允许

  - **'*'**

  对`Origin`无限制, 但是在发送`Cookie`的情况下如果设置为`*`则请求不会被允许

- **Access-Control-Expose-Headers**
  允许曝光的响应头, 限制了`xhr.getAllResponseHeaders()`和`xhr.getResponseHeader(name)`的返回值

  默认情况下只有`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`这几个字段可以被返回, 其他字段需要被设置在`Access-Control-Expose-Headers`中才能够被获得
  ```JavaScript
  Access-Control-Allow-Origin: '*'
  Access-Control-Expose-Headers: 'Access-Control-Allow-Origin' 
  
  xhr.getResponseHeader('Access-Control-Allow-Origin'); // *
  xhr.getResponseHeader('Date'); // null
  ```
  上面提到的内容都是在**CORS**请求中, 在正常的xhr请求中`xhr.getAllResponseHeaders()`和`xhr.getResponseHeader(name)`可以获得的字段不只是上面提到的那几个

- **Access-Control-Allow-Credentials**
  默认情况下**CORS**是不发送`Cookie`的，如果需要发送`Cookie`需要客户端设置`xhr.withCredentials = true`同时服务端需要配置`Access-Control-Allow-Credentials`为`true`
  ```JavaScript
  // client
  xhr.withCredentials = true;

  // server(node)
  res.setHeader('Access-Control-Allow-Credentials', true);
  ```

  同时`Access-Control-Allow-Origin`不能为`*`

  ⚠️  发送的`Cookie`是这个**CORS**域名的`Cookie`, 不是主站的`Cookie`, 同理对于`Set-Cookie`也是一样的
  ```JavaScript
  // a.xx.com
  ajax.get(
    'https://b.xx.com/cors/userInfo' 
  );
  // 发送/更改的都是b.xx.com这个域名下的cookie，不影响主站a.xx.com的cookie
  ```

#### 需预检请求

**需预检请求** = **预检请求** + **简单请求**

##### 预检请求

遇见请求中服务端需要进行如下配置

- **Access-Control-Allow-Origin**

同上

- **Access-Control-Allow-Credentails**

同上

- **Access-Control-Allow-Methods**

设置**预检请求**允许的请求方法
```JavaScript
res.setHeader('Access-Control-Allow-Methods', 'GET, PUT, POST');
// 其他方法不被允许
```

- **Access-Control-Allow-Headers**

设置**预检请求**允许的请求头字段, 前面提到过非那几个字段外的请求会触发**预检请求**, 这个时候需要配置`Access-Control-Allow-Headers`来支持这个字段，否则请求会失败

```javaScript
// client
xhr.setRequestHeader('token', 1231231231);

// server
res.setHeader('Access-Control-Allow-Headers', 'device-id, token');
```

- **Access-Control-Max-Age**

**预检请求**的缓存时间, 以秒(s)为单位, 在这个时间内不需要再次发送**预检请求**
```JavaScript
// server
res.setHeader('Access-Control-Max-Age', 10);

// 第一次打开后的10s内都不会发送预检请求，使用上次的结果进行校验
```
