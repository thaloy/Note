# Refs

@time 2020/05/06

> Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素 ----- React官网

React期望开发者使用声明式的开发方式而不是命令式

- **声明式**
```JavaScript
<Dialog
  show={show}
  title={...}
  {...}
/>

this.setState({
  show: true,
  title: 'xxx',
  ...
});
```

- **命令式**
```JavaScript
Dialog.show({
  title: 'xxx',
  ...
});
```

React认为命令式的代码破坏了其数据流

React自身定位为lib而非framework,所以其不会限制开发者编写命令式的代码,并且一些情况下(暂时没遇到过)使用编写命令式的代码是必然的

Refs就是React提供的对命令式的支持,通过Refs可以获得Dom和React Class Component

### Refs的作用目标

- **Dom**
```JavaScript
<div></div>
```

- **Class Component**
```JavaScript
<Div />
```

需要注意的是Refs不能获得function component,因为其没有实例

### 创建Refs的3种方式

- **React.createRef()**
创建了一个{ current: null }不可扩展对象

- **callback**
接受一个参数element,为我们想要获得的实例

- **useRef(initialValue)**
创建了一个{ current: initialValue }不可扩展对象
// TODO 链接到Hooks种的useRef

tips: React官网还提到了一种字符串形式的Refs的创建方式,因为官网提到其有一些问题,以及将被弃用,所以我没去了解没多大意义

### Refs的本质
Refs的运行机理就是组件/Dom在**mount**,**didUpdated**,**willUnmount**的时候将实例更新到传入组件的ref上

伪代码如下
```JavaScript
function updateRef(ref, element) {
  if (!ref) return;

  if (isFunc(ref)) {
    ref(element);
  }

  if (isObject(ref)) {
    ref.current = element;
  } 
}

function mount() {
  updateRef(ref, element);
}

function unmount() {
  updateRef(ref, null);
}

function update() {
  if (prevRef === currentRef) return;

  updateRef(prefRef, null);
  updateRef(currentRef, element); 
}
```
**ummount** 和 **update**的逻辑都是为了防止内存泄漏

所以React官网中有提到

> 如果 ref 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 null，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

这也解释了为什么React.creatRef()和useRef() 会得到一个对象, 因为对象会打破react的数据流向

同时为了安全考虑,React在创建对象时,会将这个对象变成不可扩展的

- **Object.preventExtensions({ current })**
- **Object.seal({ current })**
- **Object.freeze({ current })**

即:只有current属性可以更改

### 如何获得子组件内的组件/Dom

这种需求的主要场景是高阶组件

```JavaScript
function LogHoc(Comopnent) {
  return class LoggerWrap extends React.Component {
    
    render() {
      return (
        <Component />
      );
    }
  }
}

Div = LogHoc(OriginDiv);

<Div ref={ref} />

// 这个时候我们不能获得OriginDiv的实例,相反获得的是<LoggerWrap />的实例
```

那么我们应该怎么办呢?

- **层层遍历**

```JavaScript
// 上面的例子中
ref.props.children[0]; // 这样就获得了实例

e....................😂
```

- **间接获得**

```JavaScript
function LogHoc(Comopnent) {
  return class LoggerWrap extends React.Component {
    componentRef = React.createRef();

    render() {
      return (
        <Component ref={this.componentRef} />
      );
    }
  }
}

<Div ref={this.ref} />

this.ref.componentRef.current.todo();

// 这种方式的问题在于Refs有2种形态 [对象,callback]基本不能做兼容处理,因为对象的Refs变化对父组件是无感知的
// 只能保持一致,都使用对象或者都使用callback
```

- **透穿**

```JavaScript
function LogHoc(Comopnent, { ref }) {
  return class LoggerWrap extends React.Component {
    render() {
      const ref = ref || this.props.componentRef;

      return (
        <Component ref={ref} />
      );
    }
  }
}

Div = LogHoc(OriginDiv, {
  ref: this.ref,
});

<Div componentRef={this.ref} />

// 感觉还行哈
```

### Refs转发

Refs转发就是为了fix上个内容的,虽然方案3看起来很nice(可以满足我们的需求,且没什么难度)

但是对lib来讲需要提供一个统一的解决方案来处理需要透穿ref的场景, **Refs**就诞生了

