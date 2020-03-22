# 规范提交Message

> [阮一峰的网络日志 Commit message和Change log编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

之前我都是使用👇的方式提交git mesasge
```Shell
git commit -m 'add/fix/delete: 支持xxx功能'
```
构建脚手架,新增MD,增加新功能都是**add**关键字表示

后来看某前辈的提交记录是这样的
```
chore(webpack_confg): 增加环境变量env
```
这样的提交记录更规范,表述的信息更全面

这种格式的message属于[Angular规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.uyo6cb12dt6w)

------------

## Anguler规范

#### Message的格式
Anguler规范要求git message要按照如下格式编写.其格式包括3个部分
	- **Header**
	- **Body**
	- **Footer**
 ```
 	<type>(<scope>): <subject> // Header

	<body> // Body
  
	<footer> // Footer
 ```

- **Header**
header部分主要描述了本次提交的类型,影响的范围和简要描述
	- **<type>**
	type用于表示本次提交的类型,归纳为7种类型并具有不用的关键字
		- **feat**: 新功能
		- **fix**: 修复bug
		- **docs**: 文档修改
		- **chore**: 构建工具的修改
		- **style**: UI的变化
		- **test**: 增加测试用例等和测试相关
		- **refactor**: 重构
	- **<scope>可选**
	本次提交影响的范围
	- **<subject>**
	本次提交目的的简短描述,不超过50个字符

- **Body**
body部分描述了本次提交的详细信息,可以分成多行,每行尽量不要超过72字符(好看)

- **Footer**
footer部分只在2种情况下被需要
	- **不兼容变动**
	如果存在不兼容变动,footer部分需要以**BREAKING CHANGE**开头,后面是对变动的原因,变动理由以及如何迁移的描述
	```
	BREAKING CHANGE: isolate scope bindings definition has changed.

		To migrate the code follow the example below:

		Before
			
		scope: {
		  myAttr: 'attribute',
		}

		After:

		scope: {
		  myAttr: '@',
		}

		The removed `inject` wasn't generaly useful for directives so there should be no code using it.
	```
	- **关闭Issues**
	footer如果以Close开头,可以用来关闭某些Issue。这个关闭操作同时会同步到github上,将对应的issue关闭
	```
	Close #1
	Close #1, #2
	```
从Footer功能的描述上可以知道其多应用于开源项目,业务项目几乎不涉及

#### 规范化Message的好处
- 规范化的Message可以借助开源工具方便的生成Changelog
-	表述清晰,如果发布后出现问题可以快速定位

#### 交互式提交Message的工具

-	**Commitizen**
不是很喜欢这个工具,这里不做介绍
[commitizen git地址](https://github.com/commitizen/cz-cli)

#### TODO
-	**如何校验message格式**
-	**如何生成Changelog**
- **补充一些操作图片**

