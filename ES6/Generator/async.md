# Async(Generator)

`@time 2020/06/06`

```JavaScript
const api = {
  "login": "github",
  "id": 9919,
};

function getAPI() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(api);
    }, 1000);
  });
}

function* readFile(filePath) {
  const api = yield getAPI();
  console.log(api, 'api');
  return api;
}

const controller = readFile();
controller.next()
  .value
  .then(res => controller.next(res));
```

用`Generator`编写异步代码看上去非常清晰, 但是如何控制其执行就变得很重要, 为每一个`Genertor`函数都编写一个控制器显然是不合理的, 我们需要找到一种通用的方式让`Generator`能够自动执行, 即: **yield 后面的表达式具有规律**

比如`Promise`或者`Thunk`函数

### yield 后面是一个 Promise

```JavaScript
const gc /* generatorController */ = function generatorController(generator) {
  return new Promise((resolve, reject) => {
    const iterator = generator();

    function onFulfill(data) {
      let nextIteratorObj = null;
      try {
        nextIteratorObj = iterator.next(data);
      } catch(err) {
        return reject(err);
      }
      
      handlNextIteratorObj(nextIteratorObj);
    }

    function onReject(reason) {
      let nextIteratorObj = null;
      try {
        nextIteratorObj = iterator.throw(reason);
      } catch(err) {
        return reject(err);
      }

      handlNextIteratorObj(nextIteratorObj);
    }

    function handlNextIteratorObj(nextIteratorObj) {
      if (nextIteratorObj.done) {
        resolve(nextIteratorObj.value);
        return;
      }

      nextIteratorObj.value.then(
        onFulfill,
        onReject,
      );
    }

    onFulfill();
  });
}

// ⚠️  封装gc的时候不要使用 while 循环，要使用函数递归的形式，因为then内的callback是异步的形式使用while会死循环

// test
const api = {
  "login": "github",
  "id": 9919,
};

function getAPI() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(api);
    }, 1000);
  });
}

function* readFile(filePath) {
  const api = yield getAPI();
  console.log(api, 'api');
  return api;
}

gc(readFile).then(res => console.log(res, 'res'));
```

### yield 后面是一个 Thunk函数 

关于什么是函数以及其诞生原因请看这里[什么是Thunk函数?](https://es6.ruanyifeng.com/#docs/generator-async#Thunk-%E5%87%BD%E6%95%B0), 挺有意思的

形如👇形式的函数都是`Thunk`函数
```JavaScript
thunk(argv)(callback)
```
因为`thunk`函数要求最后传入一个回调函数, 我们可以利用这点来完成`generator`执行器

```JavaScript
// thunk generator控制器
function controller(generator) {
  return new Promise((resolve, reject) => {
    const iterator = generator();

    function nextIteratorHandler(nextIterator) { // eslint-disable-line
      if (nextIterator.done) return resolve(nextIterator.value);

      nextIterator.value(handler); // eslint-disable-line
    }

    function handler(err, data) { // eslint-disable-line
      let nextIterator = null;
      try {
        if (err) {
          nextIterator = iterator.throw(err);
        } else {
          nextIterator = iterator.next(data);
        }
      } catch (e) {
        return reject(e);
      }

      nextIteratorHandler(nextIterator);
    }

    handler(null);
  });
}

// test
const api = {
  login: 'github',
  id: 9919,
};

function getAPI(callback) {
  setTimeout(() => {
    callback(null, api);
  }, 1000);
}

function thunkify(func) {
  return function thunk(...argvs) {
    const context = this;

    return function thunkFunc(callback) {
      return func.call(context, ...argvs, callback);
    };
  };
}

function* readFile() {
  const api = yield thunkify(getAPI)();
  console.log(api, 'api');
  return api;
}

controller(readFile).then((res) => console.log('res', res))
  .catch((e) => console.log(e, 'reject'));
```

可以看到`thunk`函数和`promise`自动执行`generator`函数其结果都大同小异, 只是找到规律递归就好

### 额外注意的是
封装`thunkify`函数的时候需要加锁控制`func`的执行次数, 因为其会影响控制器执行的速度, 举个🌰
```JavaScript
// 上面的getApi方法中不小心执行了2次callback
function getAPI(api, callback) {
  setTimeout(() => {
    callback(null, api);
    callback(null, api);
  }, 1000);
}

function* readFile() {
  const api1 = 1;
  const api2 = 2;
  const ret1 = yield thunkify(getAPI)(api1);
  const ret2 = yield thunkify(getAPI)(api2);
  console.log(ret1, 'ret1'); // 这里应该是1, 输出1
  console.log(ret2, 'ret2'); // 这里应该是2, 输出1 error ⚠️
}

controller(readFile).then((res) => console.log('res', res))
  .catch((e) => console.log(e, 'reject'));
```

所以我们在实现`thunkify`的时候要加锁, 让`func`只执行1次[tj大神的实现](https://github.com/tj/node-thunkify/blob/master/index.js)

`Promise`由于其天生的设定**状态不可被更改**所以没有这个问题, 2次`resolve`也不会导致`onFulfill`执行2次

### CO实现
[tj-co](https://github.com/tj/co/blob/master/index.js)
