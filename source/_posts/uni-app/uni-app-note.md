---
title: uni-app-note
date: 2023/09/22 14:21
updated: 2023/10/7 9:59:00
categories:
  - [uni-app]
tags:
  - Note
---

# 概念

## 核心

> 统一规则 -> 多端转义

## 逻辑层和渲染层分离

原生：js、css、html -> webview -> render -> web，加载渲染相互阻塞，抢占资源，性能差。

小程序、app：逻辑层单独配置 js 引擎，渲染层仍是 webview。

> 注意小程序和 app 的逻辑层都不支持浏览器专用的 window、dom 等 API。app 只能在渲染层操作 window、dom，即 renderjs。

# 目录结构

```bash
┌─uniCloud              云空间目录，阿里云为uniCloud-aliyun,腾讯云为uniCloud-tcb（详见uniCloud）
│─components            符合vue组件规范的uni-app组件目录*
│  └─comp-a.vue         可复用的a组件
├─utssdk                存放uts文件
├─pages                 业务页面文件存放的目录*
│  ├─index
│  │  └─index.vue       index页面
│  └─list
│     └─list.vue        list页面
├─static                存放应用引用的本地静态资源（如图片、视频等）的目录，注意：静态资源只能存放于此*
├─uni_modules           存放[uni_module](/uni_modules)*
├─platforms             存放各平台专用页面的目录，详见
├─nativeplugins         App原生语言插件
├─nativeResources       App端原生资源目录
│  ├─android            Android原生资源目录
|  └─ios                iOS原生资源目录
├─hybrid                App端存放本地html文件的目录，详见
|-windows               拓展窗口：leftWindow、topWindow、rightWindow
├─wxcomponents          存放小程序组件的目录，详见
├─unpackage             非工程代码，一般存放运行或发行的编译结果
├─AndroidManifest.xml   Android原生应用清单文件
├─Info.plist            iOS原生应用配置文件
├─main.js               Vue初始化入口文件*
├─App.vue               应用配置，用来配置App全局样式以及监听 应用生命周期*
├─manifest.json         配置应用名称、appid、logo、版本等打包信息，详见*
├─pages.json            配置页面路由、导航条、选项卡等页面类信息，详见*
└─uni.scss              这里是uni-app内置的常用样式变量*
```

> static 目录下文件会直接复制到打包目录，不会进行编译处理，除非被引用。

# 页面

## 页面加载时序

uni-app -> pages.json -> template -> vnode -> onLoad(data) -> [start animation] -> [vnode to dom] -> onReady -> [end animation]

## 页面生命周期

onShow、onHide：进入、离开，重复触发。

onReachBottom：触底事件（pages.json -> onReachBottomDistance）。

onPageScroll：滚动事件 `({scrollTop}) => void`。

## 组件生命周期

原 Vue 生命周期，beforeUpdate、updated 仅 H5 页面支持

## 页面通信

