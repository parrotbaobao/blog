---
title: qiankun 简介
authors: [bb]
---

qiankun 本质上是一个基于 single-spa 的微前端框架，它的运行步骤可以理解为一个 主应用注册子应用、加载子应用并渲染、应用间通信与卸载 的过程。下面我给你梳理一个完整的运行流程：

1、定义子应用入口信息
当匹配到activeRule的时候，请求entry资源，渲染到container中
```js
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'vueApp', // 子应用名称
    entry: '//localhost:8080', // 子应用入口
    container: '#sub-container', // 渲染到哪个容器
    activeRule: '/vue', // 路由规则
  },
  {
    name: 'reactApp',
    entry: '//localhost:3000',
    container: '#sub-container',
    activeRule: '/react',
  },
]);

// 启动 qiankun
start();

```

2、子应用导出生命钩子函数
qiankun加载子应用资源会去拉去子应用index.html，并解析其中的JS、CSS，这时候需要子应用需要在入口模块中导出生命钩子bootstrap（渲染之前）、mount（渲染）、unmount（卸载），qiankun会按需调用他们。
bootstrap() → 初始化（只执行一次）

mount(props) → 渲染（每次进入路由时执行）

unmount(props) → 卸载（离开路由时执行）

子应用要允许跨域、要打包出umd格式的库文件
```js
output: {
  library: `${name}-[name]`,
  libraryTarget: 'umd',
  chunkLoadingGlobal: `webpackJsonp_${name}`,
}
```

3、qiankun 会劫持浏览器的路由变化（pushState、replaceState、popstate）。

每当路由切换时，qiankun 会检查当前 URL 是否匹配某个子应用的 activeRule。

5. 样式隔离 & 资源加载

qiankun 提供 沙箱机制（基于 Proxy 或快照）来隔离子应用的全局变量。

提供 CSS 样式隔离，避免子应用样式污染主应用。


6. 应用间通信

主应用可以通过 props 给子应用传值。

也可以通过 initGlobalState 建立全局状态管理，实现主子应用之间的事件通信。

7. 子应用卸载

当路由切换到另一个子应用时，qiankun 会调用当前子应用的 unmount 方法，清理 DOM、事件监听等资源。
