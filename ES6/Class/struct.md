# Class

`@time 2020/06/03`

**Class**和一层语法糖, 其目的是为了让有面向对象编程基础的开发者更容易进行**JavaScript**开发

**Class**对应了**构造函数**, **JavaScript**通过**构造函数/原型链**实现的继承和其他面向对象语法(Java)具有本质上的区别, 为了抹平原理上的差异, **JavaScript**提供了**Class**, 实现和其他面形对象语言类似的**API**

**Class**只是一层`糖🍬`, 其实现的功能我们都可以用**构造函数**来实现

### 额外的优势

虽然**Class**只是一层 🍬, 但是其也具有一些优势，因为其和传统本质的区别

在传统继承中, 无非有 3 种方式

1.  创建对象，然后通过其他构造函数改造对象
2.  将其他构造函数的实例设置为当前构造函数的原型, 然后创建对象
3.  组合 1&&2

这样对于某些原生构造函数的继承来讲会有一些问题

```JavaScript
function SuperArray() {}

SuperArray.prototype = new Array();

var superArray = new SuperArray();

superArray[2] = 1;
console.log(superArray.length); // 0
```

这是由于`superArray`是一个普通的对象, `superArray[2] = 1`并不会影响`SuperArray.prototype`上的`length`属性

然而, 使用**Class**就规避掉了这个问题

```javaScript
class SuperArray extends Array {}

var superArray = new SuperArray();

superArray[2] = 1;
console.log(superArray.length); // 3
```

为什么会这样请看[TODO class 继承原理]()

### API

```JavaScript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  getName() { return this.name }

  getAge() { return this.age }
}

const my = new Person('thalo', 18);
```

#### 类的方法和属性

由于类是可以被继承的, 所以类约定了类的方法被定义在`原型`上, 类的属性被定义在`实例`上(方法需要被共享，属性则每个实例唯一, 不共享)

基于此, `getName`和`getAge`被定义到`my.__proto__`上, `name`和`age`被定义到`my`上

##### 方法不可枚举

与默认的`寄生组合继承`不同的是，类中的方法是不可被枚举的，这是理所应当的(想不到什么场景需要遍历方法)

```JavaScript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.getName = function() { return this.name };
Person.prototype.getAge = function() { return this.age };

function F() {}
F.prototype = Person.prototype;

Child.prototype = new F();
Child.prototype.constructor = Child;

function Child(name, age) {
  Person.call(this, name, age);
}

const my = new Child('thalo', 18);

for (let prop in my) { console.log(prop) } // name, age, constructor, getName, getAge

// -------------------------------------------

class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

class Child extends Person {

}

const my = new Child('thalo', 18);

for (let prop in my) { console.log(prop) } // name, age
```

##### constructor 方法

前面提到类只是一层 🍬, 所以其`constructor`就是我们使用传统继承声明的`构造函数`

```JavaScript
function Person() {
  this.name = 'thalo';
}

class Person {
  constructor() {
    this.name = 'thalo';
  }
}

// 等价的
```

所以`constructor`可以返回其他对象

```JavaScript
class Person {
  constructor() {
    return Object.create({});
  }
}

console.log(new Person().constructor); // Object
```

并且`constructor`的默认值为`function constructor() {}`

```JavaScript
class Person {} // 等价于
class Person {
  constructor() {}
}
```

对类的索引就是对其构造函数的索引

```JavaScript
class Person {
}

console.log(Person === Person.prototype.constructor); // true
```

##### 属性的提案

为了让属性和方法在形式上统一，有一种新的提案, 已经被 babel 和 chrome 支持

```JavaScript
class Person {
  name = 'thalo';
  age = 18;

  getName() {
    return this.name;
  }
}

// 等价于

class Person {
  constructor() {
    this.name = 'thalo';
    this.age = 18;
  }

  getName() {
    return this.name;
  }
}
```

