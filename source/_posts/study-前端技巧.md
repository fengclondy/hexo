---
layout: post
title: study--前端技巧
date: 2018-07-31 10:45:24
tags: study
categories: study
---


### Bootstrap

1、给图片添加 img-responsive 的class属性,图片的宽度就能自动适配你手机屏幕的宽度啦。

2、text-center的class属性,标题文字居中。

3、btn-block这个class讲按钮升级为块级元素,使button标签的宽度充满整个屏幕.


### Vue一些bug修复

#### 1、Vue项目的发布

config目录下的index.js文件，修改 build 的生产环境（dev 是开发环境）的 assetsPublicPath 属性，该属性对应项目名称

#### 2、Vue下路由History mode导致页面无法渲染的问题（白屏）

一般开发的单页应用都是带有#号hash模式，如果觉得带#号不美观，那么可以使用history模式
```
history: mode
```
但是要 通过 npm run build 发布项目的时候，则需要 将其注释掉，这样路由才会加上项目名称，才可以访问。

https://juejin.im/post/5ad56d86518825556534ff4b


### 安装webpack

1、先在项目中`npm init`初始化一下，生成package.json

2、运行 `npm i webpack webpack-cli -D`命令

npm i -D 是 npm install --save-dev 的简写，是指安装模块并保存到 package.json 的 devDependencies中，主要在开发环境中的依赖包

3、在使用webpack进行打包的时候，默认情况下会将src下的入口文件(index.js)进行打包

// node v8.2版本以后都会有一个npx
// npx会执行bin里的文件

npx webpack     // 不设置mode的情况下 打包出来的文件自动压缩

npx webpack --mode development  // 设置mode为开发模式，打包后的文件不被压缩

当执行npx webpack命令的时候，webpack会自动查找项目中src目录下的index.js文件，然后进行打包，生成一个dist目录并存在一个打包好的main.js文件

<!-- more -->

### 文件结构

在项目下创建一个webpack.config.js(默认，可修改)文件来配置webpack
```
module.exports = {
    entry: '',               // 入口文件
    output: {},              // 出口文件
    module: {},              // 处理对应模块
    plugins: [],             // 对应的插件
    devServer: {},           // 开发服务器配置
    mode: 'development'      // 模式配置
}
```

启动devServer需要安装一下webpack-dev-server

`npm i webpack-dev-server -D`

参考：https://juejin.im/post/5adea0106fb9a07a9d6ff6de

### CONSOLE页面输出
CONSOLE对象的一些方法，经常使用的是console.log("ok!")。
```
Object {
    assert: ...,
    clear: ...,
    count: ...,
    debug: ...,
    dir: ...,
    dirxml: ...,
    error: ...,
    group: ...,
    groupCollapsed: ...,
    groupEnd: ...,
    info: ...,
    log: ...,
    markTimeline: ...,
    profile: ...,
    profileEnd: ...,
    table: ...,
    time: ...,
    timeEnd: ...,
    timeStamp: ...,
    timeline: ...,
    timelineEnd: ...,
    trace: ...,
    warn: ...
}
```


