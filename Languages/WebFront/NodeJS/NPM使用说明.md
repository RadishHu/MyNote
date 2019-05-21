# npm



## npm 简介

NPM 是跟 NodeJS 一块安装的包管理工具，用来解决 NodeJS 代码部署问题，常用的使用场景有：

- 从 NPM 服务器下载别人编写的第三方包到本地使用
- 从 NPM 服务器下载并安装别人编写的命令行程序到本地使用
- 将自己编写的包或命令行程序上传到 NPM 服务器供别人使用

新版 NodeJS 已经集成 npm，可以通过 `npm -v` 来测试是否安装成功：

```
$ npm -v
5.6.0
```

升级 npm 版本：

```
$ npm install npm -g
```

## 使用 npm 命令安装模块

npm 安装 NodeJS 模块语法：

```
$ npm install <Module Name>
```

以安装 express 模块为例：

```
$ npm install express
```

安装好之后，express 包放在工程目录下的 node_modules 目录中，代码中引用包：

```js
var express = require('express');
```

## 全局安装与本地安装

npm 包安装分为本地安装 (local) 、全局安装 (global) 两种：

```
# 本地安装
npm install express

# 全局安装
npm install express -g
```

**本地安装**

- 将安装包放在 ./node_modules 下 (运行 npm 命令时所在的目录)
- 可以通过 require() 来引入本地安装包

**全局安装**

- 将安装包放在 /user/local 下或 node 的安装目录
- 可以直接在命令行使用

## 查看安装信息

查看所有全局安装的模块：

```
npm list -g
```

查看某个模块的版本号：

```
npm list modelName
```

## 卸载模块

卸载已经安装的 NodeJS 模块：

```
$ npm uninstall express
```

## 更新模块

更新模块：

```
$ npm update express
```

## 使用淘宝 NPM 镜像

国内直接使用 npm 官方非常慢，可以使用淘宝 npm 镜像：

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

这样就可以使用 cnpm 命令来安装模块：

```
$ cnpm install moduleName
```