[同 Vue 自定义事件](https://uniapp.dcloud.net.cn/tutorial/page.html#%E9%A1%B5%E9%9D%A2%E9%80%9A%E8%AE%AF)。

[页面通信 API](https://uniapp.dcloud.net.cn/api/window/communication.html)

```js
function handleUpdate(data) {
  console.log(data.msg);
}

uni.$emit("update", { msg: "页面更新" });

uni.$on("update", handleUpdate);

uni.$once("update", handleUpdate);

uni.$off("update", this.update);
```

## 路由

[pages.json](https://uniapp.dcloud.net.cn/collocation/pages#pages) 中配置。

### 路由跳转

跳转方式：[navigator](https://uniapp.dcloud.net.cn/component/navigator)、[API](https://uniapp.dcloud.net.cn/api/router)。

# CSS

尺寸单位：rpx。

内联样式：多为动态样式，静态样式通过 class 形式编写，避免内联静态样式影响渲染。

`::after ::before`：仅在 Vue 页面生效。

uni-app 提供内置 css 变量：

| CSS 变量            | 描述                   | App                           | 小程序 | H5                   |
| ------------------- | ---------------------- | ----------------------------- | ------ | -------------------- |
| --status-bar-height | 系统状态栏高度         | 系统状态栏高度、nvue 注意见下 | 25px   | 0                    |
| --window-top        | 内容区域距离顶部的距离 | 0                             | 0      | NavigationBar 的高度 |
| --window-bottom     | 内容区域距离底部的距离 | 0                             | 0      | TabBar 的高度        |

**注意：**

1. uni-app 中不能使用 \* 。

2. body -> page || div、ul、li -> view || span、font -> text、a -> navigator、img -> image。

3. NavigationBar 导航栏高度固定 44px（不可更改）。

4. TabBar 底部选项卡高度固定 50px。

5. 100vh 包含导航栏的高度，使用时需要减去导航栏和 tabBar 高度，部分浏览器还包含浏览器操作栏高度，使用时请注意。

6. 本地背景图片的引用路径推荐使用以 ~@ 开头的绝对路径。

7. 微信小程序仅支持 class 选择器。

8. 小程序不支持在 css 中引用文件，包括图片和字体，需要转为 base64 使用。

9. 微信小程序不支持相对路径。

10. 小程序和 H5 正常，但 APP 异常，通常是 css 兼容性问题。（小程序不存在浏览器兼容问题，其内置很大的定制 webview）

11. 各家小程序浏览器内核不同，可能存在 css 兼容性问题，[细节参考](https://ask.dcloud.net.cn/article/1318)。

# JS

1. 原生组件层级问题 H5 没有原生组件概念问题，非 H5 端有原生组件并引发了原生组件层级高于前端组件的概念，要遮挡 video、map 等原生组件，请使用 cover-view 组件。

2. 非 H5 端不支持 API：document、cookie、window、location、navigator、localstorage、websql、indexdb、webgl 等对象。

3. APP 端通过 renderJS 操作 window、document 库。

> 小程序和 App 的 js 运行在 jscore 下而不是浏览器里，没有浏览器专用的 js 对象。若三方库中使用到这些 API 需要在插件市场寻找替代品。
> APP 端可在 [renderJS](https://uniapp.dcloud.io/tutorial/renderjs) 中操作浏览器对象。

# nvue

采用原生渲染，但 css 会有限制。

**注意：**

1. 只能使用 v-if 进行显隐。

2. 只支持 flex 布局。

3. 布局不能使用百分比、没有媒体查询。

4. 不能在 style 中引入字体文件。

5. 内容只能在 \<text\> 中，且仅 text 中文字可设置颜色、大小。

6. nvue 页面布局默认 column，需要在 manifest.json -> app-plus -> nvue -> flex-direction 进行修改，仅 uni-app 生效。

7. nvue 切换横竖屏时可能导致样式出现问题，建议有 nvue 的页面锁定手机方向。

# API

## getApp()

```ts
const getApp: <AnyObject>(opts?: App.GetAppOption) => App.AppInstance<AnyObject> & AnyObject;
interface AppInstance<T extends AnyObject = {}> {
  globalData?: AnyObject;
  onLaunch?(options?: LaunchShowOption): void;
  onShow?(options?: LaunchShowOption): void;
  onHide?(): void;
  onError?(error: string): void;
  onPageNotFound?(options: PageNotFoundOption): void;
  onUnhandledRejection?(options: UniNamespace.OnUnhandledRejectionCallbackResult): void;
  onThemeChange?(options: UniNamespace.OnThemeChangeCallbackResult): void;
  onUniNViewMessage?(options: AnyObject): void;
}
```

用于获取当前应用实例，一般用于获取 globalData，全局数据。

```js
const app = getApp();
console.log(app.globalData);
app.doSomething(); // 调用 App.vue methods 中的 doSomething 方法
```

globalData 原本是小程序中的概念，uni-app 引入后在 H5、APP 等平台都实现了。

```vue
<script>
export default {
  globalData: {
    text: "text",
  },
};
</script>

<style>
/*每个页面公共css */
</style>
```

## getCurrentPages()

`getCurrentPages: <{}>() => Page.PageInstance<AnyObject, {}>[]`

`instance: {route?: string, $getAppWebview?:  () => PlusWebviewWebviewObject,$vm?: any;}`

用于获取当前页面栈的实例，以数组形式按栈的顺序给出，数组中的元素为页面实例，第一个元素为首页，最后一个元素为当前页面。

## $getAppWebview()

在 getCurrentPages()获得的页面里内置了一个方法 $getAppWebview() 可以得到当前 webview 的对象实例，从而实现对 webview 更强大的控制，可参考：[WebviewObject](http://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject)

**此方法仅 App 支持**

```js
var pages = getCurrentPages();
var page = pages[pages.length - 1];
// #ifdef APP-PLUS
var currentWebview = page.$getAppWebview();
console.log(currentWebview.id); //获得当前webview的id
console.log(currentWebview.isVisible()); //查询当前webview是否可见
// #endif
```

## dom

通过 uni.requireNativePlugin 引入 App 原生插件：`uni.requireNativePlugin('dom')`。

### addRule

Weex 提供 DOM.addRule 以加载自定义字体。

```ts
interface IContentObject {
  fontFamily: string;
  src: string;
}
function addRule(type: string, contentObject: IContentObject) {}
```

### scrollToElement

滚动 ref 对应组件到可视区域，仅可作用于可滚动组件的子组件：`<scroller>、<list>、<waterfall>` 等可滚动组件中。

```ts
interface IOptions {
  /** 偏移 */
  offset: number;
  /** 动画 */
  animated: boolean;
}
function scrollToElement(ref: IRefDom, options: IOptions) {}
```

### getComponentRect

获取节点外框 computed。

```ts
function getComponentRect(ref, callback) {}
```

## animation

指定组件执行动画。

[animation](https://uniapp.dcloud.net.cn/tutorial/nvue-api.html#animation)

# 参考

## 快速入门

1. [H5(Vue) 转 uni-app](https://ask.dcloud.net.cn/article/36174)

2. [Vu2 转 Vue3](https://uniapp.dcloud.net.cn/tutorial/migration-to-vue3.html)

3. [页面生命周期](https://uniapp.dcloud.net.cn/tutorial/page.html#lifecycle)

4. [UI 组件库](https://ask.dcloud.net.cn/article/35489)

5. [uni-app 组件](https://uniapp.dcloud.net.cn/component/)

## 常见问题

1. [页面加载常见问题](https://uniapp.dcloud.net.cn/tutorial/page.html#pagefaq)

2. [nvue 开发与 vue 开发的常见区别](https://uniapp.dcloud.net.cn/tutorial/nvue-outline.html#nvue%E5%BC%80%E5%8F%91%E4%B8%8Evue%E5%BC%80%E5%8F%91%E7%9A%84%E5%B8%B8%E8%A7%81%E5%8C%BA%E5%88%AB)

## 导航栏

1. [导航栏开发指南](https://ask.dcloud.net.cn/article/34921)

2. [导航栏示例](https://ext.dcloud.net.cn/plugin?id=1765)

## 数据通信

1. [全局数据流方案](https://ask.dcloud.net.cn/article/35021)

2. [页面通信方案](https://uniapp.dcloud.net.cn/tutorial/page.html#%E9%A1%B5%E9%9D%A2%E9%80%9A%E8%AE%AF)。

3. [页面通信 API](https://uniapp.dcloud.net.cn/api/window/communication.html)

4. [小程序浏览器内核细节参考](https://ask.dcloud.net.cn/article/1318)。

## API 文档

1. [pages.json](https://uniapp.dcloud.net.cn/collocation/pages#pages) 中配置。

2. [navigator](https://uniapp.dcloud.net.cn/component/navigator)

3. [API](https://uniapp.dcloud.net.cn/api/router)

4. [renderJS](https://uniapp.dcloud.io/tutorial/renderjs)

5. [WebviewObject](http://www.html5plus.org/doc/zh_cn/webview.html#plus.webview.WebviewObject)
