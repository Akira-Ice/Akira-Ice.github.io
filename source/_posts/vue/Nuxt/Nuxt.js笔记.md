---
title: Nuxt.js
categories: 
  - Nuxt3
tags: 
  - Nuxt3
---

# Nuxt.js

## 安装

1. 使用 npx

   `npx create-nuxt-app <项目名>`

2. 使用 yarn

   `yarn create-nuxt-app <项目名>`

## 目录结构

```bash
- assets   			// 静态资源- css /image /js等
- components  	// 组件存放
- layouts  			// 页面主布局 相当于APP.vue
- middleware 		// 中间件存放
- pages    			// 主页面 此文件夹文件会自动生成相对路由(按文件名生成)
- plugins  			// 用于存放引入插件的js
- static   			// 静态资源-不需要经过处理的
- store   	 		// vuex

nuxt.config.js  // nuxt 相关配置
```

## 配置

```js
head: { 
    title: '', 
    meta: [],  
    link: []
},
loading: { color: '#3B8070' },  // 加载时的顶部进程颜色
css: [],     										// 引入css
plugins:[],  										// 引入插件 js
router: {},  										// 配置router相关内容
build: {}    										// 配置webpack相关内容
```

## 路由

Nuxt.js 可根据 page 目录结构自动生成 vue-router 模块的路由配置。

1. `nuxt-link`

   ```vue
   <template>
     <nuxt-link to="/">首页</nuxt-link>
   </template>
   ```

2. `$router`

   同 Vue 中编程导航一致。

   `$router.push("path");`

### 基础路由

匹配规则：文件夹名作为路径，匹配 index.vue 作为路由组件。

```bash
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
```

生成路由配置表为：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'user',
      path: '/user',
      component: 'pages/user/index.vue'
    },
    {
      name: 'user-one',
      path: '/user/one',
      component: 'pages/user/one.vue'
    }
  ]
}
```

> 注意：在嵌套路由中则是优先匹配，**与文件夹同级的同名文件**作为当前文件夹所代表的路径。

### 动态路由

匹配规则：通过 \_params.vue 命名，直接匹配拼接在路径后面，并设置为可选。

```bash
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```

生成路由配置表为：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      name: 'users-id',
      path: '/users/:id?',
      component: 'pages/users/_id.vue'
    },
    {
      name: 'slug',
      path: '/:slug',
      component: 'pages/_slug/index.vue'
    },
    {
      name: 'slug-comments',
      path: '/:slug/comments',
      component: 'pages/_slug/comments.vue'
    }
  ]
}
```

### 路由参数校验

可以在路由组件中配置，相关参数的校验。

```js
/** pages/users/_id.vue */
export default {
  validate({ params }) {
    // 必须是number类型
    return /^\d+$/.test(params.id)
  }
}
```

### 嵌套路由

创建内嵌子路由，你需要添加一个 Vue 文件，同时添加一个**与该文件同名**的目录用来存放子视图组件。

**注意**：需要再父组件中增加 `<nuxt-child/>` 用来显示子组件内容。

```bash
pages/
--| users/
-----| _id.vue
-----| index.vue
--| users.vue
```

生成路由配置表为：

```js
router: {
  routes: [
    {
      path: '/users',
      component: 'pages/users.vue',
      children: [
        {
          path: '',
          component: 'pages/users/index.vue',
          name: 'users'
        },
        {
          path: ':id',
          component: 'pages/users/_id.vue',
          name: 'users-id'
        }
      ]
    }
  ]
}
```

### 万能路由

类似一些 404 重定向路由，可以通过 _.vue 来实现。

### SPA fallback

`spa fallback` 是一个可以用于解决单页应用程序(SPA)中 404 页面的问题的特殊配置。

当 Nuxt.js 应用程序按照 SPA 模式生成为静态文件时，如果用户访问不存在的页面，服务器将会返回 404 状态码的响应。但是，由于 SPA 模式下只有一个 HTML 文件，因此无法直接处理这类请求。

```js
export default {
  generate: {
    fallback: true
  }
}
```

> 需要注意的是，启用 `fallback` 功能可能会对你的应用程序产生一定的影响。例如，具有重要性的 SEO 数据可能无法准确获取，因为所有页面都会返回 200 状态码，而不管其是否实际存在。因此，使用 `fallback` 功能时需要根据自己的需求进行权衡和调整。

### transtion

仍可以使用 `transition` 组件来实现路由切换动效。

#### 全局过渡动效

nuxt.js 默认使用 page 作为过渡效果名。

通过 `nuxt.config.js` 中配置 css。

```css
/** assets/globalTransition.css */
.page-enter-active,
.page-leave-active {
  transition: opacity 0.5s;
}
.page-enter,
.page-leave-active {
  opacity: 0;
}
```

```js
/** nuxt.config.js */
module.exports = {
  css: ['assets/globalTransition.css']
}
```

#### 局部过渡动效

在上述 `globalTransition.css` 中增加动效，并在页面中配置 `transition` 属性。

```css
/** assets/globalTransition.css */
...

.test-enter-active,
.test-leave-active {
  transition: opacity 0.5s;
}
.test-enter,
.test-leave-active {
  opacity: 0;
}
```

```js
/** xxx.vue */
export default {
  transition: 'test'
}
```

### 中间件

自定义函数 -> 执行在一个页面或一组页面渲染之前。

中间件位置：`middleware/`

每个中间件函数接收 [context](https://www.nuxtjs.cn/api/context) 作为参数。

```javascript
/** middleware/stats.js */
export default function (context) {
  context.userAgent = process.server
    ? context.req.headers['user-agent']
    : navigator.userAgent
}
```

```javascript
/** nuxt.config.js */
/** 全局 */
module.exports = {
  router: {
    middleware: 'stats'
  }
}

/** pages/xxx.vue or layouts/xxx.vue */
/** 局部 */
export default {
  middleware: 'stats'
}
```

执行流程：

1. nuxt.config.js
2. 匹配布局
3. 匹配页面
