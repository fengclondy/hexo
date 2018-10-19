---
layout: post
title: study--Vue疑惑解答
date: 2018-10-10 07:18:15
tags: study
categories: study
---

首先，最重要的安装淘宝镜像： `$ npm install -g cnpm --registry=https://registry.npm.taobao.org`

### Vue 生命周期

- beforeCreate：
在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。

- created：
在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见。

- beforeMount：
在挂载开始之前被调用：相关的 render 函数首次被调用。 
该钩子在服务器端渲染期间不被调用。以下周期在服务端渲染期间都不被调用。

- mounted：
el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$el 也在文档内。注意 mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.$nextTick 替换掉 mounted。

- beforeUpdate：
数据更新时调用，发生在虚拟 DOM 打补丁之前。这里适合在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。

- updated：
由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。

- activated：
keep-alive 组件激活时调用。

- deactivated：
keep-alive 组件停用时调用。

- beforeDestroy：
实例销毁之前调用。在这一步，实例仍然完全可用。

- destroyed：
Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

<!-- more -->

### Vue 中的 MVVM 模型
Vue是以数据为驱动的，Vue自身将DOM和数据进行绑定，一旦创建绑定，DOM和数据将保持同步，每当数据发生变化，DOM会跟着变化。

ViewModel是Vue的核心，它是Vue的一个实例。Vue实例是作用在某个HTML元素上的，这个HTML元素可以是body，也可以是某个id所指代的元素。 

DOM Listeners和Data Bindings是实现双向绑定的关键。DOM Listeners监听页面所有View层DOM元素的变化，当发生变化，Model层的数据随之变化；Data Bindings监听Model层的数据，当数据发生变化，View层的DOM元素随之变化。


### Vue 路由中 hash 模式和 history 模式区别

- hash模式：

hash 出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。特点：hash虽然在URL中，但不被包括在HTTP请求中；用来指导浏览器动作，hash不会重加载页面。

- history模式：

history 利用了 html5 history interface 中新增的 pushState() 和 replaceState() 方法。这两个方法应用于浏览器记录栈，在当前已有的 back、forward、go 基础之上，它们提供了对历史记录修改的功能。只是当它们执行修改时，虽然改变了当前的 URL ，但浏览器不会立即向后端发送请求。

**原理**：

hash 模式的原理是 onhashchange 事件，可以在 window 对象上监听这个事件。 
history ：hashchange 只能改变 # 后面的代码片段，history api （pushState、replaceState、go、back、forward） 则给了前端完全的自由，通过在window对象上监听popState()事件。

### Vue 路由中 $route 和 $router 的区别
- $route是“路由信息对象”，包括path，params，hash，query，fullPath，matched，name等路由信息参数。 
- $router是“路由实例”对象包括了路由的跳转方法，钩子函数等。


###  v-show和v-if指令的共同点和不同点?

- `v-show`指令是通过修改元素的`display`CSS属性让其显示或者隐藏
- `v-if`指令是直接销毁和重建DOM达到让元素显示和隐藏的效果

###  如何让CSS只在当前组件中起作用?

将当前组件的`<style>`修改为`<style scoped>`


### `<keep-alive></keep-alive>`的作用是什么?

`<keep-alive></keep-alive>` 包裹动态组件时，会缓存不活动的组件实例,主要用于保留组件状态或避免重新渲染。

大白话: 比如有一个列表和一个详情，那么用户就会经常执行打开详情=>返回列表=>打开详情…这样的话列表和详情都是一个频率很高的页面，那么就可以对列表组件使用`<keep-alive></keep-alive>`进行缓存，
这样用户每次返回列表的时候，都能从缓存中快速渲染，而不是重新渲染

###  Vue中引入组件的步骤?

1）采用ES6的`import ... from ...`语法 或 CommonJS的`require()`方法引入组件。  
2）对组件进行注册,代码如下：
```vue
Vue.component('my-component', 
{  template: '<div>A custom component!</div>'})
```
3）使用组件`<my-component></my-component>`

### 指令`v-el`的作用是什么?

提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标.可以是 CSS 选择器，也可以是一个 HTMLElement 实例,

### 在Vue中使用插件的步骤

1. 采用ES6的`import ... from ...`语法。

2. 使用全局方法`Vue.use( plugin )`使用插件,可以传入一个选项对象`Vue.use(MyPlugin, { someOption: true })`


###  请列举出3个Vue中常用的生命周期钩子函数?

1. **created:** 实例已经创建完成之后调用，在这一步，实例已经完成数据观测，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没有开始，`$el`属性目前还不可见。

2. **mounted:** `el`被新创建的 `vm.$el` 替换，并挂载到实例上去之后调用该钩子。如果 `root`实例挂载了一个文档内元素，当 mounted 被调用时 `vm.$el` 也在文档内。

3. **activated:**:`keep-alive`组件激活时调用。


###  请简述下Vuex的原理和使用方法

![](http://p2jr3pegk.bkt.clouddn.com/vue01.png)

数据单向流动

一个应用可以看作是由上面三部分组成: **View, Actions,State**,数据的流动也是从View => Actions => State =>View 以此达到数据的单向流动.

但是项目较大的, 组件嵌套过多的时候, 多组件共享同一个State会在数据传递时出现很多问题。Vuex就是为了解决这些问题而产生的.


Vuex可以被看作项目中所有组件的数据中心,我们将所有组件中共享的State抽离出来,任何组件都可以访问和操作我们的数据中心


Vuex的组成：一个实例化的Vuex，Store 由 **state**，**mutations** 和 **actions** 三个属性组成:

- **state**中保存着共有数据。
- 改变state中的数据有且只有通过**mutations**中的方法，且mutations中的方法必须是同步的。
- 如果要写异步的方法,需要些在**actions**中，并通过commit到mutations中进行state中数据的更改。