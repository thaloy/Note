# Generator

`@time 2020/06/03`

`Generator`是一个遍历器的生成函数, 通过`Generator`我们可以获得其迭代器对象, 提供了统一的迭代机制

### 应用场景

1.  迭代器

`for of`循环就是利用迭代器完成的最常见应用

2.  状态机

可以利用`Generator`来做订单的状态管理(目前，笔者没有实际的应用)

3.  处理异步流程

由于其返回的是一个迭代器对象，通过迭代器对象可以在`Generator`函数体外对函数内的逻辑进行控制(暂停/恢复), 所以我们可以利用这个特点进行异步编程, 同时由于其`函数体内外的数据交互(next)`, `完善的异常处理(throw)`, 我们可以将其作为一个统一的异步解决方案

需要使用*TJ**大神完成的`CO`库合作进行异步开发, 可以很好的解决`callback hell`, 让异步流程更加清晰, 易维护

可以这样理解

使用`callback`完成异步操作，类似于你看一本没有目录结构的书, 下一章的内容只有在当前章节的末尾才能看到

而使用`Generator + CO`处理异步就是在编写目录, 这样对于后续迭代的过程中可以准确对某个'章节'内容进行修改, 不用从头读到当前内容

`redux-saga`是常见的应用`Generator`的项目

tips: 目前使用最多的异步编程方案是`Async/Await`, 本质上其就是`Generator+CO`的🍬, 因为`Generator`来处理异步简直太好用了, 所以`ECMA`将其定义为标准

4.  定制任务流

和3类似通过`Generator`来定制任务流, 比如App启动的时候要预先进行的一些用户登陆/验证等流程, 只要你可以编写合理的`Generator执行器`像`CO`一样

### API

```JavaScript
function* Generator() {
  yield 'status1';
  yield 'status2';
  yield 'status3';
}

const gen = Generator();
gen.next();
```

#### 声明一个Generator函数

声明一个`Generator`函数有多种方式(形式上)

```JavaScript
function* foo(x, y) { ··· }
// --------------------------//
function * foo(x, y) { ··· }
function *foo(x, y) { ··· }
function*foo(x, y) { ··· }
```

个人觉得第一种是最科学的声明`Generator`函数的方式, 这体现在`foo`是一个特殊的函数`function* (Generator)`

#### yield关键字

`yield`是`Generator`迭代器的标识, 遇到`yield`交出当前的执行权限, 执行其他逻辑, 等待下一次`next, throw, return`恢复执行权限

```JavaScript
function* Generator() {
  yield console.log(1);
  yield console.log(2);
  yield console.log(3);
}

const gen = Generator();
gen.next(); // 1 { value: undefined, done: false }
gen.next(); // 2 { value: undefined, done: false }
gen.next(); // 3 { value: undefined, done: false }
gen.next(); //   { value: undefined, done: true }
```

##### yield的优先级

`yield`的优先级很低

```JavaScript
function* add() {
  yield 1 + 2;
}

add().next(); // { value: 3, done: false }, 注意这里value是3, 不是1

function* add() {
  (yield 1) + 2;
}

add().next(); // { value: 1, done: false } 

function* logger() {
  console.log(123 + yield 456);
}

// 异常
// 由于yield优先级低，所以其被解析为123 + yield 456, 而 + 运算符不能正确识别yield关键字
// 所以需要提升yield表达式的优先级

function* logger() {
  console.log(123 + (yield 456));
}
```

#### Generator.prototype.next

`next`方法可以恢复迭代器的执行, 直到遇到下一个`yield`

1.  `next`方法的返回值为`{ value, done }`, `value`为遇到的`yield`后面的表达式的值, `done`表示了迭代器是否结束(return)

2.  `next`接受一个参数这个参数会替换`yield运算`

以上2点是`Generator`重要的函数体内外的数据交互