同时这种写法也支持方法, 但是这种情况下方法是被定义到实例上的, 也就可以`被枚举`, `不共享`, 相当于属性, 常用于绑定`this`(react 的事件)

```JavaScript
class Person {
  name = 'thalo';
  age = 18;

  getName = () => {
    return this.name;
  };
}

// 等价于
class Person {
  constructor() {
    this.name = 'thalo';
    this.age = 18;

    this.getName = () => {
      return this.name;
    };
  }
}
```

##### getter 和 setter

我们也可以为定义属性取值器(getter)和属性存储器(setter), getter 和 setter 定义的属性不可被枚举

```JavaScript
class Person {
  _name = 'thalo'

  get name() {
    return this._name;
  }

  set name(val) {
    this._name = val;
  }
}
```

##### 属性表达式

也像对象一样支持属性表达式

```JavaScript
const propA = 'name';
const getPropA = 'getName';

class Person {
  [propA] = 'thalo';

  [getPropA]() {
    return this[propA];
  }
}
```

#### 静态方法和静态属性

前面提到的`类的方法和属性`都是通过类的实例访问的, 还有一些方法/属性是定义到类上面的, 即: `静态方法和静态属性`

##### 静态方法

静态方法通过`static`关键字定义

```JavaScript
class Person {
  static toString() {
    return 'Person'
  }
}

Person.tostring(); // Person

// 等价于
Person.toString = function() {
  return 'Person'
};
```

##### 静态属性

类的静态属性只能通过`Class.prop = value`的方式添加, 但是现在有一种提案允许通过`static`来添加静态属性

```JavaScript
class Person {
  static time = new Date();
}

console.log(Person.time);

// 等价于
Person.time = new Date();
```

#### 类的注意事项和技巧

##### 方法绑定 this

有的时候我们要为类的方法绑定`this`, 比如 react 的事件, 一般我们有 2 种方式

- **箭头函数**
- **bind**

```JavaScript
class Button extends React.Component {

  // 方案1
  handleButtonClick = () => {
    // something
  }

  // 方案2
  constructor() {
    this.handleButtonClick = this.handleButtonClick.bind(this);
  }

  handleButtonClick() {
    // something
  }

  render() {
    <button onClick={this.handleButtonClick}>点击</button>
  }
}
```

##### 暂时性死区

`class`关键字存在暂时性死区，就像`let`一样, 不允许重复，声明前不可以使用

##### 严格模式

`class`内的代码默认采用严格模式

##### 只能通过 new 来实例化

前面我们提到过对类的访问就是访问其构造函数, 我们知道构造函数是可以单独执行的(没有`new`), 但是类是不可以的 🙅‍♂️

```JavaScript
class Person {}
Person(); // TypeError
```

##### 类的 name 属性和类的表达式

对于函数而言存在`函数声明`和`函数表达式`2 种创建函数的方式, 类也是一样的，前面创建类的方式都是声明, 当然其也存在表示式创建, 其`name`属性和函数是一样的

```JavaScript
const Person = class {}; // Person.name = Person
const Person = class A {}; // Person.name = A
class Person {} // Person.name = Person
```

### new.target

**ES6**对`new`运算符做了扩展, 其也可以作为对象来访问`new.target`, 这个属性表示了当前函数/类是如何执行的

- **正常执行, 返回undefined**
- **被`new`运算符执行, 返回当前构造函数/类**
- **如果当前类是作为基类被执行, 返回其子类**

```JavaScript
function Person() {
  console.log(new.target, 'new.target');
}

Person(); // undefined
new Person(); // Person.toString
```

```JavaScript
class Person {
  constructor() {
    console.log(new.target, 'new.target');
  }
}

new Person(); // Person.toString
```

```JavaScript
class Animal {
  constructor() {
    console.log(new.target, 'animal new.target');
  }
}

class Dog extends Animal {
  constructor() {
    super();
    console.log(new.target, 'dog new.target');
  }
}

const dog = new Dog(); // Dog.toString * 2
```

