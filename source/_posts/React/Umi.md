---
title: UmiJS-v4
date: 2023/08/02 14:05:00
updated: 2023/08/02 17:28:00
categories:
  - [React, Umi]
tags:
---

# API

[API](https://umijs.org/docs/api/api)

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

### Link 组件跳转

`<Link>` 是 React 组件，是带路由跳转功能的 `<a>` 元素。

类型定义如下：

```ts
declare function Link(props: {
  /** 预加载 */
  prefetch?: boolean;
  to: string | Partial<{ pathname: string; search: string; hash: string }>;
  replace?: boolean;
  state?: any;
  /** <Link> -> <a> */
  reloadDocument?: boolean;
}): React.ReactElement;
```

示例：

```jsx
import { Link } from "umi";

function IndexPage({ user }) {
  return <Link to={user.id}>{user.name}</Link>;
}
```

## 路由参数获取

路由相关参数有：routeMatch、location、params、query。

### match

`useMatch`

```js
const match = useMatch('/comp/:id')
// match
{
  "params": {
    "id": "paramId"
  },
  "pathname": "/comp/paramId/",
  "pathnameBase": "/comp/paramId",
  "pattern": {
  "path": "/comp/:id",
  "caseSensitive": false,
  "end": true
  }
}
```

### location

`useLocation`

```js
const location = useLocation();
// location
{
"pathname": "/path/",
"search": "",
"hash": "",
"state": null,
"key": "default"
}
```

> 🚨
> 推荐使用 useLocation, 而不是直接访问 history.location. 两者的区别是 pathname 的部分。 history.location.pathname 是完整的浏览器的路径名；而 useLocation 中返回的 pathname 是相对项目配置的 base 的路径。
>
> 举例：项目如果配置 base: '/testbase', 当前浏览器地址为 https://localhost:8000/testbase/page/apple
>
> history.location.pathname 为 /testbase/page/apple
>
> useLocation().pathname 为 /page/apple

### Params

`useParams`

```js
// 路由配置 /comp/:id
// 当前 location /comp/paramId
const params = useParams();

// params
{
"id": "paramId"
}
```

### query

`useSearchParams`

```js
// 当前 location /comp?a=b;
const [searchParams, setSearchParams] = useSearchParams();
searchParams.get('a') // b
searchParams.toString() // a=b

setSearchParams({a:'c',d:'e'}) // location 变成 /comp?a=c&d=e
searchParams 的 api 参考
```

# Mock

示例：

```js
// ./mock/users.ts

export default {
  // 返回值可以是数组形式 GET 默认可以省略
  "GET /api/users": [
    { id: 1, name: "foo" },
    { id: 2, name: "bar" },
  ],

  // 返回值也可以是对象形式
  "GET /api/users/1": { id: 1, name: "foo" },
  "POST /api/users": { result: "true" },
  "PUT /api/users/1": { id: 1, name: "new-foo" },
};
```

`defineMock` 开启 ts 提示。

```js
import { defineMock } from "umi";

export default defineMock({
  "/api/users": [
    { id: 1, name: "foo" },
    { id: 2, name: "bar" },
  ],
  "/api/users/1": { id: 1, name: "foo" },
  "GET /api/users/2": (req, res) => {
    res.status(200).json({ id: 2, name: "bar" });
  },
});
```

引入 Mock.js 生成数据：

```js
import mockjs from "mockjs";

export default {
  // 使用 mockjs 等三方库
  "GET /api/tags": mockjs.mock({
    "list|100": [{ name: "@city", "value|1-100": 50, "type|0-2": 1 }],
  }),
};
```

# 代理

同 vite 一样。

```js
export default {
  proxy: {
    "/api": {
      target: "http://jsonplaceholder.typicode.com/",
      changeOrigin: true,
      pathRewrite: { "^/api": "" },
    },
  },
};
```

# 数据预加载

Umi 会自动根据当前路由或准备跳转的路由，并行地发起他们的数据请求，因此当路由组件加载完成后，已经有马上可以使用的数据了。

配置开启：

```js
/** .umirc.ts */

export default {
  clientLoader: {},
};
```

使用方式:

在路由文件中，除了默认导出的页面组件外，再导出一个 clientLoader 函数，并且在该函数内完成路由数据加载的逻辑。

```js
/** pages/.../some_page.tsx */

import { useClientLoaderData } from "umi";

export default function SomePage() {
  const { data } = useClientLoaderData();
  return <div>{data}</div>;
}

export async function clientLoader() {
  const data = await fetch("/api/data");
  return data;
}
```

# 微生成器

Umi 中内置了众多微生成器，协助你在开发中快速的完成一些繁琐的工作。

1. 页面生成器

2. 组件生成器

3. RouteAPI 生成器

4. Mock 生成器

5. Prettier 配置生成器

6. Jest 配置生成器

7. Tailwind CSS 配置生成器

8. DvaJS 配置生成器

9. Precommit 配置生成器

# Umi Max

Umi Pro，内置很多插件。

## Layout

集成 Ant Design 的 Layout（ProLayout）。

### 启用方式

```js
/** config/config.ts */
export default {
  layout: {
    title: "your app title",
  },
};
```

### 构建时配置

```js
/** config/config.ts */
import { defineConfig } from "umi";

export default defineConfig({
  layout: {
    /** 显示在布局左上角的产品名，默认值为包名。 */
    title: "Ant Design",
    /** 是否开始国际化配置。 */
    locale: false,
  },
});
```

### 运行时配置

运行时配置写在 `src/app.tsx` 中，key 为 `layout`。

```ts
import { RunTimeLayoutConfig } from "@umijs/max";

export const layout: RunTimeLayoutConfig = (initialState) => {
  return {
    /** 显示在布局左上角的产品名，默认值为包名 */
    title: "Ant Design",
    /** 显示在布局左上角产品名前的产品 Logo */
    logo: "https://img.alicdn.com/tfs/TB1YHEpwUT1gK0jSZFhXXaAtVXa-28-27.svg",
    /** 用于运行时配置默认 Layout 的 UI 中，点击退出登录的处理逻辑，默认不做处理 */
    logout: (initialState: any) => {}
    /** 发生错误后的回调（可做一些错误日志上报，打点等） */
    onError: (error: Error, info: any) => {};
    /** 发生错误后展示的组件 */
    ErrorComponent: (error: Error) => React.ReactElement<any>;

    // 其他属性见：https://procomponents.ant.design/components/layout#prolayout
  };
};
```

### 拓展路由配置

```ts
// config/route.ts
export const routes: IBestAFSRoute[] = [
  {
    path: "/welcome",
    component: "IndexPage",
    name: "欢迎", // 兼容此写法
    icon: "testicon",

    // 更多功能查看
    // https://beta-pro.ant.design/docs/advanced-menu

    // 新页面打开
    target: "_blank",
    // 不展示顶栏
    headerRender: false,
    // 不展示页脚
    footerRender: false,
    // 不展示菜单
    menuRender: false,
    // 不展示菜单顶栏
    menuHeaderRender: false,
    // 权限配置，需要与 plugin-access 插件配合使用
    access: "canRead",
    // 隐藏子菜单
    hideChildrenInMenu: true,
    // 隐藏自己和子菜单
    hideInMenu: true,
    // 在面包屑中隐藏
    hideInBreadcrumb: true,
    // 子项往上提，仍旧展示,
    flatMenu: true,
  },
];
```

## antd

整合 antd 组件库。

### 启用方式

```js
/** config/config.ts */
export default {
  antd: {
    /** 配置 antd 的 configProvider */
    configProvider: {},
    /** 开启暗色主题 */
    dark: true,
    /** 开启紧凑主题 */
    compact: true,
    /** 配置 antd 的 babel-plugin-import 按需加载 */
    import: true,
    /** 配置使用 antd 的样式，默认 less */
    style: "less",

    /** ---antd.v5--- */
    /** 配置 antd@5 的 theme token，等同于配置 configProvider.theme，且该配置项拥有更高的优先级 */
    theme: {},
    /** 配置 antd 的 App 包裹组件，请注意 antd@5.1.0 ~ 5.2.3 仅能通过 appConfig: {} 启用，只有 antd >=5.3.0 才支持更多 App 配置项目 */
    appConfig: {},
    /** 配置 antd 的 DatePicker、TimePicker、Calendar 组件是否使用 moment 作为日期处理库，默认为 false */
    momentPicker: true,
    /** 配置 antd 的 StyleProvider 组件，该组件用于兼容低版本浏览器，如 IE11。当你的项目配置了 legacy 或者 targets 包含 ie 时，会自动进行降级处理，不需要手动配置。 */
    styleProvider: {
      hashPriority: "high",
      legacyTransformer: true,
    },
  },
};
```

> [configProvider](https://ant.design/components/config-provider-cn?theme=dark) 属性注入。

## Model

`@umi/max` 内置了数据流管理插件，它是一种基于 `hooks` 范式的轻量级数据管理方案，可以在 Umi 项目中管理全局的共享数据。

### Create Model

约定式目录结构：

1. `src/models/xxx.{js,jsx,ts,tsx}`

2. `src/pages/xxx/models/xxx.{js,jsx,ts,tsx}`

> 注意 namespace 需要全局唯一。

### Use Model

```js
/** src/components/Username/index.tsx */
import { useModel } from 'umi';

export default function Page() {
  const { user, loading } = useModel('userModel');

  /** 数据过滤 */
  const { add, minus } = useModel('counterModel', (model) => ({
    add: model.increment,
    minus: model.decrement,
  }));

  return (
    {loading ? <></>: <div>{user.username}</div>}
  );
}
```

### Global Init State

在 `src/app.js` 内暴露 `getInitialState` 方法。

示例：

```ts
/** src/app.ts [createInitState] */
import { fetchInitialData } from "@/services/initial";

export async function getInitialState() {
  const initialData = await fetchInitialData();
  return initialData;
}

/** src/page/xxxComponent.jsx [useInitState] */
import { useModel } from "umi";

export default function Page() {
  const { initialState, loading, error, refresh, setInitialState } = useModel("@@initialState");
  return <>{initialState}</>;
}
```

| 对象属性        | 类型                 | 介绍                                                                                                    |
| --------------- | -------------------- | ------------------------------------------------------------------------------------------------------- |
| initialState    | any                  | 导出的 getInitialState() 方法的返回值                                                                   |
| loading         | boolean              | getInitialState() 或 refresh() 方法是否正在进行中。在首次获取到初始状态前，页面其他部分的渲染都会被阻止 |
| error           | Error                | 如果导出的 getInitialState() 方法运行时报错，报错的错误信息                                             |
| refresh         | () => void           | 重新执行 getInitialState 方法，并获取新的全局初始状态                                                   |
| setInitialState | (state: any) => void | 手动设置 initialState 的值，手动设置完毕会将 loading 置为 false                                         |

## Request

它基于 axios 和 ahooks 的 useRequest 提供了一套统一的网络请求和错误处理方案，主要 hooks 有：`request`、`useRequest`

### 构建时配置

```js
/** config/config.js */
export default {
  request: {
    /** 请求返回数据解构 */
    dataField: "data",
  },
};
```

### 运行时配置

```js
/** src/app.ts */
import type { RequestConfig } from "umi";

export const request: RequestConfig = {
  timeout: 1000,
  /** 请求统一的错误处理方案 */
  errorConfig: {
    errorHandler() {},
    errorThrower() {},
  },
  /** 拦截器 */
  requestInterceptors: [],
  /** 响应器 */
  responseInterceptors: [],
};
```

### API-request

通过 `import { request } from '@@/plugin-request'` 你可以使用内置的请求方法。

```js
request("/api/user", {
  params: { name: 1 },
  timeout: 2000,
  // other axios options
  skipErrorHandler: true,
  getResponse: false,
  requestInterceptors: [],
  responseInterceptors: [],
});
```

### API-useRequest

插件内置了 @ahooksjs/useRequest ，你可以在组件内通过该 Hook 简单便捷的消费数据。

```ts
import { useRequest } from "umi";

export default function Page() {
  const { data, error, loading } = useRequest(() => {
    return services.getUserList("/api/test");
  });
  if (loading) {
    return <div>loading...</div>;
  }
  if (error) {
    return <div>{error.message}</div>;
  }
  return <div>{data.name}</div>;
}
```

> 上面代码中 data 并不是你后端返回的数据，而是其内部的 data，（因为构建时配置默认是 'data')。

[**Umi3 -> Umi4 变化**](https://umijs.org/docs/max/request#umi3-%E5%88%B0-umi4)

## Access

权限管理。

### 启用方式

```js
export default {
  access: {},
  /** access 插件依赖 initial State 所以需要同时开启 */
  initialState: {},
};
```

### 权限定义

我们约定了 `src/access.ts` 为我们的权限定义文件，该文件需要默认导出一个方法，导出的方法会在项目初始化时被执行。该方法需要返回一个对象，对象的每一个值就对应定义了一条权限。如下所示：

```js
/** src/access.ts */
export default function (initialState) {
  const { userId, role } = initialState;

  return {
    canReadFoo: true,
    canUpdateFoo: role === "admin",
    canDeleteFoo: (foo) => {
      return foo.ownerId === userId;
    },
  };
}
```

### 路由配置

路由配置中通过 access 限定权限。

```js
export const routes = [
  {
    path: "/pageA",
    component: "PageA",
    /** 权限定义返回值的某个 key */
    access: "canReadPageA",
  },
];
```

### API-useAccess

组件中获取权限相关信息。

```js
/** src/page/xxxComponent */
import React from "react";
import { useAccess } from "umi";

const PageA = (props) => {
  const { foo } = props;
  const access = useAccess();

  if (access.canReadFoo) {
    // 如果可以读取 Foo，则...
  }

  return <>TODO</>;
};

export default PageA;
```

### API-Access

权限控制组件，支持的属性如下：

1. accessible

2. fallback

3. children

示例：

```js
import React from "react";
import { useAccess, Access } from "umi";

const PageA = (props) => {
  const { foo } = props;
  /** access 的成员: canReadFoo, canUpdateFoo, canDeleteFoo */
  const access = useAccess();

  if (access.canReadFoo) {
    /** 如果可以读取 Foo，则... */
  }

  return (
    <div>
      <Access accessible={access.canReadFoo} fallback={<div>Can not read foo content.</div>}>
        Foo content.
      </Access>
      <Access accessible={access.canUpdateFoo} fallback={<div>Can not update foo.</div>}>
        Update foo.
      </Access>
      <Access accessible={access.canDeleteFoo(foo)} fallback={<div>Can not delete foo.</div>}>
        Delete foo.
      </Access>
    </div>
  );
};
```

## 国际化

约定在 src/locales 目录下引入多语言文件，目录结构如下：

```bash
src
  + locales
    + zh-CN.ts
    + en-US.ts
  pages
```

### 启用方式

```js
/** config/config.js */
export default {
  locale: {
    // 默认使用 src/locales/zh-CN.ts 作为多语言文件
    default: "zh-CN",
    baseSeparator: "-",
  },
};

/** src/locales/zh-CN.ts */
export default {
  welcome: '欢迎光临 Umi 的世界！',
  ...
};

/** src/locales/en-US.ts */
export default {
  welcome: "Welcome to Umi's world!",
  ...
};
```

### FormattedMessage

通过 UmiMax 预制的 `<FormattedMessage />` 组件实现渲染。

```tsx
import { FormattedMessage } from "umi";

export default function Page() {
  return (
    <div>
      <FormattedMessage id="welcome" />
    </div>
  );
}
```

渲染结果：

```html
<!-- zh-CN -->
<div>欢迎光临 Umi 的世界！</div>

<!-- en-US -->
<div>Welcome to Umi's world!</div>
```

### 组件参数使用

```js
import { Alert } from "antd";
import { useIntl } from "umi";

export default function Page() {
  const intl = useIntl();
  const msg = intl.formatMessage({
    id: "welcome",
  });

  return <Alert message={msg} type="success" />;
}
```

### 切换语言

方式：

1. `<SelectLang />`

2. `setLocale()`

```js
/** SelectLang UmiMax 预设的切换语言组件 */
import { SelectLang } from "umi";

export default function Page() {
  return <SelectLang />;
}

/** 编程式调用 */
import { setLocale } from "umi";

/** 切换时刷新页面 */
setLocale("en-US");
/** 切换时不刷新页面 */
setLocale("en-US", false);
```

### 配置

```js
/** config/config.js */
export default {
  locale: {
    antd: false, // 如果项目依赖中包含 `antd`，则默认为 true
    baseNavigator: true,
    baseSeparator: "-",
    default: "zh-CN",
    title: false,
    useLocalStorage: true,
  },
};
```

配置的详细介绍如下：

| 配置项          | 类型    | 默认值  | 介绍                                                                                                                                                 |
| --------------- | ------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| antd            | Boolean | false； | 如果项目包含 antd 依赖，则为 true antd 的国际化支持。更多介绍可参见此文档。                                                                          |
| baseNavigator   | Boolean | true    | 开启浏览器语言检测。默认情况下，当前语言环境的识别按照：localStorage 中 umi\*locale 值 > 浏览器检测 > default 设置的默认语言 > zh-CN                 |
| baseSeparator   | String  | -       | 语言（Language）与国家（Country） 之间的分割符。默认情况下为 -，返回的语言及目录文件为 zh-CN、en-US 和 sk 等。若指定为 \*，则 default 默认为 zh_CN。 |
| default         | String  | zh-CN   | 项目默认语言。当检测不到具体语言时，使用 default 设置的默认语言。                                                                                    |
| title           | Boolean | false   | 开启标题国际化。                                                                                                                                     |
| useLocalStorage | Boolean | true    | 自动使用 localStorage 保存当前使用的语言。                                                                                                           |
