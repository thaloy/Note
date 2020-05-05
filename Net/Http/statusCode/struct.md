# StatusCode

http状态码是响应报文的一部分,是Client和Server间的约定,Client根据Server提供的不同的状态码做不同类型的响应
e.g.
- 101
  浏览器进行协议升级

- 200
  由JavaScript处理响应

### 状态码的分类
状态码被分为5类
- [100~200)

响应Client的消息,Client还需要后续的处理
一般都是指Server要验证是否支持Client的请求
常用**101**

### 状态码的应用场景
- 101 Switching Protocol

webSocket,浏览器发送如下header
```JavaScript
Connection: upgrade
Upgrade: webSocket
```
服务端如果支持webSocket则返回101,后续浏览器和服务端进行webSocket通信
