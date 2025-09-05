---
title: qiankun 手写思路
authors: [bb]
---

qiankun 有两个关键函数、registerMicroApps 和 start，主要是实现这两个函数
registerMicroApps 很简单，就是配置子应用参数

```js
let _apps: any[] = [];
export const registerMicroApps: any = (apps: any[]) => {
  _apps = apps;
};
```

其次是 start，start 主要实现四个步骤，1、劫持路由 2、查找子应用 3、加载子应用 4、渲染子应用

1、劫持路由

路由有两种情况，我们主要介绍 history 路由
1、history 函数的 go、back、forward 使用的 popstate
2、还有 pushstate、replaceState
上面这三个需要重写，要使用 window.addEventListener 重写

```js
window.addEventListener("popstate", () => {
  // 监听浏览器的前进后退
  // 路由处理
});

window.history.pushState = (...args) => {
  // pushState 就是 往浏览器历史栈里插一条记录、改地址栏、但不刷新页面。
  // 路由处理
};

window.history.replaceState = (...args) => {
  // 路由处理
  // 参数和 pushState 一样：state、title、url。区别在于：它是 替换当前历史记录，不会新增一条
};
```

2、查找子应用
在上面 registerMicroApps 寸的子应用中，查找当前路由与注册子应用时的 activeRule

```js
const apps = getApps(); // _apps
const app = apps.find((item) =>
  window.location.pathname.startsWith(item.activeRule)
);
if (!app) {
  return;
}
```

3、加载子应用
我们可以用 fetch 请求子应用资源，然后再给他插入到子应用注册时留的 container

```js
const html = await fetch(url).then((res) => res.text());
const tempalte = document.createElement("div");
tempalte.innerHTML = html;
```

但是这样子应用并不会展示，因为 innerHTML 并不会执行 script，qiankun 是通过 import-html-entry 将 html 的 script、css 等资源解析后请求。
我们就简单实现为解析 script、执行 script

script 分为两种，直接写内容的和用 src 的，要分别处理

```js
const scripts: any = tempalte.querySelectorAll("script");

// 获取所有script
async function getExternalScripts() {
  return Promise.all(
    Array.from(scripts).map((script: any) => {
      const src = script.getAttribute("src");
      if (!src) {
        return Promise.resolve(script.innerHTML);
      } else {
        src;
        return fetchResource(src.startsWith("http") ? src : url + src);
      }
    })
  );
}

// 执行所有script
async function execScripts() {
  const scripts = await getExternalScripts();
  const module = { exports: {} };
  const exports = module.exports;
  scripts.forEach((script: any) => {
    eval(script);
  });
  return module.exports;
}
```
