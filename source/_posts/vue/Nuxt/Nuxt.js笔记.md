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

## 视图

![nuxt-views-schema](https://www.nuxtjs.cn/nuxt-views-schema.svg)

### 模板

入口： app.html

```html
<!DOCTYPE html>
<html {{ HTML_ATTRS }}>
  <head {{ HEAD_ATTRS }}>
    {{ HEAD }}
  </head>
  <body {{ BODY_ATTRS }}>
    {{ APP }}
  </body>
</html>
```

### 布局

layout 目录下创建自定义布局。

#### 默认布局

入口：`layouts/default.vue`

**注意**：在布局文件中需要添加 `<nuxt/>` 组件用于显示页面的主体内容。

```vue
<template>
  <nuxt />
</template>
```

#### 自定义布局

每个页面可以配置独自的布局。

入口：`layouts/xxx.vue`

```vue
<template>
  <!-- Your template -->
</template>
<script>
  export default {
    layout: 'xxx'
  }
</script>
```

#### 错误页面

入口：`layouts/error.vue`

```vue
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">页面不存在</h1>
    <h1 v-else>应用发生错误异常</h1>
    <nuxt-link to="/">首 页</nuxt-link>
  </div>
</template>

<script>
  export default {
    props: ['error'],
    layout: 'blog' // 你可以为错误页面指定自定义的布局
  }
</script>
```

### 页面

同 Vue 组件一致。

```vue
<template>
  <h1 class="red">Hello {{ name }}!</h1>
</template>

<script>
  export default {
    asyncData (context) {
      // called every time before loading the component
      return { name: 'World' }
    },
    fetch () {
      // The fetch method is used to fill the store before rendering the page
    },
    head () {
      // Set Meta Tags for this Page
    },
    // and more functionality to discover
    ...
  }
</script>

<style>
  .red {
    color: red;
  }
</style>
```

页面配置项：

> 只要是函数式写法，第一个参数都可以接收到当前页面的上下文(context)。

1. `asyncData: Function`

   服务端获取并渲染数据，asyncData 在组件**初始化之前**执行。

2. `fetch: Function`

   用于渲染前处理状态机(store)中的数据，与 `asyncData` 方法类似，但他不会设置组件的数据。

3. `head: Object | Function`

   配置head、html。

4. `key: String | Function`

   配置 `<router-view>` 组件的 `key` 属性。

5. `layout: String | Function`

   配置页面布局文件。

6. `loading: Boolean`

   配置是否加载进度条选项。

7. `middleware: String | Array`

   配置中间件文件。

8. `scrollToTop: Boolean`

   控制页面渲染前是否滚动至页面顶部。

9. `transition: String | Object | Function`

   配置过渡动效。

10. `validate: (context: any) => Boolean`

    配置路由参数校验。

11. `watchQuery: Boolean | Array`

    监听属性变化，执行所有组件方法(asyncData, fetch, validate, layout,...)

## 数据获取

### asyncData

组件初始化前调用，第一个参数为当前页面上下文，并且在获取数据之后，nuxt 会将 asyncData 返回的数据融合到组件的 data 方法中，提供给模版使用。

> 注意：由于`asyncData`方法是在组件 **初始化** 前被调用的，所以在方法内是没有办法通过 `this` 来引用组件的实例对象。

```js
/** async-await */
export default {
  async asyncData({ params }) {
    const { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}

/** 回调函数 */
export default {
  asyncData({ params }, callback) {
    axios.get(`https://my-api/posts/${params.id}`).then(res => {
      callback(null, { title: res.data.title })
    })
  }
}
```

### 上下文对象

 [API `context`](https://www.nuxtjs.cn/api/context) 

#### 获取 req/res 对象

在服务器端调用`asyncData`时，您可以访问用户请求的`req`和`res`对象。

```js
export default {
  async asyncData({ req, res }) {
    // 请检查您是否在服务器端
    // 使用 req 和 res
    if (process.server) {
      return { host: req.headers.host }
    }

    return {}
  }
}
```

#### 访问路由参数

```js
export default {
  async asyncData({ params }) {
    const slug = params.slug // When calling /abc the slug will be "abc"
    return { slug }
  }
}
```

#### 错误处理

显示错误信息页面，`params.statusCode` 可用于指定服务端返回的请求状态码。

```js
export default {
  asyncData({ params, error }) {
    return axios
      .get(`https://my-api/posts/${params.id}`)
      .then(res => {
        return { title: res.data.title }
      })
      .catch(e => {
        error({ statusCode: 404, message: 'Post not found' })
      })
  }
}
```

如果你使用的是**回调函数**方式，并且在 catch 阶段执行，nuxt 将自动调用 error 方法。

```js
export default {
  asyncData({ params }, callback) {
    axios
      .get(`https://my-api/posts/${params.id}`)
      .then(res => {
        callback(null, { title: res.data.title })
      })
      .catch(e => {
        callback({ statusCode: 404, message: 'Post not found' })
      })
  }
}
```
