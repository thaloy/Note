# 为什么@ant-desigh/icons是一个独立的包

几天前,同学和我讲...
>	从 4.0 开始，antd 不再内置 Icon 组件，请使用独立的包 @ant-design/icons。
omg.
我要崩溃了(夸张).

Why?
个人不喜欢这样,比如:
使用babel-loader需要配合安装@babel/core @babel/preset-env.
使用react需要配合使用react-dom.

难道这些不应该是个黑盒么,对外只提供接口就好了么
一般人去饭店吃饭会自备食材么?
一般饭店会把几种食材拿出来让顾客选择嘛?

万一某天xxxx发布了个不向后兼容的版本?	难道我还要维护版本间的关系么？
这不应该是lib自己的功能么,为什么要暴露给开发者呢?

#### 为什么要这么做?
-	按需引入?
显然是不正确的,因为Webpack是有TreeShaking功能的,就算Webpack的TreeShaking没有shaking的特别干净,antd还提供有babel-import-plugin来做到按需引入.

-	和原有Icon的冲突?
这也是不正确的.
antd4.0为了解决这个[Issue](https://github.com/ant-design/ant-design/issues/12011#issuecomment-420038579)
将每个Icon的type都变成了一个组件
```JavaScript
<Icon type="A" />
<A />
// 等价的
```

-	代码迁移
我想这个是拆包的原因.
原有的<Icon /> 在antd4.0中是不存在的,取而代之的是各种Type类型的Icon.
对于使用旧antd的项目来讲,如果升级antd就要做许多的回归测试和代码变更,这是一个耗时且易错的过程.
独立出来的@ant-design/icons能够让项目渐进式的迁移,一点点回归.

😈
😈
😈
😈😈😈😈😈😈😈😈或者可以在antd4.0中保留<Icon />😈😈😈😈😈😈😈😈😈
😈
😈
😈
'开玩笑'任何一个开发者都不能忍受好嘛.

#### 结论
拆包是lib在迭代过程中为了兼容'旧'（这个旧是指使用了低版本的该lib）项目的必然产物.
理解了为什么要有各种功能的包,好像也不是不能接受......