```JavaScript
function* logger() {

  console.log(1 + (yield /* lo.next(123) 在这里被暂停, 123被忽略 */ 1) /* yield1 */);

  console.log(2 + (yield /* lo.next(100) 在这里被暂停, 100替换了yield1 */ 2) /* yield2 */);

  console.log(3 + (yield /* lo.next(200) 在这里被暂停, 200替换了yield2 */ 3) /* yield 3 */);

  return 4; // lo.next(300) 在这里结束, 300替换了yield3
}

const lo = logger();

lo.next(123);  // { value: 1, done: false }
lo.next(100); // { value: 2, done: false } 输出101
lo.next(200); // { value: 3, done: false } 输出202
lo.next(300); // { value: 4, done: true } 输出303
```

可以发现`next`接受的参数和替换的`yield`运算是-1的关系, 第一个`next参数被忽略`, 第二个替换的是第一个的`yield`...

一种记忆方式是认为`next的参数替换的是`上一个`yield表达式`

我的理解方式是, `Generator`遇到`yield`会交出执行权限, `next`会恢复执行, 但是暂停点不在于`yield`而是`yield`后面的表达式。以上面`logger`为例子函数将被按照下面分解
```JavaScript
function* logger() {
  // 第一次next
  1;
  // 第二次next
  console.log(1 + (yield));
  2;
  // 第三次next
  console.log(2 + (yield));
  3;
  // 第四次next
  console.log(3 + (yield));
  
  return 4;
}
```
`next`参数替换的是在这次运行期间的`yiled`, 第一次没有**yield**所以被忽略, 第二次`next`替换的是第一个`yield运算`

#### Generator.prototype.throw

`throw`方法可以将函数体外的异常仍到`Generator`函数体内, 通过`throw`方法, 可以统一处理异常. 如果函数体内没有对异常进行捕获的话，异常将在控制迭代器迭代的地方被抛出`next, throw, return`

和`next`一样
1.  `throw`方法的返回值为`{ value, done }`, `value`为遇到的`yield`后面的表达式的值, `done`表示了迭代器是否结束(return)

2.  `throw`接受一个参数这个参数会替换`yield运算`

```JavaScript
function* logger() {
  try {
    yield;
    // 相当于
    // throw 1;
  } catch(err) {
    console.log('err', err);
  }
}

const lo = logger();

lo.next(); // { value: undefined, done: false }
lo.throw(1); // { value: undefined, done: true} 'err', 1
```

由于`throw`本质上是将`yield`替换为`throw argv`, 所以在执行`throw`之前一定要执行`next`, 否则如果第一次执行的是`throw`方法的话会直接抛出异常, 导致整个`Generator`迭代器迭代完成

```JavaScript
function* logger() {
  try {
    yield;
  } catch(err) {
    console.log(err, 'err'); // err, 'error'
  }
}

const lo = logger();
console.log(lo); // logger {<suspended>} --- 迭代器是暂停状态
lo.throw();
console.log(lo); // logger {<closed>} --- 迭代器是完成状态的

// 相当于
function* logger() {
  throw undefined; // 这个异常在generator函数中是无法被捕获的
  try {
    yield;
  } catch(err) {
    console.log(err, 'err'); // err, 'error'
  }
}

// 正确的处理方法是
lo.next();
lo.throw(1); // err, 1
```

如果异常没有在函数体内被捕获，则会在恢复`Generator`执行的地方抛出, 如果迭代器结束依然可以继续抛出异常, 但是需要在函数体外进行捕获

```JavaScript
function* logger() {
  throw 'error';
  yield;
}

const lo = logger();
try {
  lo.next();
} catch(err) {
  console.log(err, 'err'); // err, 'error'
}
```
```JavaScript
function* logger() {
  try {
    yield 1;
  } catch(err) {
    yield 2;
  }
}
const lo = logger();
lo.next();
lo.throw('函数内捕获');
lo.throw('函数外捕获'); // 这个异常在函数体内没有被捕获到
lo.throw('generator迭代器关闭，但是还是可以继续抛出错误');
```

在函数体内捕获和函数体外捕获的区别在于, 函数体内捕获不会改变迭代器的状态, 函数体外捕获由于异常最造成**迭代器状态提前完成**