#### 创建一个不能被实例化的基类

```JavaScript
class Basic {
  constructor() {
    if (new.target === Basic) {
      throw new Error('当前类不可以被实例化，只能被继承');
    }
  }
}

class Derived extends Basic {

}

new Derived();
new Basic(); // Error
```

### 类的私有属性/方法

#### 传统的私有属性/方法

##### 私有属性

传统的私有属性有以下3种方案

1.  构造函数内闭包

```JavaScript
function Person() {
  let age = 18;
}
```

其问题在于, 对于应用到这个`私有属性`方法, 需要在每次实例化的时候都创建一遍，造成`内存泄漏`

```JavaScript
function Person() {
  let age = 18;

  this.upgrade = function upgrade() {
    age ++; 
  };

  this.getAge = function getAge() {
    return age;
  };
}

const person1 = new Person();
const person2 = new Person();
```

2.  立即执行函数内闭包

```JavaScript
const Person = (function() {
  let age = 18;  

  class PersonFactory {}

  return PersonFactory;
}());
```

这种方案的问题在于，需要保证类/构造函数的实例单例

```JavaScript
const Person = (function() {
  let age = 18;

  class PersonFactory {
    
    upgrade() {
      age ++;
    }

    getAge() {
      return age;
    }
  }

  return PersonFactory
}());

const person1 = new Person();
person1.upgrade();

const person2 = new Person();
person2.getAge(); // person2共享了person1的age
```

所以这种情况下我们需要加锁🔒

```JavaScript
const Person = (function() {
  let age = 18;
  let instance = null;
  let instantiateLock = false;

  class PersonFactory {
    static getPerson() {
      if (instance) return instance;

      instantiateLock = true;

      instance = new PersonFactory();

      instantiateLock = false;

      return instance;
    }

    constructor() {
      if (!instantiateLock) throw new TypeError('请使用getPerson静态方法获得实例');
    }

    upgrade() {
      age ++;
    }

    getAge() {
      return age;
    }
  }

  return PersonFactory;
}());

const person1 = Person.getPerson();
const person2 = Person.getPerson();
console.log(person1 === person2);
```

3.  借助于`WeakMap`和立即执行函数

```JavaScript
const Person = (function() {

  const age = new WeakMap();
  
  class PersonFactory {
    constructor() {
      age.set(this, 18);
    }
  
    upgrade() {
      age.set(this, age.get(this) + 1);
    }
  
    getAge() {
      return age.get(this);
    }
  }

  return PersonFactory;
}());

const person1 = new Person();
const person2 = new Person();
```

这种方法比较好的实现了私有属性, 借助于`Weak`的弱索引机制, 在内存回收的时候不会考虑到`WeakMap`对实例的引用

##### 私有方法

私有方法的实现是由私有属性决定的, 这里根据`方案3`的实现来实现私有方法, 一个闭包函数就可以解决

```JavaScript
const Person = (function() {

  const age = new WeakMap();
  const foodList = new WeakMap();

  function addFood(food) {
    foodList.get(this).push(food);
  }

  class PersonFactory {
    constructor() {
      age.set(this, 18);
      foodList.set(this, []);
    }
  
    upgrade() {
      age.set(this, age.get(this) + 1);
    }
  
    getAge() {
      return age.get(this);
    }
    
    eat(food) {
      console.log(`吃 ${food}`);

      addFood.call(this, food);
    }

    getFoodList() {
      return foodList.get(this);
    }
  }

  return PersonFactory;
}());

const person1 = new Person();
const person2 = new Person();
```

#### 关于私有方法/属性的新提案

新提案中约定了用`#`作为前缀的属性/方法是私有属性/方法

tips: 当前`chrome`并未实现对私有方法的支持

```JavaScript
class Person {
  #age = 18;

  upgrade() {
    this.#age += 1;
  }

  getAge() {
    return this.#age;
  }
}

const person = new Person();
```
