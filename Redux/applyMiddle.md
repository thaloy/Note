# Redux-applymiddleware

applymiddleware(redux)本质上是store.dispatch方法的劫持, 数据在真正到达store(redux)前被不同中间件进行处理

👇的代码是我能想象到的一种劫持实现
```JavaScript
[applymiddleware1, applymiddleware2, applymiddleware3].reverse().forEach(middleware => {
  const dispatch = store.dispatch;

  store.dispatch = function(...argvs) {
    middleware({
      next: dispatch,
      argvs,
    });  
  };
});
```

由上面的code可以知道我们每一次劫持都会由一个next变量保存了其所劫持的store.dispatch

TODO
| applymiddleware1 |
| applymiddleware2 |
| applymiddleware3 |
|  store.dispatch  |

阅读/开发过redux中间件的话可以知道, redux中间件除了保存了next外, 还保存了被所有middleware最终劫持后的store.dispatch

即: redux的中间件系统提供了跳出当前任务流, 重新执行任务的机会, 这为定制任务(redux-thunk)提供了可能

TODO
        ------------------|
        |                 |
| applymiddleware1 |      |
| applymiddleware2 |      |
| applymiddleware3 | -----
|  store.dispatch  |

其巧妙的利用了对象和闭包, 看一下下面的demo

```JavaScript
function updateObjPropA(obj) {
  obj.a += 1; 
}

function getObjPropA(obj) {
  console.log(obj.a); 
}

function funcWrap(getObjPropA, updateObjPropA) {
  const obj = { a: 0 };  
  
  return [getObjPropA.bind(null, obj), updateObjPropA.bind(null, obj)];
}

const [get, update] = funcWrap(getObjPropA, updateObjPropA);
update();
update();
update();
get(); // 3

getObjectPropA  相当于中间件
obj.a           相当于最终劫持后的dispatch
update()        相当于复合中间件(一次次劫持dispatch的过程)
```

### middleware(redux)源码分析
```JavaScript
// src/compose.js
function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  // 复合中间件的核心逻辑
  // 与上面的revers().forEach()作用一致
  // 需要注意的是这里的funcs不是完成的中间件,在这之前会先对所有中间件传入{ getState, dispatch }得到funcs
  // funcs = middlewares.map(middleware => middleware({ getState, dispatch }));
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

```JavaScript
// src/applyMiddleware.js
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    // 在createStore的核心逻辑里面会做处理, 如果传入了applyMiddleware会在applyMiddleware
    // 方法内进行'实例'的过程

    // 这里和我的想法有出入
    // 如果是我的话, 我应该会在creatStore内实例化，然后在applyMiddleware内做劫持dispatch的处理
    // 即:
    /*
     * createStore(reducer, state, [...middlewares])
     */
    // 目前我没有理解到redux提供
    /*
     * createStore(reducer, state, applyMiddleware(...middlewares))
     */
    // 这样的好处是在什么地方
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    // 这个就是前面提到的redux为了跳出当前任务流,重新开始的那个闭包对象
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // 传入闭包对象的过程, 对中间件进行一次偏函数的处理
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 复合中间件的过程
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
### 总结
applyMiddleware(redux)核心逻辑很简单, 重点在于其能够打破向下传递的任务流, 以新的action重新开始任务(redux-thunk), 开发applyMiddleware(redux)需要注意定义好结束当前applyMiddleware的出口,否则容易出现死循环.
