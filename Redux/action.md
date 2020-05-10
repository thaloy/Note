# Action

@time 2020/05/10

**Redux**中使用**Action**来表述用户的行为,通过**Action**能让代码更加清晰,通过其属性**Type**,能够让程序开发者更好的了解程序具有的功能!

### Action的格式

**Action**是一个用来描述程序功能的**Plain Object**, 一般都是采用基于**Flux**架构的Action, 其能够更好的和社区内的middleware配合使用

- **type**

行为的名称, 比如**ADD_TODO**
```JavaScript
const addTodo = { type: 'ADD_TODO' };
```

- **payload**

当前行为附带的内容
```JavaScript
const addTodo = { type: 'ADD_TODO', payload: 'study' };
```

### Action的优化

在我认真看了一遍[减少样板代码](https://www.redux.org.cn/docs/recipes/ReducingBoilerplate.html)前, 我对一些概念并不理解, 比如**Action Creator**
```JavaScript
const addTodo = { type: 'ADD_TOTO', payload: 'study' };

// Action Creator
function addTodo(payload) {
  return {
    type: 'ADD_TODO',
    payload,
  };
}
```

- **Type单独成为常量文件**

1.  **Type**单独成为根据业务划分的常量文件, 可以代替一部分文档的工作, 开发者通过Types和Types的PR可以清楚的直到业务已有/新增的功能, 防止功能的重复
2.  **import**时如果拼写错误会得到undefined, 这样**Redux**能帮助我们更快的发现错误, 否则, 我们需要通过对比状态的变化和预期的变化是否一致来检测异常

tips: 当然你的Type要定义的足够有语义化

- **Action Creators**

前面提到的**Action Creator**(addTodo)太简单了, 以至于很难通过这个例子来找到**Action Creators**的优势

我是通过阅读[redux-actions](https://github.com/redux-utilities/redux-actions)找到其优势的

其优势在于: **增强Action, 减少重复代码**

e.g.
如果我们**addTodo**的内容是从一个固定的上下文中得到的
有一个合法的**todo**内容列表, **addTodo**只能从中选择, 这个时候就能体现出**Action Creators**的优势了
```JavaScript
const todoList = [
  '吃饭',
  '睡觉',
  '打豆豆',
];

// 使用了Action Creators的版本 -- v1
const addTodo = createAction(types.ADD_TODO, (index) => todoList[index]);
dispatch(addTodo, 0);
dispatch(addTodo, 1);
dispatch(addTodo, 2);

// 未使用Action Creators的版本 -- v2
function getTodo(index) {
  return todoList[index]);
}
dispatch({ types.ADD_TODO, payload: getTodo(0) });
dispatch({ types.ADD_TODO, payload: getTodo(1) });
dispatch({ types.ADD_TODO, payload: getTodo(2) });
```

从代码简洁性上来看, v1 > v2, 同时如果这个`disaptch(addTodo)`被触发在不同的上下文环境内, 还要定义多个`getTodo`方法, `todoList`也需要引入多次

### 自定义Action

前面提到的**Action**都是原子级别的, 很多时候我们会将**Action**组合起来使用, 以一个关注于请求状态的xhr为例子
```JavaScript
function requestPageContainer() {
  dispatch(actions.startRequest);

  request(url, method)
    .then(
      () => {
        dispatch(actions.successRequest);
      },
      () => {
        dispatch(actions.failRequest);
      }
    );
}

requestPageContainer();
```

由上面的代码就可以直到, 很大概率这是个可以被复用的**actionFlow**, 我们可以开发一个工具方法来完成这个事情
```JavaScript
utils.requestPageContainer(...argvs);
```

换一种理解方式的话, 其实这个**actionFlow**也是一个**Action**, 我称其为**复合Action**

**Redux**通过[redux-thunk](https://github.com/reduxjs/redux-thunk)提供了对**复合Action**的支持, 我们可以借由**redux-thunk**的功能, 自定义一个**actionFlow**
```JavaScript
function requestPageContainer() {
  dispatch(actions.startRequest);

  request(url, method)
    .then(
      () => {
        dispatch(actions.successRequest);
      },
      () => {
        dispatch(actions.failRequest);
      }
    );
}

dispatch(requestPageContainer);
```

很多网上的资源描述**redux-thunk**为处理异步，在我看来这有点`以偏概全`了

不仅如此, 设置是将**redux-thunk**和**redux-promise**做对比, 在我想来这更是个错误, 因为适用于不同的场景(怎么做比较), 甚至于我不觉得**redux-promise**是一个合理的中间件, 后面**redux-thunk**, **redux-saga**和**redux-promise**对比中会给出我这样认为的原因

### 同时触发的Action

e.g.
埋点

事实上, 我超级讨厌埋点这个事情
为什么呢？en...
```JavaScript
function click() {
  ...

  log(...);
}
```
很讨厌这样的代码，因为`click`的核心业务逻辑已经完成, 当给出数据需求或者增加对核心功能无影响的附加功能时, 不希望对核心`click`进行更改, 万一影响核心逻辑了呢?
比如`log`函数报错了, 你又凑巧将它放到了第一行

一般会用这2种方法优化上面的代码
```JavaScript
// v1
@logger(...)
click() {
  ...
}

// v2
function click() {
  ...

  observer.notify('click');
}
```
个人比较推荐v1，不过问题是`Decorator`这个提案的API被修改了，目前处于征求意见阶段，后续会改变，并且其只能修饰方法(非函数)

在**Redux**中有一种更优雅的方式处理这种问题, 因为**Redux**是通过**Action**映射逻辑的, 所以一种思路是采用v2的方式去监听**Action**，然后派发定义好的事件
```JavaScript
function log() {
  ...
}

function click() {
  ...
}

dispatch({ type: 'CLICK' });
// 触发
// log
// click
```
借由**Redux**优秀的middleware设计，将observer的逻辑通过middleware隐藏做到了逻辑的显示解耦

[redux-saga](https://github.com/redux-saga/redux-saga)就是一个类似的中间件，它将上述的行为理解为**Effect**(副作用), 通过不同的saga(线程)来完成多个**Effect**的'并发'

**redux-saga**的源码我只阅读了一点
```JavaScript
// 以下代码来自saga github, 用来做一下解释
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// createSagaMiddleware方法通过stdChannel方法创建了一个channel对象, 这个对象提供了订阅/发布effect的逻辑
/*
 {
    put, // 通过内置的调度器执行action(effect)
    close, // TODO
    take, // 订阅action(effect)
 }
*/
const sagaMiddleware = createSagaMiddleware()
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
// run方法会将mySaga执行得到对应的迭代器对象，通过这个迭代器对象将订阅action/effect(最终会执行channel.take)
sagaMiddleware.run(mySaga)

// sagas
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

function* mySaga() {
  // takeEvery方法会将'USER_FETCH_REQUESTED'和fetchUser变成saga内置的Effect结构, 最终被订阅由redux-saga统一管理
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

export default mySaga;
```

### thunk, saga, promise的对比

- **redux-promsie**
先说说**redux-promise**吧，我是认为这个东西比较鸡肋
```JavaScript
import isPromise from 'is-promise';
import { isFSA } from 'flux-standard-action';

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action) ? action.then(dispatch) : next(action);
    }

    return isPromise(action.payload)
      ? action.payload
          .then(result => dispatch({ ...action, payload: result }))
          .catch(error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          })
      : next(action);
  };
}
```

可以看到其对异常的处理是通过error来标识的，这就造成了我们要在reducer里面进行判断
```JavaScript
if (action.error) {
  ...
} else {
  ...
}
```
前面提到过**Action**应该是原子级别的，这样的代码破坏了**Action**的原子性，这不就是在reducer里面去**自定义Action**了, 同时这可能会引导开发者写出这样的代码
```JavaScript
if (action.error) {
  ... 这里做了一个和已经定义过的action一样的行为
} else {
  ... 
}
```

同时还有一个很重要的问题是,一般我们会用combineReducer来组合reducer函数
```JavaScript
const reducer = combineReducer({
  componentA,
  componentB,
});

// componentB
<>
  <B />
  { isError && <Error /> }
</>

// 如果使用redux-promise的话这个isError在componentA(reducer)中是获得不到的，如果想触发B的isError的话, 需要
// 通过ComponentB内引入componentA(reducer)内的状态

<>
  <B />
  { isError || otherError && <Error /> }
</>
```

en... 基于上述的原因我这里认为其比较鸡肋, 更别说处理异步了, 可能就只能处理一下payload是promise(fulfilled)

- **redux-thunk**

redux-thunk用来定制Action是比较不错的，不过我还是去实现一个自己的redux-thunk来保证action结构的统一
(最初我就比较迷惑，不是说action 是一个 plain object么，怎么还能传函数呢... 😂)

```JavaScript
{
  type: 'REQUEST',
  payload: () => fetch(...)
}
```

redux-thunk也有一点小问题, 就是前面提到的**同时触发的Action**, 使用thunk实现的话，总归要将附加逻辑内联进核心逻辑中(无论是在action内实现还是在调用action的时候), 或者在thunk内使用v2的方式.

- **redux-saga**

saga是一个完美的可以同时fix **自定义Action**和 **同时触发Action**的middleware

其唯一问题就是(鸡蛋里面挑骨头)
- API多
- 代码多
- 学习成本高(很多概念都是相关的，缺少独立性，不能渐进式的学习)

之前看到过一篇文章(文章我找不到了, 哎，当时没有添加书签的习惯)提到了thunk和saga对异步'竞态'的处理(顺序执行多个并发请求的回调)

其实这个并不是middleware提供的功能，而是我们定义action时增加的.

按照我的理解，我们可以实现`saga`提供的`take*`函数去封装action就好了呀!