因为ref不能作用于function component, 即: function component没有针对ref做逻辑 Refs转发就利用了这个特点, 通过React内置的高阶组件(**React.forwardRef**)抹平组件对外提供的ref上的差异

```JavaScript
function LoggerHoc(Component) {

  class LoggerWrap extends React.Component {
    render() {
      
      return (
        <Component ref={this.props.forwardRef} />
      );
    }
  }

  // forwardRef接受2个参数一个是props，另外一个是之前function component不支持的ref
  return React.forwardRef(props, ref) => {
    return (
      <LoggerWrap forwardRef={ref} /> {/* 这里还是要改名字滴!!!, 高阶组件内部的实现,对外是隐藏的 */}
    ); 
  }
}

<Div ref={this.ref} />
```

为了方便在react-dev-tools上调试,可以设置React.forwardRef接受的function component的displayName, 这个就看官网吧

### useImperativeHandler

在**间接获得**子组件的组件/Dom中，我提到过不能对2种形态的ref做兼容处理,因为ref对象的变化是无感知的.

我不准备推翻前面的话,但是我要给他加一个上下文 即: 在**直接**获得子组件的组件/Dom中是不可能的

如果在高阶组件中不返回这个ref实例,  而是返回一个方法结合, 同时让这个ref实例成为其上下文上的属性/闭包, 就无需感知ref实例的变化了

```JavaScript
function LoggerHoc(WrapComponent) {

  return class DivWrap extends Component {
    ref = createRef();

    hocRef = {
      log: () => {
        console.log(this.ref.current.innerText);
      },
    };

    render() {
      console.log(this.props);
      return (
        <WrapComponent {...this.props} ref={this.ref} />
      );
    }
  }
}

const OriginDiv = React.forwardRef(function OriginDiv(props, ref) {
  return (
    <div ref={ref}>{props.count}</div>
  );
})

const Div = LoggerHoc(OriginDiv);

export default class ImperativeHandler extends Component {

  ref = createRef();

  componentDidMount() {
    this.refInstance.hocRef.log();
    this.ref.current.hocRef.log();
  }

  refCallback = (element) => {
    this.refInstance = element;
  }

  render() {
    return (
      <>
        <Div ref={this.ref} count={1} />
        <Div ref={this.refCallback} count={2} />
      </>
    );
  }
}
```

这其实就是**useImperativeHandler**的作用

**useImperativeHandler**并不是一个新的API，它只是提醒了我们在实现类似Refs转发功能的时候，不一定要转发这个ref实例而已, 只是封装好的一种统一的处理方案

一般情况下userImperativeHandler都和Refs转发一起使用,它帮助我们隐藏组件，只暴露组件的某些接口给父组件
```JavaScript
React.forwardRef(function(props, ref) {
  const hiddenRef = useRef(null);
  
  useImperativeHandler(ref, () => ({
    log() {
      console.log(hiddenRef.current.innerText);
    } 
  }), []);

  return (
    <div ref={hiddenRef}>1</div>
  );
});
```

下面是基于Class Component 实现的imperativeHandler的功能
```JavaScript
// 只是举个例子,还有很多实现细节,就是react Refs防止内存泄漏的一些措施都需要实现
export class ClassImperativeHandler extends Component {
  ref = createRef();

  componentDidMount() {
    this.refInstance.log();
    this.ref.current.log();
  }

  refCallback = (element) => {
    this.refInstance = element;
  }

  render() {
    return (
      <>
        <Input inputRef={this.ref} defaultValue={1} />
        <Input inputRef={this.refCallback} defaultValue={2} />
      </>
    );
  }
}

class Input extends Component {
  ref = React.createRef();

  updateInputRef = () => {
    if (
      Object.prototype.toString.call(this.props.inputRef) === '[object Object]'
    ) {
      this.props.inputRef.current = {
        log: () => {
          console.log(this.ref.current.value);
        }
      }
    } 

    if (
      Object.prototype.toString.call(this.props.inputRef) === '[object Function]'
    ) {
      this.props.inputRef({
        log: () => {
          console.log(this.ref.current.value);
        }
      });
    } 
  }

  componentDidMount() {
    this.updateInputRef();
  }

  render() {
    return (
      <input ref={this.ref} defaultValue={this.props.defaultValue} />
    );
  }
}
```
