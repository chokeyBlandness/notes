# 前端微服务single-SPA探索

### 前端微服务化的优势

1. 复杂度可控。每一个UI业务模块由独立的前端团队开发，避免代码巨无霸，保持开发时的高速编译，保持较低的复杂度，便于维护与开发效率。
2. 独立部署。每一个模块可单独部署，颗粒度可小到单个组件的UI独立部署，不对其他模块有任何影响。
3. 技术栈无关。也是最具吸引力的，在同一项目下可以使用如今市面上所有前端技术栈，也包括未来的前端技术栈。
4. 容错性高。单个模块发生错误，不影响全局。
5. 拓展性强。每一个服务可以独立横向扩展以满足业务伸缩性，与资源的不必要消耗。

### 何时需要前端微服务化

项目过于庞大，代码编译慢，开发体验差，需要一种更高维度的解耦方案。

### 为何不使用Iframe

为什么不用 iframe，这几乎是所有微前端方案第一个会被 challenge 的问题。但是大部分微前端方案又不约而同放弃了 iframe 方案，自然是有原因的，并不是为了 "炫技" 或者刻意追求 "特立独行"。

如果不考虑体验问题，iframe 几乎是最完美的微前端解决方案了。

iframe 最大的特性就是提供了浏览器原生的硬隔离方案，不论是样式隔离、js 隔离这类问题统统都能被完美解决。但他的最大问题也在于他的隔离性无法被突破，导致应用间上下文无法被共享，随之带来的开发体验、产品体验的问题。

1. url 不同步。浏览器刷新 iframe url 状态丢失、后退前进按钮无法使用。
2. UI 不同步，DOM 结构不共享。想象一下屏幕右下角 1/4 的 iframe 里来一个带遮罩层的弹框，同时我们要求这个弹框要浏览器居中显示，还要浏览器 resize 时自动居中..
3. 全局上下文完全隔离，内存变量不共享。iframe 内外系统的通信、数据同步等需求，主应用的 cookie 要透传到根域名都不同的子应用中实现免登效果。
4. 慢。每次子应用进入都是一次浏览器上下文重建、资源重新加载的过程。

其中有的问题比较好解决(问题1)，有的问题我们可以睁一只眼闭一只眼(问题4)，但有的问题我们则很难解决(问题3)甚至无法解决(问题2)，而这些无法解决的问题恰恰又会给产品带来非常严重的体验问题， 最终导致iframe方案被舍弃。

## 一、Single-SPA

一个用于前端微服务化的JavaScript解决方案

- (兼容各种技术栈)在同一个页面中使用多种技术框架(React， Vue， AngularJS， Angular， Ember等任意技术框架)，并且不需要刷新页面.
- (无需重构现有代码)使用新的技术框架编写代码，现有项目中的代码无需重构.
- (更优的性能)每个独立模块的代码可做到按需加载，不浪费额外资源.
- 每个独立模块可独立运行.

##### 普通前端单体应用架构图

![普通前端单体应用架构图](D:\notes\pictures\普通前端单体应用架构图.png)

##### 微前端架构图

![微前端架构图](D:\notes\pictures\微前端架构图.png)

### 简单创建一个Single-SPA项目

以react应用为例

###### 1、创建一个HTML文件

~~~html
<html>
<body>
    <div id="root"></div>
    <script src="single-spa-config.js"></script>
</body>
</html>
~~~

###### 2、创建single-spa-config.js文件

~~~javascript
// single-spa-config.js
import * as singleSpa from 'single-spa';
// 加载react 项目的入口js文件 (模块加载)
const loadingFunction = () => import('./react/app.js');
// 当url前缀为 /react的时候.返回 true (底层路由)
const activityFunction = location => location.pathname.startsWith('/react');
// 注册应用 
singleSpa.registerApplication('react', loadingFunction, activityFunction);
//singleSpa 启动
singleSpa.start();
~~~

###### 3、封装React项目的渲染出口文件

~~~javascript
import React from 'react'
import ReactDOM from 'react-dom'
import singleSpaReact from 'single-spa-react'
import RootComponent from './root.component'

if (process.env.NODE_ENV === 'development') {
  // 开发环境直接渲染
  ReactDOM.render(<RootComponent />, document.getElementById('root'))
}

//创建生命周期实例
const reactLifecycles = singleSpaReact({
  React,
  ReactDOM,
  rootComponent: RootComponent,
  domElementGetter: () => document.getElementById('root')
})

// 项目启动的钩子
export const bootstrap = [
  reactLifecycles.bootstrap，
]
// 项目启动后的钩子
export const mount = [
  reactLifecycles.mount
]
// 项目卸载的钩子
export const unmount = [
  reactLifecycles.unmount
]
~~~

当我们的浏览器url的前缀有`/react`的时候，程序就可以正常渲染这个应用 所以，所以我们这个react应用的所有路由前缀都得有`/react`

## 二、模块加载器

主要功能：**项目配置文件的加载**、**项目对外接口文件的加载（消息总线会用到）**、**项目入口文件的加载**。

##### 配置文件

我们实践微前端的过程中，我们对每个模块项目，都有一个对外的配置文件，是模块在注册到singe-spa时候所用到的信息。

~~~javascript
{
    "name": "name", //模块名称
    "path": "/project", //模块url前缀
    // "path":["/project-url-path1","/project-url-path2"], // 当模块有多种url前缀的时候
    "prefix": "/module-prefix/", //模块文件路径前缀
    "main": "/module-prefix/main.js", //模块渲染出口文件
    "store": "/module-prefix/store.js",//模块对外接口
    "base": true 
    // 当模块被定性为baseApp的时候,
    // 不管url怎么变化,项目也是会被渲染的,
    // 使用场景为,模块职责主要为整个框架的布局或者一直被渲染,不会改变的部分
  }
~~~

##### 模块加载器

把这个project.config.js配置文件直接引入，优化single-spa-config.js文件。在加载模块时，这里使用了SystemJS作为模块加载工具。

~~~javascript
import '../libs/es6-promise.auto.min'
import * as singleSpa from 'single-spa'; 
import { registerApp } from './Register'

async function bootstrap() {
    // project.config.js 文件为所有模块的配置集合
    let projectConfig = await SystemJS.import('/project.config.js' )

    // 遍历,注册所有模块
    projectConfig.projects.forEach( element => {
        registerApp({
            name: element.name,
            main: element.main,
            url: element.prefix,
            store:element.store,
            base: element.base,
            path: element.path
        });
    });
    
    // 项目启动
    singleSpa.start();
}

bootstrap()
~~~

## 三、消息总线

模块与模块站之间通讯的桥梁。

#### 问题

1、应用微服务化之后，每一个单独的模块都是一个黑盒子，里面发生了什么,状态改变了什么，外面的模块是无从得知的。比如模块A想要根据模块B的某一个内部状态进行下一步行为的时候，黑盒子之间没有办法通信。这是一个大麻烦。

2、每一个模块之间都是有生命周期的。当模块被卸载的时候，如何才能保持后续的正常的通信？

#### 解决方法

参考github上的`single-spa-portal-example`。

## 四、路由分发

在单页面前端的路由，目前有两种形式，一种是所有主流浏览器都兼容多hash路由，基本原理为url的hash值的改变，触发了浏览器onhashchange事件来触发组件的更新。

还有一种是高级浏览器才支持的 History API，在`window.history.pushState(null, null, "/profile/")`的时候触发组件的更新。

## 五、构建与部署

以webpack为例

加载器都是用SystemJS来做的，所以微前端的模块最终打包是要符合模块规范的。可使用**amd**模块规范来构建我们的模块。