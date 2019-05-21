# Vue-Cli



## 安装

在命令行执行安装命令

```
npm install --global vue-cli
```

## 创建 vue 项目目录

```
vue init webpack my-project
```

> 使用 webpack 模板，创建一个名字叫 my-project 的标准项目



## 启动项目

切换到项目目录，执行命令

```
npm run dev
```

或

```
npm run start
```

> start 也可以启动项目，是因为 package.json 文件中定义了一些命令
>
> ```js
> "scripts": {
>     "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
>     "start": "npm run dev",
>     "lint": "eslint --ext .js,.vue src",
>     "build": "node build/build.js"
>   }
> ```

## 项目目录

- build webpack 的配置文件
- config 针对开发环境和线上环境的配置文件
- node_modules  项目的依赖
- src 源代码
- static 静态资源