# React

### What React ?
React是一种JavaScript-UI-Lib,帮助我们更好的构建视图

### Why React ?
React提升了代码可读性,开发效率,以及应用的**平均**性能。

-	**表达力,代码可读性/维护性**
JavaScript通过JSX让JS表示HTML成为可能。更好的实现了Web Component

	刚开始接触前端时,React/Vue正流行起来.JQuery已经成为过去时,原声JS只写过蛮简单的Demo。没有在稍大型的项目是用过,所以具体原声JS实现大型项目中可能遇到的问题这里并不描述.

	在公司接触过一个原声JS实现的h5营销页面,其实现的方式大致如下
	-	根据页面拆封成多个html字符串template
	-	在入口JS处控制各个html模版的显示
	最初把项目交给我的时候我是拒绝的!!!!
	(这里吐槽一下,除了调取后台提供的和微信交互的登陆接口外,还串行请求了大概3个后端接口,才能拿到显示页面必须用到的数据)

	后面这个项目越来越复杂
	-	引导其他页面的banner
	- 弹窗
	-	列表
	-	前端路由
	...

	迭代了几次后发现不能将所有的逻辑都放到入口js处了,主要怕后面的人崩溃.
	受到React/Vue的影响将功能封装成组件(类),自身的功能自身控制。	
	入口JS负责处理各个类之间的关系,大概实现如👇

	并且这种方式并没有React的那样容易理解,尤其是出现一些字符串模版是需要动态计算拼接的时候

	同时React/Vue提供的VDom/VNode,让实现中不会出现一些BOM API,即使不懂BOM,也很容易可以理解这个组件在各个状态的UI.

  ```JavaScript
  class Dialog {
  	constructor(parentNode) {
  		this.init(parentNode);	
  	}
  
  	init(parentNode) {
  		this.parentNode = parentNode;
  		this.initTemplate();
  		this.mountTemplate();
  		this.bindEvent();
  	}
  
  	initTemplate() {
  		this.template = `
  			<div id="dialog">	
  				<div class="container">
  					这是一个Dialog
  				</div>
  			</div>
  		`	
  	}
  
  	mountTemplate() {
  		this.parentNode.innerHTML = this.template;	
  	}
  
  	bindEvent() {
  		// 这里绑定事件	
  	}
  
  	close() {
  		this.closeDom = this.closeDom || document.getElementsByClassName('close')[0];
  
  		this.closeDom.style.display = 'none';
  	}
  	
  	show() {
  		this.closeDom = this.closeDom || document.getElementsByClassName('close')[0];
  
  		this.closeDom.style.display = 'fixed';
  	}
  }
  ```

- **提升应用的平均性能**
> 这里强调**平均**性能

在理解React渲染机制前,一直认为React是为了提升性能而诞生的.这几乎形成了条件反射

事实上React为了diff出视图的变化,进行了很多Cpu运算,而这些变化在开发前我们就知道了。React花费了额外的时间和Cpu资源在计算出一个已知的结果上。所以其并不会提升性能。

事实上,上面的情况发生在React和一段优秀的原生代码做比较.

举个例子
已知
	浏览器是有重绘和回流
	cpu愈发优秀
	相较cpu运算,渲染线程会更加耗时
场景
	当点击页面上某个按钮的时候,页面的A,B,C三个组成部分都发生变化
实际
	优秀的代码会触发一次页面渲染
	xxxx的代码就不一定会触发多少次页面渲染了 -_-
而使用React几乎所有人的实现都渲染一次, 只不过cpu在diff VDom时占用的时间可能会有不同.这个时间并不是影响性能差异的主要因素.

同时由于VDom的存在,获得dom上的属性并不需要通过DOM API(**它是个很耗时的操作**)

所以React相当于减少了开发者在实现前端页面上的差距,甚至没有开发过前端页面的开发者,也能够实现性能足够优秀的页面

- **开发效率**

React可以让开发者将更多的精力放到业务逻辑上。而非页面渲染性能, 解藕页面逻辑. 
