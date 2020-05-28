# MessageChannel

`MessageChannel`是为了在 2 个`worker`间持续通信而创建的`API`

其会创建一个`数据通道`, 该`数据通道`的 2 个`端口`可以借由其进行双向通信

### API

```JavaScript
const { port1, port2 } = new MessageChannel();
port1.onmessage = function() { console.log('port1'); } // 声明port2数据的响应
port2.onmessage = function() { console.log('port2'); } // 声明port1数据的响应
port1.postMessage('port1'); // port1发送数据给port2
port2.postMessage('port2'); // port2发送数据给port1
```

### ⚠️

- 响应`message`的回调函数在`Macrotask`队列中，异步执行，并且其优先级高于`setTimeout`

```JavaScript
setTimeout(() => console.log('timeout'), 0);

const { port1, port2 } = new MessageChannel();

port2.onmessage = function(event) {
  console.log('message');
};
port1.postMessage('message');

// message
// timeout
```

可能有人理解为是`setTimeout`存在最小延迟时间造成的, 但是实际上`setTimeout`和`setInterval`只有嵌套层级超过`n`(这个值在不同的浏览器中表现不同)时才会存在这个最小延迟时间(其原因是浏览器的批处理)

- 消息通道存在`队首阻塞`

`队首阻塞`是我参照`tcp协议`而理解的, 真正的消息通道情况需要阅读源码(目前没有阅读`v8`源码的能力)

```JavaScript
setTimeout(() => console.log('timeout'), 0);

const { port1, port2 } = new MessageChannel();

port2.onmessage = function(event) {
  console.log('message');
};

for (let i = 0; i < 10; i += 1) {
  port1.postMessage('message');
}

// message
// timeout
// message * 9

// 或者

// message * 10
// timeout
```

我理解的造成 2 种输出顺序的原因可能是

1. `setTimeout`进程间通信造成的时间延迟

2. 消息通道的队首阻塞

由于`callback(timeout)`时间延迟的大小, 导致其可能在第一次`callback(postMessage)`后执行或者在全部`callback(postMessage)`后执行

---

实际上上面的解释是我能想到的一种比较合理的解释了，但是也存在一些无法解释的问题.

e.g.为什么`callback(timeout)`只在全部/第一次`callback(postMessage)`后执行, 而不会出现在第 n 次`callback(postMessage)`后

这个要等有能力阅读`v8`源码才能够知道, 本质原因就是`v8`是如何处理`EventLoop`中各种不同异步队列的`竞态`问题(`dom event`, `timer`, `messageChannel`)

e.g. 在 1000ms 后同时触发点击事件和`setTimeout`那么其执行顺序是什么样的? 这是应当存在但是无法验证的场景, 浏览器是如何处理这样竞态关系的?

### 其他 hack 用法

- **deepClone**

因为`MessageChannel`多数应用在多个 worker, window 之间, 所以其一定应该具有 copy 数据的能力, 利用这个特点我们可以完成**deepClone**

通过`MessageChannel`实现`deepClone`可以有效的处理`JSON.parse(JSON.stringify(obj)))`的局限性

- `undefined`会被忽略
- `正则/error对象`会被序列化成`{}`
- `NaN/Infinity/-Infinity`会被序列化成`null`
- `Date对象`会被序列化为`string`
- 无法处理循环引用自身的问题

不足的是

- 无法处理`function`

```javascript
function deepClone(obj) {
  return new Promise(resolve => {
    const { port1, port2 } = new MessageChannel();

    port2.onmessage = function(event) {
      resolve(event.data);
    };
    port1.postMessage(obj);
  });
}

var obj = {a: null, b: undefined, c: new Date(), d: /\d+/, e: NaN, f: Infinity, g: new Error(1)};
obj.h = obj;

deepClone(obj)
.then(res => {
  console.log(res === obj, res);
});
console.log(JSON.parse(JSON.stringify(obj)));
```

- **代替setTimeout(() => {}, 0)**

Vue中使用`MessageChannel`来做`nextTick`的降级处理, 最后会被降级为`setTimeout(() => {}, 0)`

这是因为`setTimeout`可能会触发最小延迟时间, 并不一定是`setTimeou(() => {}, 0)`可能是`setTimeout(() => {}, 4)`

同时经过我的测试`MessageChannel`会比`setTimout(() => {}, 0)`更接近0ms
```JavaScript
// test channel
const { port1, port2 } = new MessageChannel();

port2.onmessage = function(event) {
  console.timeEnd('channel');
};

console.time('channel');
port1.postMessage('message');

// test timeout
console.time('timeout');
setTimeout(() => {
  console.timeEnd('timeout');
}, 0);
```
### 其他
`window.postMessage`和`worker.postMessage`应该都是借助与`MessageChannel`来实现的