```JavaScript
function* logger() {
  yield 1;
  yield 2;
}

function* logger1() {
  try {
    yield 1;
  } catch(err) {
    console.log(err, 'err');
  }
  yield 2;
}

const lo = logger();
const lo1 = logger1();

lo.next();
lo.throw(1);
console.log(lo) // <closed>

lo1.next();
lo1.throw(1);
console.log(lo1); // <suspended>
```

#### Generator.prototype.return

与`next`, `throw`一样, `return`是将`yield`替换成`return argv`
1.  `return`方法的返回值为`{ value, done }`, `value`为遇到的`yield`后面的表达式的值, `done`表示了迭代器是否结束(return)

2.  `return`接受一个参数这个参数会替换`yield运算`

这个API应该并没有很大用处, 暂未想到其应用场景

```JavaScript
function* logger() {
  try {
    yield console.log(1);
    yield console.log(2);
  } finally {
    yield console.log(3);
  }
}

const lo = logger();
lo.next(); // 1 { value: undefined, done: false }
lo.return(2); // 3 { value: undefined: done: false }
lo.next();  // { value: 2, done: true }
```

与`throw`一样，如果在执行`next`前调用`return`, 会导致迭代器提前结束(这是无法通过`try...finally`进行处理的)

```JavaScript
function* logger() {
  try {
    yield console.log(1);
    yield console.log(2);
  } finally {
    yield console.log(3);
  }
}

const lo = logger();
lo.return(2); // { value: 2, done: true }
lo // <closed>
```

#### yield*运算符

使用`yield*`运算符可以在`Generator`函数内迭代另外一个`Generator`迭代器

```JavaScript
function* foo() {
  yield 'a';
  yield 'b';
  yield 'c';
}

function* bar() {
  yield* foo();
  yield* 'd';
  yield* 'e';
}
[...bar()]; // [a, b, c, d, e, f];

// 等价于
function* bar() {
  yield 'a';
  yield 'b';
  yield 'c';
  yield* 'd';
  yield* 'e';
}
```

##### 任何具有iterator接口结构都可以被yield*迭代

```JavaScript
const foo = ['a', 'b', 'c'];

function* bar() {
  yield* foo;
  yield* 'd';
  yield* 'e';
}
[...bar()]; // [a, b, c, d, e, f];
```

实质上`yield*`不是对foo进行迭代，而是对`foo[Symbol.itertor]()`进行迭代, 一个有意思的地方是, 如果`foo`是一个迭代器对象，那么其`foo[Symbol.iterator]() === foo`

```JavaScript
function* foo() {
  yield 'a';
  yield 'b';
  yield 'c';
}

const fo = foo();
fo[Symbol.iterator]() === fo;
```

##### yield*的返回值

`yield`的返回值可以由`next`来决定, `yield*`的返回值由迭代器对象最终的value决定

```JavaScript
function* foo() {
  yield 'a';
  yield 'b';
  yield 'c';
  return 'foo';
}

function* bar() {
  const fooRet = yield* foo();
  console.log(fooRet, 'fooRet');
  yield* 'd';
  yield* 'e';
}
[...bar()]; // foo
```

#### 迭代器对象与this

`Generator`函数中的`this`和其他函数没什么区别，比较特殊的是其返回的迭代器对象，继承了`Generator.prototype`

##### 不能使用new实例化

`Generator`使用`new`实例会报错

##### 迭代器对象继承了Generator.prototype

因为返回的迭代器对象不是`this`, 所以如果想在`Generator`内给迭代器对象添加属性只能利用其继承了`Generator.prototype`这个特点

```JavaScript
function* gen() {
  this.a = 1;
}

const g = gen.call(gen.prototype);
g.next();
console.log(g.a);
```

值得一说的是上面的属性a并不会被其他迭代器共享, 逻辑类似于这样

```JavaScript
function Generator {

}

Generator.ptototype.next = function() {}
Generator.ptototype.throw = function() {}
Generator.ptototype.return = function() {}

const F = function() {};

F.prototype = Generator.prototype;
gen.prototype = new F();

function gen() {
}
```

通过类似于上面的逻辑实现了操作`gen.prototype`的安全性
