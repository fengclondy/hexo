---
layout: post
title: package.json详解
date: 2018-02-07 05:06:58
tags: node
categories: node
---

### 初识
1. npm安装package.json时  直接转到当前项目目录下用命令npm install 或npm install --save-dev安装即可，自动将package.json中的模块安装到node-modules文件夹下
2. package.json 中添加中文注释会编译出错
3. 每个项目的根目录下面，一般都有一个package.json文件，定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）。
    npm install 命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境。

4. package.json文件可以手工编写，也可以使用npm init命令自动生成。

>注意：npm init 时，用户需回答一些问题，然后在当前目录生成一个基本的package.json文件。所有问题之中，
>
>只有项目名称（name）和项目版本（version）是必填的，其他都是选填的。

<!-- more -->

### 进一步理解

1、下面是最简单的的一个package.json 文件（只有两个数据，项目名称和项目版本，他们都是必须的，如果没有就无法install）
```json
{
  "name": "kocla_test",
  "version": "1.0.0",
}
```
命名规则：
* name的长度必须小于等于214个字符。
* name不能以"."(点)或者"_"(下划线)开头。
* name中不能包含大写字母。
* name最终将被用作URL的一部分、命令行的参数和文件夹名。因此，name不能含有非URL安全的字符。
* name和version字段的组合必须是唯一的，这样才能被npm认证，才可以通过上传，在npm install安装。

2、scripts指定了运行脚本命令的npm命令行缩写，比如start指定了运行npm run start时，所要执行的命令
```json
"scripts": {
    "dev": "node build/dev-server.js",
    "build": "node build/build.js",
    "unit": "cross-env BABEL_ENV=test karma start test/unit/karma.conf.js --single-run",
    "test": "npm run unit",
    "lint": "eslint --ext .js,.vue src test/unit/specs"
  }，
```
上面的设置指定了`npm run dev`、`npm run bulid`、`npm run unit`、`npm run test`、`npm run lint`时，所要执行的命令。

3、dependencies和devDependencies两项，分别指定了项目运行所依赖的模块、项目开发所需要的模块。
它们都指向一个对象，该对象的各个成员，分别由模块名和对应的版本要去组成，表示依赖的模块及其版本范围
--save参数表示将该模块写入dependencies属性，
--save-dev表示将该模块写入devDependencies属性。
```json
"dependencies": {
    "vue": "^2.2.2",
    "vue-router": "^2.2.0"
  },
  "devDependencies": {
    "autoprefixer": "^6.7.2",
    "babel-core": "^6.22.1",
    "babel-eslint": "^7.1.1",
    "babel-loader": "^6.2.10",
    "babel-plugin-transform-runtime": "^6.22.0",
    "babel-preset-env": "^1.2.1",
    "babel-preset-stage-2": "^6.22.0",
    "babel-register": "^6.22.0",
    "chalk": "^1.1.3",
}
```
规则：
* \>=version 必须大于等于这个version
* ^version 与version兼容
* 1.2.x 可以是1.2.0、1.2.1等，但不能是1.3.0
* http://... 参考下面的URL作为依赖项
* \* 匹配任何版本
* ""(空字符串) 匹配任何版本，和\*一样
* version1 - version2 相当于 >=version1 <=version2
* range1 || range2 range1或range2其中一个满足时采用该version
* git... 参考下面的Git URL作为依赖项
* path/path/path 参考下面的本地Paths


4、config字段用于向环境变量输出值。
```json
{ 
  "name" : "foo", 
  "config" : { "port" : "8080" }, 
  "scripts" : { "start" : "node server.js" } 
} 
```

5、engines字段指明了该项目所需要的node.js版本
```json
"engines": {
   "node": ">= 4.0.0",
   "npm": ">= 3.0.0"
 },
```

6、bin许多包有一个或多个可执行文件希望被安装到系统路径，它是一个命令名和本地文件名的映射。
在安装时，如果是全局安装，npm将会使用符号链接把这些文件链接到prefix/bin，如果是本地安装，会链接到./node_modules/.bin/
```
{ "bin" : { "myapp" : "./cli.js" } }
```
这样，当你安装myapp，npm会从cli.js文件创建一个到/usr/local/bin/myapp的符号链接(这使你可以直接在命令行执行myapp)
如果你只有一个可执行文件，那么它的名字应该和包名相同，此时只需要提供这个文件路径(字符串)

7、cpu 如果你的代码只能运行在特定的cpu架构上，你可以指明：
```
"cpu" : [ "x64", "ia32" ]
```

8、description包的描述，description是一个字符串。它可以帮助人们在使用npm search时找到这个包。

9、keywords包的关键字，keywords是一个字符串的数组。它可以帮助人们在使用npm search时找到这个包。

10、homepage项目主页的url。

更多详细配置，请移步至    [官方文档](https://docs.npmjs.com/files/package.json)