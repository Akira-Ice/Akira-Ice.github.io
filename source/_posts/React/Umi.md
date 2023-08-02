---
title: UmiJS-v4
date: 2023/08/02 14:05:00
updated: 2023/08/02 17:28:00
categories:
  - [React, Umi]
tags:
---

# 设计思路

设计思路包括:

1. 技术收敛

2. 插件和插件集

3. import all in umi

4. 编译时框架

5. 依赖预打包

6. 默认快

7. 约束与开放

[4-7 详见](https://mp.weixin.qq.com/s?__biz=MjM5NDgyODI4MQ%3D%3D&mid=2247484533&idx=1&sn=9b15a67b88ebc95476fce1798eb49146)

## 技术收敛

1. 术栈收敛

2. 依赖收敛

![](https://img.alicdn.com/tfs/TB1hE8ywrr1gK0jSZFDXXb9yVXa-1227-620.png)

## 插件和插件集

拓展一类业务，而不仅仅是拓展某一功能。

![](https://img.alicdn.com/tfs/TB1mrhuwqL7gK0jSZFBXXXZZpXa-956-728.png)

## import all in umi

所有的导入都从 umi 中进行导出，减少 import 数量。

`import { connect, useModel, useIntl, useRequest, MicroApp, ... } from 'umi';`

# 目录结构

```bash
.
├── config
│   └── config.ts
├── dist
├── mock
│   └── app.ts｜tsx
├── src
│   ├── .umi
│   ├── .umi-production
│   ├── layouts
│   │   ├── BasicLayout.tsx
│   │   ├── index.less
│   ├── models
│   │   ├── global.ts
│   │   └── index.ts
│   ├── pages
│   │   ├── index.less
│   │   └── index.tsx
│   ├── utils // 推荐目录
│   │   └── index.ts
│   ├── services // 推荐目录
│   │   └── api.ts
│   ├── app.(ts|tsx)
│   ├── global.ts
│   ├── global.(css|less|sass|scss)
│   ├── overrides.(css|less|sass|scss)
│   ├── favicon.(ico|gif|png|jpg|jpeg|svg|avif|webp)
│   └── loading.(tsx|jsx)
├── node_modules
│   └── .cache
│       ├── bundler-webpack
│       ├── mfsu
│       └── mfsu-deps
├── .env
├── plugin.ts
├── .umirc.ts // 与 config/config 文件 2 选一
├── package.json
├── tsconfig.json
└── typings.d.ts
```

# 路由

## 配置式路由

同 Vue-Router 路由配置基本一致。

```js
/** .umirc.ts or config/config.js */
export default {
  routes: [
    /** 嵌套路由 & layout */
    {
      path: "/",
      component: "@/layouts/index",
      routes: [
        { path: "/list", component: "list" },
        { path: "/admin", component: "admin" },
      ],
    },
    /** redirect */
    { path: "/err", redirect: "/list" },
    /** wrapper */
    { path: "/user", component: "user", wrappers: ["@/wrappers/auth"] },
  ],
};

/** src/wrappers/auth */
import { Navigate, Outlet } from 'umi'

export default (props) => {
  const { isLogin } = useAuth();
  if (isLogin) {
    return <Outlet />;
  } else{
    return <Navigate to="/login" />;
  }
}
```

> wrappers 中的每个组件会给当前的路由组件增加一层嵌套路由，如果你希望路由结构不发生变化，推荐使用高阶组件。先在高阶组件中实现 wrapper 中的逻辑，然后使用该高阶组件装饰对应的路由组件。

## 约定式路由

通过文件目录的结构生成路由表。

比如以下文件结构：

```bash
.
  └── pages
    ├── index.tsx
    └── users.tsx
```

会得到以下路由配置，

```js
[
  { path: "/", component: "@/pages/index" },
  { path: "/users", component: "@/pages/users" },
];
```

## 动态路由

约定，带 $ 前缀的目录或文件为动态路由。若 $ 后不指定参数名，则代表 \* 通配，比如以下目录结构：

1. src/pages/users/$id.tsx 会成为 /users/:id

2. src/pages/users/$id/settings.tsx 会成为 /users/:id/settings

举个完整的例子，比如以下文件结构，

```bash
+ pages/
  + foo/
    - $slug.tsx
  + $bar/
    - $.tsx
  - index.tsx
```

会生成路由配置如下：

```js
[
  { path: "/", component: "@/pages/index.tsx" },
  { path: "/foo/:slug", component: "@/pages/foo/$slug.tsx" },
  { path: "/:bar/*", component: "@/pages/$bar/$.tsx" },
];
```

## 路由跳转

### History

和 history 相关的操作，用于获取当前路由信息、执行路由跳转、监听路由变更。

```js
import { history } from "umi";

history.push("/list");

/** string */
history.push("/list?a=b&c=d#anchor", state);

/** object */
history.push(
  {
    pathname: "/list",
    search: "?a=b&c=d",
    hash: "anchor",
  },
  /** historyState */
  {
    some: "state-data",
  }
);

history.push({}, state);

history.back();
history.go(-1);
```

> 注意：history.push 和 history.replace 需要使用 state 需将 state 作为这两个 API 的第二个参数传递
