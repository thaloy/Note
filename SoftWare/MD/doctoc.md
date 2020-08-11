# Doctoc

> 根据markdown # 标签自动生成目录结构的工具

[git地址](https://github.com/thlorenz/doctoc)

### 常用功能

- **指定目录位置**

```markdown
<!-- START doctoc -->
<!-- END doctoc -->
```
相当于作用域标示, 这段注释下面的`markdown`内容会作为目录内容, 同时生成的目录会被插入到这段注释间

### 下载安装

需要使用npm包来安装这个工具

```Shell
npm install -g doctoc

echo '# Doctoc' > init.md

doctoc ./init.md # 文件会被覆写
```
