# Function

#### 什么是函数
函数用来表示一段逻辑.比如:
```JavaScript
function run() { ... }
```
表示了大脑神经元和四肢相互协作的过程.

#### 创建函数
在JavaScript中有3种可以创建函数的方式
-	**函数声明**
```JavaScript
function run() { ... } // 函数声明
```

- **函数表达式**
函数表达式分为匿名/命名函数表达式.
```JavaScript
var run = function() { ... } // 匿名函数表达式
var run = function peopleRun() { ... } // 命名函数表达式
```
命名函数的函数名(peopleRun)只允许在其函数内部访问,是一种代替arguments.callee实现

还有一种比较特殊的函数表达式--->箭头函数
```JavaScript
var run = () => { ... }
```
它具有如下可以简写的方式
```JavaScript
// 如果参数个数为1,可以省略()
var run = argv => { ... }
// 如果函数体内只有返回值,可以省略函数体
var run = argv => 'run'
// 如果返回值是对象,需要使用()包装一些,这是因为JS默认将{}解析为语法块
var run = argv => ({ run: 'run' })
```

- **实例化Function**
因为JavaScript中的函数本质上是可执行对象.所以可以通过实例化Function的方式创建一个函数.
```
var run = new Function(' ... ');
```

#### 执行期上下文
函数执行需要使用到一些变量..., 这些变量就来自执行期上下文.
正如字面意思,执行期上下文仅存在于函数的执行阶段,一旦函数执行完毕(除了[闭包](#闭包)),就会被JS的垃圾回收机制回收掉.
执行期上下文分为2个阶段
- **初始化阶段**
	1.	创建VariableObject,函数中索引变量的时候会优先在VariableObject中查找.
	```JavaScript
	// 伪代码
	VariableObject = {
		this: window,
		arguments: { callee: current function reference },
		// 如果是命名函数的话,并且其值是不可以被赋值操作更改的TODO	
		current function reference
	};
	```
	2.	链接[[Scope Chain]]	
	3. 	初始化变量声明
		- **ES5声明**
		如果是用var声明的变量将其挂载到VariableObject上,其值为undefined
		- **ES6声明**
		let/const/class声明的变量将其挂载到TDZ(暂时性死区内)(我认为let/const...是存在变量声明的,区别是挂载地点不一致,关于我为什么这么认为和TDZ的内容可以看另一篇文章TODO)
	4.	初始化形参,形参和实参统一,挂载到VariableObject上, arguments同步修改.	
	5.	初始化函数声明并且赋值为函数体. arguments同步修改.

上面3和5的过程就是常常提到的Hositing(变量提升),即我们可以在函数声明前执行这个函数.在变量声明前访问这个变量只不过得到的是undefined.

- **执行**
	-	变量的索引优先查找VariableObject. 如果VariableObject上不存在的话则会顺着[[Scope Chain]]递归着索引VariableObject直到直到这个值.
		类似于对象索引属性的过程,不过不同的是
		赋值的过程是直接在索引到的VariableObject上赋值,不是在当前执行期上下文的VariableObject上.
		```JavaScript
		console.log(window.name) // ''

		function executionContextFunc() {
			var privateName = name = 2;	
		}
		
		executionContextFunc();
		console.log(window.name); // 2;
		```

	-	如果是由命名函数表达式方式创建的函数, 是不能修改其current function reference变量的.	
		```JavaScript
		const executionContextFunc = function func() {
			func = 2;	

			console.log(func); // output = exectionContextFunc.toString();
	 	}

		executionContextFunc();
		// 严格模式下抛出错误.
		// 非严格模式下静默失败.
		```

	- 对变量的操作如何涉及到形参,那么arguments会同步更改.

#### 作用域和作用域链
- **作用域**
执行期上下文创建的VariableObject就是当前函数的作用域.

- **作用域链**
有的时候我们需要一些当前作用域不存在的变量,这个时候引擎会顺着作用域链索引到这个变量(类似于原型链)

JavaScript的函数作用域是基于**词法分析**的.
即函数的作用域上下文是在函数被创建的时候确定的,而不是在函数执行的时候.
可以这样理解,当前函数创建的时候会将创建这个函数的执行期上下文作为函数的一个属性.
```JavaScript
function outer() {
	const outerA = 'outerA';	
	const outerB = 'outerB';

	/*
	 * 此时outer的执行期上下文
	 * {
	 *	 variableObject: {
	 *			outerA: 'outerA',
	 *      outerB: 'outerB, 
	 *      this: window,
	 *      arguments: { callee: outer }
	 *   }
	 *	 [[Scope Chain]]: { Global(window) }
	 * } 
	 */ 
	const inner = function inner() {
	 	/*
	   * 此时inner的执行期上下文
	   * {
	   *	 variableObject: {
	   *      this: window,
		 *			inner,
	   *      arguments: { callee: inner }
	   *   }
	   *	 [[Scope Chain]]: { exectionContext(outer) }
	   * } 
	   */ 
		console.log(outerA);	
		console.log(outerB);
	}
	// 这个时候innter有一个属性[[Scopes]]指向了executionContext(outer)

	return inner();
}

const inner = outer();
inner(); // 可以访问到outerA,outerB

// 执行期上下文会在函数执行后由于没有被其他引用导致被JS垃圾回收机制回收
// 但是上面代码中outer的执行期上下文成为了inner的一个属性导致其并没有被回收.依旧可以被访问.
```
```JavaScript
function outer() {
	const sameVariableName = 'outer';

	return function() {
		console.log(sameVariableName);	
	}
}

const inner = outer();

function callInner() {
	const sameVariableName = 'call';

	inner();
}
// 输出outer
```

#### 函数的方法/属性
- **name**
函数的名字,如果是匿名函数与其被赋值的变量名一致.
```JavaScript
function a() {};
let b = function() {};
let c = function d() {};
[a, b, c].map(func => func.name); // ['a', 'b', 'd']
```

- **length**
形参的数量
```JavaScript
function sum(a, b, c, d) { return a + b + c + d }
sum.length // 4
```

- **caller**
当前函数是在那个函数内执行的.
```JavaScript
function outer() {
	inner();
}

function inner() {
	console.log(inner.caller); // outer.toString();
}

outer();
```
如果全局执行的话是null.
```JavaScript
[1, 2].forEach(function caller() { console.log(caller.caller) });
// 输出null,并没有输出forEach的源码..😢😁
```
由于其不安全的性质,在ES标准中被废除

- **call()/apply()**
👆提到了函数执行期上下文初始化过程有一个this.
call和apply是用来改变这个this的指向的.区别是
call将参数one by one的传入到函数中.
apply要将参数作为数组传入到参数中.
```JavaScript
const obj = { count: 1 };
function addCount(argv1, argv2) {
	this.count = this.count || 0;
	return this.count + argv1 + argv2;	
}
addCount.call(obj, 1, 2); // 4
addCount.apply(obj, [1, 2]); // 4

// ES6为我们提供了...运算符,所以我们也可以这样使用call
addCount.call(obj, ...[1, 2]);
```
有人和我讲过call比apply运行快...
快在不用遍历实参嘛? 😁😁😁😁

- **bind()**
bind函数的作用和call/apply一样都是改变this的指向.区别在于
-	call和apply是直接执行
-	bind函数返回一个函数
```JavaScript
同👆call和apply的例子
const newAddCount = addCount.bind(obj, 1, 2);
newAddCount(); // 4
```

bind函数还有一个额外的作用(偏函数)[#柯里化/偏函数/Thunk函数]
```JavaScript
const newAddCount = addCount.bind(this);
newAddCount(1, 2); // 4
```

#### 闭包
👆提到的将inner函数保存着outer函数执行期上下文的行为就是闭包.
很难讲清楚闭包的定义.
有人认为每一个函数的创建过程都是闭包.
有人认为只有本应该销毁的执行期上下文被意外保存下来的行为才是闭包.
闭包的定义并不重要,你需要知道的是.
-	**闭包是基于词法分析的语言的天然性质**
-	**闭包是如何产生的**

######闭包的优势
- **私有变量**
参见上文IIFE
- **缓存**
有的时候我们需要缓存一些值,就可以利用闭包来实现.比如: 缓存长列表
需求背景:我们需要根据用户输入来过滤列表.
优化方式:可以缓存用户最近几次输入的结果(一般都是1次,防止内存泄漏),如果用户的输入在缓存的几次内重复,则直接用缓存的值.
github上有关于此的实现[memoize-one](https://github.com/alexreardon/memoize-one)
这里给出其大致实现
```JavaScript
function equalsArgvsDefault(prevArgvs, nextArgvs) {
	const prevLen = prevArgvs.length - 1;
	const nextLen = nextArgvs.length - 1;

	if (prevLen !== nextLen) return false;

	return prevArgvs.every((preArgv, index) => preArgv === nextArgvs[index]);
}

function equalsContextDefault(prevContext, nextContext) {
	return prevContext === nextContext;
}

function comparedEquals(equalsMethod) {
	let called = false;

	return function(...values) {

		if (!called) {
			called = true;
			return false;
		}

		return equalsMethod(...values);
	}
}

function memoizeOne(func, equalsArgvs = equalsArgvsDefault, equalsContext = equalsContextDefault) {
	let prevResult = null;
	let prevArgvs = [];
	let prevContext = null;

	const compare = comparedEquals(function({
		prevContext,
		nextContext,
		prevArgvs,
		nextArgvs
	}) {
		return equalsContext(prevContext, nextContext) && equalsArgvs(prevArgvs, nextArgvs);
	});

	return function(...nextArgvs) {
		if (!compare({
			prevContext,
			nextContext: this,
			prevArgvs,
			nextArgvs,
		})) {
				prevResult = func.call(this, ...nextArgvs);
				prevArgvs = nextArgvs;
				prevContext = this;
		}

		return prevResult;
	}
}

```

######闭包的劣势
-	**内存泄露**
	闭包保存了本应该被垃圾回收装置回收的执行期上下文.

#### IIFE
IIFE(立即执行函数)是一种特殊的函数表达式
函数声明是不能被立即执行的
```JavaScript
function iife() {}() // error
```
能被执行的只能是表达式
```JavaScript
iife(); // right
```
所以可以使用()将函数声明变成函数表达式,然后执行
```JavaScript
(function() {
 
}())

(function() {})()
```

##### IIFE的作用
ES6之前JavaScript中是只存在函数作用域的,这意味着私有变量的实现需要借助函数.IIFE大部分是用来做这个的
```JavaScript
const myModule = (function() {
	
	var a = 1;	
	
	return {
		setA(value) { a = value }
		getA() { return a }
	}		
}())

// 这样这个a就是个myModule的私有属性
```

还有一种情况是
```JavaScript
for (var i = 0; i < 10; i += 1) {
	setTimeout(() => {
		console.log(i);	
	});
}
// 打印10个10.
// 这是因为JavaScript的EventLoop机制.

// ES6之前我们这样解决
for (var i = 0; i < 10; i += 1) {
	(function(j) {
		setTimeout(() => {
			console.log(j);		
		}); 
	}(i));
}
// 1 -> 10

// ES5中可以直接使用let块作用域解决.
```

#### 上下文
因为JavaScript的函数是基于词法分析的,但是有的时候我们又需要使用一些和运行时有关的状态.
上下文就是解决👆问题的方式.
函数中访问的this就是该函数的上下文.
这个上下文会随着函数的调用情况而改变,在函数方法/属性中提到的call,apply,bind就是改变上下文的几种方式之一.
改变this指向的几种方法(按照优先级排列)
- **箭头函数**
一种比较好理解的方式是箭头函数没有this(在chrome调试中可以发现箭头函数的this是undefined),在箭头函数中访问this的话会遍历其原型链(词法作用域).
-	**new 运算符**
new 运算符让函数的this指向了以函数原型为原型对象创建的函数...😂
```JavaScript
function Context() {
	// 这里this = Object.create(Context.prototype)
}

new Context();
```
- **bind**
- **call/apply**
- **JS引擎默认的上下文规则**
	this指向函数的调用者.
	没有调用者的话,严格模式下是undefined,非严格模式下是globalThis(一种处于提案中的语法,浏览器环境下是window,node环境下是global)

比较难以记忆的就是new和bind,bind和call/apply之间的关系,手动实现一下bind函数能帮助记忆.

#### 实现一个bind函数
[TODO这里是一个地址]()

#### 柯里化/偏函数/Thunk函数
如果按照范围来划分的话
柯里化:{ 偏函数: { Thunk函数: {} }  }

- **柯里化**
函数的参数可以一部分一部分的传入.eg:
```JavaScript
add(1)(2)(3)(4);
add(1,2,3)(4);
add(1)(2,3,4);
```
- **偏函数**
柯里化的子集,参数分为2部分传入.eg:
```JavaScript
add(1)(2, 3, 4);
add(1,2,3,4)();
```
bind就是一个偏函数.

- **Thunk函数**
关于什么是Thunk/Thunk函数可以去看[阮老师的ES6API](https://es6.ruanyifeng.com/#docs/generator-async#Thunk-%E5%87%BD%E6%95%B0)
柯里化的子集,参数分为2部分传入,最后一部分的参数必须是callback(err, data)的形式
```JavaScript
add(1, 2, 3, 4)((function(err, data) { console.log(data) })
```

这里是柯里化/偏函数/Thunk函数的实现.
TODO
