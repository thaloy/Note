# React的逻辑复用

@time 2020/04/27

React有3种逻辑复用的方式
-	**Mixin**
-	**HOC**
-	**Hooks**

这篇笔记主要谈谈我对这3种逻辑复用的理解
看一下下面的例子

例子:
一个组件Component
5个用来增强ComponentA的复用逻辑
A,B,C,D,E

-	**使用Mixin实现**
没使用过React的Mixin,这里谈一下自己的理解
```JavaScipt
// 伪代码(Vue)
@mixin(logicA)
@mixin(logicB)
@mixin(logicC)
@mixin(logicD)
@mixin(logicE)
ComponentA
```
logicA,logicB,logicC,logicD,logicE都被
