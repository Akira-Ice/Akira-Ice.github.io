---
title: Nuxt.js
categories: 
  - Nuxt
tags: 
  - Nuxt
---

# Nuxt.js

## 安装

1. 使用 npx

   `npx create-nuxt-app <项目名>`

2. 使用 yarn

   `yarn create-nuxt-app <项目名>`

## 目录结构

约定俗成的目录结构 -> 良好的起点

### .nuxt

生产目录，由 nuxt 自动生成的一些文件。

### pages

nuxt 根据该目录自动生成路由配置，也就是通常所熟知的路由组件的位置。

不同的是，nuxt 在原有的基础上增加了一些特殊的属性。

**动态路由**

类似动态设置路由的 params 参数，通过  `_paramName.vue` 创建路由组件。

**忽略路由**

倘若需要忽略某些路由，通过 `-xxx.vue` 即可。

当通过 `/paramValue` 访问时，可通过上下文(context.params)可接收 paramValue。

```vue
<template>
  <h1>{{ slug }}</h1>
</template>

<script>
  export default {
    async asyncData({ params }) {
      const slug = params.slug // When calling /abc the slug will be "abc"
      return { slug }
    }
  }
</script>
```

**额外的配置**

1. asyncData: Function

   组件加载前调用，可接受一个 context 函数参数，即当前页面上下文，返回值将被 nuxt 混入到 data 中去。

   ```js
   export default {
     asyncData(context) {
       return { name: 'World' }
     }
   }
   ```

2. fetch: Function

   用于获取异步数据，和 asyncData 类似，但只能在服务端路由渲染时调用，且只在首次加载页面时触发。

   ```vue
   <script>
     export default {
       data() {
         return {
           posts: []
         }
       },
       async fetch() {
         this.posts = await fetch('https://api.nuxtjs.dev/posts').then(res =>
           res.json()
         )
       }
     }
   </script>
   ```

3. head: Function

   设置当前页面的一些 head 信息，nuxt 通过 `vue-meta` 进行更新。

4. layout: String

   配置当前页面的布局。

5. loading: Boolean

   配置当前是否使用顶部加载样式。

   若设为 false，可手动通过 `this.$nuxt.$loading.finish()` 和 `this.$nuxt.$loading.start()`。

6. transition: Object

   配置页面动效。

7. scrollToTop: Boolean

   配置当前页面是否回调顶部。

8. middleware: String | Array

   配置当前页面中间件，将在页面渲染前调用。

9. key: String

   类似 Vue 组件上的 key，只不过这里是作用在 router-link 上。

### components

非路由组件。

通过 nuxt.config.js 下的 components 配置是否自动扫描 components 下的所有组件。

**懒加载**

直接在组件前面加上 `Lazy` 即可，在一些事件之后加载渲染组件。

**嵌套路由**

例如以下目录结构：

```bash
components/
  base/
      foo/
         CustomButton.vue
```

得到的最终路由：`<BaseFooCustomButton />`

### Assets

资源文件：Stylus、Sass、images、fonts。

**Images**

在 template 中，可通过 `~/assets/xxx.png` 访问。

```html
<template>
  <img src="~/assets/your_image.png" />
</template>
```

但在 css 文件中，则不需要斜杠。

```css
background: url('~assets/banner.svg');
```

**Styles**

nuxt 允许通过 nuxt.config.js 配置全局样式。

```js
export default {
  css: [
    // Load a Node.js module directly (here it's a Sass file)
    'bulma',
    // CSS file in the project
    '~/assets/css/main.css',
    // SCSS file in the project
    '~/assets/css/main.scss'
  ]
}
```

### Static

静态资源，改目录下一般包含一些永远不会改变的资源，且将会直接映射到服务器的根目录下。

static 下的文件可通过 `/` 直接访问。

```html
<!-- Static image from static directory -->
<img src="/my-image.png" />

<!-- webpacked image from assets directory -->
<img src="~/assets/my-image-2.png" />
```

若 nuxt.config.js 中配置了 router.base 属性，项目将在子目录下部署，届时就需要加上 base 才能访问到静态资源。

当然也可以通过配置 static.prefix 为 false 来取消前缀。

### nuxt.config.js

nuxt 的配置文件。

1. [alias](https://v2.nuxt.com/docs/configuration-glossary/configuration-alias/)

   配置目录别名。

2. [build](https://v2.nuxt.com/docs/configuration-glossary/configuration-build/)

   配置 webpack，包括 loader、filenames、transpilation。

3. [css](https://v2.nuxt.com/docs/configuration-glossary/configuration-css/)

   配置全局 css 文件。

4. [generate](https://v2.nuxt.com/docs/configuration-glossary/configuration-generate/)

   命令 `nuxt g` 相关配置。

5. [head](https://v2.nuxt.com/docs/configuration-glossary/configuration-head/)

   配置 web head。

6. [loading](https://v2.nuxt.com/docs/configuration-glossary/configuration-loading/)

   配置页面加载。

7. [modules](https://v2.nuxt.com/docs/configuration-glossary/configuration-modules/)

   加载第三方库。

8. [plugins](https://v2.nuxt.com/docs/configuration-glossary/configuration-plugins/)

   加载插件。

9. [router](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/)

   配置 nuxt 路由。

10. [server](https://v2.nuxt.com/docs/configuration-glossary/configuration-server/)

    配置服务相关属性。

11. srcDir

    配置当前 nuxt 应用的源目录。

12. .gitignore

    配置忽略文件。

## 视图

![nuxt-views-schema](https://www.nuxtjs.cn/nuxt-views-schema.svg)

### 页面

同 Vue 组件一致，只不过 nuxt 为每一个页面添加了一些特殊的属性。

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

配置项：

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

#### 动态页面

通过 _name.vue 来实现动态参数的页面，name 将整合到路由的 params 参数中。

#### 忽略页面

通过 -name.vue 来实现。

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

每个页面可以配置独自的布局，自定义布局文件应在 `layouts` 目录下，且需要确保在布局文件中添加 `<Nuxt/>` 来显示页面。

```vue
/** layouts/blob.vue */
<template>
  <div>
    <div>My blog navigation bar here</div>
    <Nuxt />
  </div>
</template>

/** pages/posts.vue */
<template>
  <!-- Your template -->
</template>
<script>
  export default {
    layout: 'blog'
    // page component definitions
  }
</script>
```

#### 错误页面

```vue
/** layouts/error.vue */
<template>
  <div>
    <h1 v-if="error.statusCode === 404">Page not found</h1>
    <h1 v-else>An error occurred</h1>
    <NuxtLink to="/">Home page</NuxtLink>
  </div>
</template>

<script>
  export default {
    props: ['error'],
    layout: 'error' // you can set a custom layout for the error page
  }
</script>
```

### app.html

可根据 nuxt.config.js 中的配置动态配置根入口文件的相关属性。

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

## context

![c12c33](https://s2.loli.net/2023/05/31/6wLcWYTUzMymNOZ.png)

context 在 nuxt 提供的一些函数中是可用的，比如：asyncData、plugins、middleware、nuxtServerInit，其属性如上图。

> 不要与 action 中的 context 混淆！

```js
function (context) { // Could be asyncData, nuxtServerInit, ...
  // Always available
  const {
    app,
    store,
    route,
    params,
    query,
    env,
    isDev,
    isHMR,
    redirect,
    error,
    $config
  } = context

  // Only available on the Server-side
  if (process.server) {
    const { req, res, beforeNuxtRender, beforeSerialize } = context
  }

  // Only available on the Client-side
  if (process.client) {
    const { from, nuxtState } = context
  }
}
```

### 获取路由 params 参数

```js
export default {
  async asyncData(context) {
    const id = context.params.id
    try {
      // Using the nuxtjs/http module here exposed via context.app
      const post = await context.app.$http.$get(
        `https://api.nuxtjs.dev/posts/${id}`
      )
      return { post }
    } catch (e) {
      context.error(e) // Show the nuxt error page with the thrown error
    }
  }
}
```

### 访问 store 并重定向（鉴权）

```js
export default {
  middleware({ store, redirect }) {
    // retrieving keys via object destructuring
    const isAuthenticated = store.state.authenticated
    if (!isAuthenticated) {
      return redirect('/login')
    }
  }
}
```

## Helper

nuxt 的辅助器。

### $nuxt

`$nuxt` 可以在 vue 组件中通过 `this.$nuxt` 访问，在客户端也可以通过 `window.$nuxt` 访问。

#### 检查用户是否联网

`$nuxt.isOffline` 提供了快速查明用户是否连接互联网。

```html
<template>
  <div>
    <div v-if="$nuxt.isOffline">You are offline</div>
    <Nuxt />
  </div>
</template>
```

#### 刷新页面数据

`$nuxt.refresh` 方法将重新调用 asyncData、fetch 重新获取数据。

```html
<template>
  <div>
    <div>{{ content }}</div>
    <button @click="refresh">Refresh</button>
  </div>
</template>

<script>
  export default {
    asyncData() {
      return { content: 'Created at: ' + new Date() }
    },
    methods: {
      refresh() {
        this.$nuxt.refresh()
      }
    }
  }
</script>
```

#### 控制加载条

可通过 `$nuxt.$loading` 实现对加载条的控制。

```js
export default {
  mounted() {
    this.$nextTick(() => {
      this.$nuxt.$loading.start()
      setTimeout(() => this.$nuxt.$loading.finish(), 500)
    })
  }
}
```

### window.onNuxtReady

在 nuxt 应用加载就绪之后执行一些回调。

```js
window.onNuxtReady(() => {
  console.log('Nuxt is ready and mounted')
})
```

### Process

nuxt 集成了三个标志（client、server、static），用于判断当前应用程序在服务器、客户端或静态站点上呈现，通常在 asyncData 中使用。

```html
<template>
  <h1>I am rendered on the {{ renderedOn }} side</h1>
</template>

<script>
  export default {
    asyncData() {
      return { renderedOn: process.client ? 'client' : 'server' }
    }
  }
</script>
```

## SSR

服务端渲染，nuxt 亦为此而生，所谓服务端渲染，即在服务端进行页面编译解析成 html，再交给客户端渲染，至此不需要浏览器进行 js 的解析以及执行。

Node.js server 是必要的：

1. js 的执行需要 node 环境的。
2. node 环境还需要做一些配置来实现编译执行 vue 应用。

nuxt 也可以通过 serverMiddleware 来拓展和控制服务端：

```js
/** server-middleware/logger.js */
export default function (req, res, next) {
  console.log(req.url)
  next()
}


/** nuxt.config.js */
export default {
  serverMiddleware: ['~/server-middleware/logger']
}
```

由于是在服务端的原因，通常情况是不可以访问到浏览器独有的 `window` 对象的，只能在  `beforeMount` 和 `mounted` 中访问。

ssr的步骤：

1. 客户端 -> 服务端

   浏览器给 nodejs 服务端发出初始化请求。Nuxt 将生成 html 并返回给浏览器，并且会执行 asyncData、nuxtServerInit、Fetch。

2. 服务端 -> 浏览器

   浏览器接收到 html 进行渲染页面，然后 Vue.js 的 hydration 机制开始运作，使页面响应式，进而实现页面交互。

3. 浏览器 -> 浏览器

   页面之间的跳转通过 `<NuxtLink>` 来完成，除非你刷新页面，整个 ssr 流程将重新进行。

注意事项：

1. window or document is undefined

   这基本是由于在服务端执行的缘故，建议使用 `process.cilent` 判断之后执行。

   ```js
   if (process.client) {
     require('external_library')
   }
   ```

## Nuxt生命周期

![de48ca](https://s2.loli.net/2023/06/01/vpoL8D9CRBxzMae.png)

下面是常用的生命周期：

1. `nuxtServerInit`

   在服务器渲染间执行的方法，初始化 Vuex 的状态或从 API 中获取数据等操作。

2. `middleware`

   执行相关的中间件，顺序是：Global middleware -> Layout middleware -> Route middleware。

3. `asyncData`

   在组件渲染前运行，在客户端和服务端都可以执行。用于异步获取数据在渲染，通常和服务端配合使用。

4. `fetch`

   组件实例化之后，在页面加载前执行。主要用于客户端获取数据，不支持服务端渲染。

5. `created`

   在组件实例化后立即被调用，在服务器端和客户端均可用。常用于初始化数据请求和事件监听器等任务。

6. `mounted`

   在组件挂载到页面上后调用，只能在客户端中使用。适合进行 DOM 操作和使用第三方库初始化页面。

7. `updated`

   在组件更新时被调用，只能在客户端中使用。常用于更新数据和操作 DOM。

8. `beforeDestory`

   在组件被销毁之前调用，适合做一些清理工作或者事件解绑定等任务。

## 路由

nuxt 将根据 pages 目录下文件自动生成路由配置。

### 基础路由

文件树：

```text
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
```

自动生成路由配置：

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

### 动态路由

类似原 vue-router 中 `/:id`，在 nuxt 中通过 `_id.vue` 这种下划线命名方式实现。

文件树：

```text
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```

自动生成：

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

#### 路由参数

在路由组件本地可通过 `this.$route.params.parameterName`。

### 嵌套路由

即子路由。nuxt 中通过创建于子目录同名的 vue 文件实现。

> 注意：需要在父级添加 `<NuxtChild/>` ；来显示子组件内容。

文件树：

```text
pages/
--| users/
-----| _id.vue
-----| index.vue
--| users.vue
```

自动生成：

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

### 路由拓展

#### [@nuxt.js/router](https://github.com/nuxt-community/router-module)

该方式可以使用自己的路由配置来覆盖重写 nuxt 自动生成的路由配置。

具体使用参考官方库。

> 注意：我们的 router.js 需要暴露 createRouter 出去。

#### [router.extendRoutes](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/#extendroutes)

通过 nuxt.config.js 进行拓展路由。

添加路由：

```js
export default {
  router: {
    extendRoutes(routes, resolve) {
      routes.push({
        name: 'custom',
        path: '*',
        component: resolve(__dirname, 'pages/404.vue')
      })
    }
  }
}
```

路由排序：

```js
import { sortRoutes } from '@nuxt/utils'
export default {
  router: {
    extendRoutes(routes, resolve) {
      // Add some routes here ...

      // and then sort them
      sortRoutes(routes)
    }
  }
}
```

### 路由配置

路由配置在 nuxt.config.js 中的 router 模块。

1. [base: string](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/#base)

   路由根路径。

2. [extendRoutes: function](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/#extendroutes)

   路由拓展。

3. [fallback: boolean](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/#fallback)

   当浏览器不支持 history 模式时，是否回退使用 hash 模式。

4. [mode: 'hash' | 'history'](https://v2.nuxt.com/docs/configuration-glossary/configuration-router/#mode)

   配置路由模式，但不建议修改，因为会影响到服务端渲染。

5. [scrollBehavior](https://v3.router.vuejs.org/guide/advanced/scroll-behavior.html#async-scrolling)

   路由跳转后配置滚动条行为。

## 数据获取

nuxt 支持传统的数据获取方式，例如 vue 的 mounted 周期中获取数据。但只有使用 nuxt 特有的 hook 才能在服务器端渲染期间呈现数据。

nuxt 获取数据的钩子有：asyncData、fetch。

### 执行周期

`fetch` 钩子函数和 `asyncData` 钩子函数都是用于在组件渲染前获取异步数据的钩子函数，但它们的调用周期和触发时机不完全一样。

在 Nuxt.js 应用程序中，`fetch` 钩子函数会在服务端渲染（SSR）期间和客户端渲染（CSR）期间分别被调用。具体来说：

- 在 SSR 期间，`fetch` 钩子函数会在服务器端组件实例化之后、页面渲染之前被调用，此时可以获取到异步数据并将其保存到 Vuex store 中，以便在页面渲染时使用。
- 在 CSR 期间，`fetch` 钩子函数会在每次路由导航切换时被调用，此时也可以获取异步数据并更新 Vuex store 中的状态。

而对于 `asyncData` 钩子函数，则仅在客户端渲染时被调用，在服务端渲染期间不会执行。具体来说：

- 在 CSR 期间，`asyncData` 钩子函数会在组件实例化之前被调用，并且会等待 Promise 对象被解决后再继续页面的渲染过程。这个阶段可以理解为初始化前。
- 在组件实例化完成之后，会进入`created`生命周期钩子函数，此时组件的 DOM 节点已经生成，但是尚未被渲染到页面上。
- 等到组件的 `mounted` 生命周期钩子函数调用时，组件的 DOM 节点已经成功地渲染到了页面上。

因此，在 Nuxt.js 应用程序中，如果需要在组件渲染前就获取异步数据并使用，应该优先考虑使用 `fetch` 钩子函数；如果异步数据获取与页面渲染无关，可以考虑使用 `asyncData` 钩子函数。

### fetch

在渲染页面前填充状态树(store)数据，不会设置组件的数据，可应用在所有组件上。

其还提供了 this.$fetchState 用于获取 fetch 状态，并在页面展示相关信息。

$fetchState 示例：

```html
<template>
  <div>
    <p v-if="$fetchState.pending">Fetching mountains...</p>
    <p v-else-if="$fetchState.error">An error occurred :(</p>
    <div v-else>
      <h1>Nuxt Mountains</h1>
      <ul>
        <li v-for="mountain of mountains">{{ mountain.title }}</li>
      </ul>
      <button @click="$fetch">Refresh</button>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        mountains: []
      }
    },
    async fetch() {
      this.mountains = await fetch(
        'https://api.nuxtjs.dev/mountains'
      ).then(res => res.json())
    }
  }
</script>
```

监听路由参数信息，进行调用 fetch 更新数据：

```js
export default {
  watch: {
    '$route.query': '$fetch'
  },
  async fetch() {
    // Called also on query changes
  }
}
```

同样，在 `<nuxt>` 组件上使用 `keep-alive` 可进行组件数据缓存。

nuxt 还提供了 `this.$fetchState.timestamp` 用于记录最后一次 fetch 的时间戳，可在缓存组件超过一定时间后，进行数据刷新。

```html
<template> ... </template>

<script>
  export default {
    data() {
      return {
        posts: []
      }
    },
    activated() {
      // Call fetch again if last fetch more than 30 sec ago
      if (this.$fetchState.timestamp <= Date.now() - 30000) {
        this.$fetch()
      }
    },
    async fetch() {
      this.posts = await fetch('https://api.nuxtjs.dev/posts').then(res =>
        res.json()
      )
    }
  }
</script>
```

### asyncData

在服务器端获取渲染数据，并返回给当前组件的 data，且不能访问到 this 以及 组件实例。不同于 fetch，该钩子函数只能应用于组件页面。

```html
<template>
  <div>
    <h1>{{ post.title }}</h1>
    <p>{{ post.description }}</p>
  </div>
</template>

<script>
  export default {
    async asyncData({ params, $http }) {
      const post = await $http.$get(`https://api.nuxtjs.dev/posts/${params.id}`)
      return { post }
    }
  }
</script>
```

## Meta 和 SEO

### 全局配置

nuxt.config.js -> head。

```js
export default {
  head: {
    title: 'my website title',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      {
        hid: 'description',
        name: 'description',
        content: 'my website description'
      }
    ],
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }]
  }
}
```

### 局部配置

对象形式：

```js
<script>
export default {
  head: {
    title: 'Home page',
    meta: [
      {
        hid: 'description',
        name: 'description',
        content: 'Home page description'
      }
    ],
  }
}
</script>
```

函数形式：

```html
<template>
  <h1>{{ title }}</h1>
</template>
<script>
  export default {
    data() {
      return {
        title: 'Home page'
      }
    },
    head() {
      return {
        title: this.title,
        meta: [
          {
            hid: 'description',
            name: 'description',
            content: 'Home page description'
          }
        ]
      }
    }
  }
</script>
```

### 外部资源

通过 head.script 引入外部资源。

```js
export default {
  head: {
    script: [
      {
        src: 'https://cdnjs.cloudflare.com/ajax/libs/jquery/3.1.1/jquery.min.js'
      }
    ],
    link: [
      {
        rel: 'stylesheet',
        href: 'https://fonts.googleapis.com/css?family=Roboto&display=swap'
      }
    ]
  }
}
```

## 配置

### css

nuxt 中允许通过配置文件配置全局 css。

```js
export default {
  css: [
    // Load a Node.js module directly (here it's a Sass file)
    'bulma',
    // CSS file in the project
    '~/assets/css/main.css',
    // SCSS file in the project
    '~/assets/css/main.scss'
  ]
}
```

并且，nuxt 支持你忽略文件后缀名。但存在优先级：['css', 'pcss', 'postcss', 'styl', 'stylus', 'scss', 'sass', 'less']

> 同名文件，将按照该顺序解析。

### 外部资源

主要通过 head 中的 script 以及 link 实现。

```js
export default {
  head: {
    script: [
      {
        src: 'https://cdnjs.cloudflare.com/ajax/libs/jquery/3.1.1/jquery.min.js'
      }
    ],
    link: [
      {
        rel: 'stylesheet',
        href: 'https://fonts.googleapis.com/css?family=Roboto&display=swap'
      }
    ]
  }
}
```

### PostCSS

nuxt 配置可替代 postcss.config.js。

```js
export default {
  build: {
    postcss: {
      // Add plugin names as key and arguments as value
      // Install them before as dependencies with npm or yarn
      plugins: {
        // Disable a plugin by passing false as value
        'postcss-url': false,
        'postcss-nested': {},
        'postcss-responsive-type': {},
        'postcss-hexrgba': {}
      },
      preset: {
        // Change the postcss-preset-env settings
        autoprefixer: {
          grid: true
        }
      }
    }
  }
}
```

### JSX

nuxt 通过 [@nuxt/babel-preset-app](https://github.com/nuxt/nuxt/tree/2.x-dev/packages/babel-preset-app)，实现 JSX 的编译渲染。

```js
export default {
  data () {
    return { name: 'World' }
  },
  render (h) {
    return <h1 class="red">{this.name}</h1>
  }
}
```

### Ignore files

**.nuxtignore**

您可以使用 .nuxtignore 文件来让 Nuxt 在构建阶段忽略项目根目录中的布局、页面、存储和中间件文件。

.nuxtignore 文件遵循与 .gitignore 和 .eslintignore 文件相同的规范，其中每行都是一个 glob 模式，指示应忽略哪些文件。

```markdown
# ignore layout foo.vue

layouts/foo.vue

# ignore layout files whose name ends with -ignore.vue

layouts/*-ignore.vue

# ignore page bar.vue

pages/bar.vue

# ignore page inside ignore folder

pages/ignore/*.vue

# ignore store baz.js

store/baz.js

# ignore store files match _.test._

store/ignore/_.test._

# ignore middleware files under foo folder except foo/bar.js

middleware/foo/*.js !middleware/foo/bar.js
```

同样，你也可以通过特殊的命名方式来实现忽略文件：`-xxx.xxx`

在配置文件中也可以通过 ignore 模块配置忽略文件。

### webpack 配置

nuxt 通过配置文件下 build 模块进 行配置 webpack，build 是一个对象，对象中包含一个 extend 函数。

该函数接收两个参数，第一个参数是从 nuxt 的 webpack 配置中导出的 webpack 配置对象。第二个参数是一个带有以下布尔属性的上下文对象：`{ isDev, isClient, isServer, loaders }`。

```js
export default {
  build: {
    extend(config, { isClient }) {
      // Extend only webpack config for client-bundle
      if (isClient) {
        config.devtool = 'source-map'
      }
    }
  }
}
```

#### 自定义 chunks

```js
export default {
  build: {
    extend(config, { isClient }) {
      if (isClient) {
        config.optimization.splitChunks.maxSize = 200000
      }
    }
  }
}
```

#### 查看webpack配置

通过 `nuxt webpack` 命令查看最终的 webpack 配置。

#### 添加webpack plugins

nuxt 通过 build 模块下 `plugins: Object` 进行添加。

```js
import webpack from 'webpack'

export default {
  build: {
    plugins: [
      new webpack.ProvidePlugin({
        // global modules
        $: 'jquery',
        _: 'lodash'
      })
    ]
  }
}
```

#### 处理音频文件

音频文件应该由 `file-loader` 进行处理，但默认并没有进行配置。

```js
export default {
  build: {
    extend(config, ctx) {
      config.module.rules.push({
        test: /\.(ogg|mp3|wav|mpe?g)$/i,
        loader: 'file-loader',
        options: {
          name: '[path][name].[ext]'
        }
      })
    }
  }
}
```

这里配置完之后，就可以直接通过 require 直接引入音频文件提供给 audio 标签使用：`<audio :src="require('@/assets/water.mp3')" controls></audio>` 

若不想使用 require 可通过配置 `vue-loader` 自动加载。

```js
export default {
  build: {
    loaders: {
      vue: {
        transformAssetUrls: {
          audio: 'src'
        }
      }
    },

    extend(config, ctx) {
      config.module.rules.push({
        test: /\.(ogg|mp3|wav|mpe?g)$/i,
        loader: 'file-loader',
        options: {
          name: '[path][name].[ext]'
        }
      })
    }
  }
}
```

### 修改主机和端口号

在配置文件下的 server 模块可进行修改 host 和 port。

```js
export default {
  server: {
    host: '0' // default: localhost
    port: 8000 // default: 3000
  }
}
```

但不建议这样做，因为可能在网站托管时引起问题。最好直接在 dev 命令中修改主机和端口号。

```json
"scripts": {
  "dev:host": "nuxt --hostname '0' --port 8000"
}
```

### 异步配置

nuxt 可将通过异步函数返回值的形式进行配置。

```js
import axios from 'axios'

export default async () => {
  const data = await axios.get('https://api.nuxtjs.dev/posts')
  return {
    head: {
      title: data.title
      //... rest of config
    }
  }
}
```

## 加载条

### 自定义进度条

nuxt 配置文件的 loading 模块下进行配置。

| Key         | Type    | Default | Description                                                  |      |
| ----------- | ------- | ------- | ------------------------------------------------------------ | ---- |
| color       | String  | 'black' | CSS color of the progress bar                                |      |
| failedColor | String  | 'red'   | CSS color of the progress bar when an error appended while rendering the route (if data or fetch sent back an error, for example). |      |
| height      | String  | '2px'   | Height of the progress bar (used in the style property of the progress bar) |      |
| throttle    | Number  | 200     | In ms, wait for the specified time before displaying the progress bar. Useful for preventing the bar from flashing. |      |
| duration    | Number  | 5000    | In ms, the maximum duration of the progress bar, Nuxt assumes that the route will be rendered before 5 seconds. |      |
| continuous  | Boolean | false   | Keep animating progress bar when loading takes longer than duration. |      |
| css         | Boolean | true    | Set to false to remove default progress bar styles (and add your own). |      |
| rtl         | Boolean | false   | Set the direction of the progress bar from right to left.    |      |

### 禁用进度条

通过 `loading: false` 禁用进度条，无论是全局配置还是局部配置均可。

### 编程式调用

nuxt 提供 `$loading` 进行直接控制进度条。

```js
export default {
  mounted() {
    this.$nextTick(() => {
      this.$nuxt.$loading.start()
      setTimeout(() => this.$nuxt.$loading.finish(), 500)
    })
  }
}
```

## 内置组件

### Nuxt

用于显示路由组件，只能用于布局文件(layouts)中使用。

```html
/** layouts/default.vue */
<template>
  <div>
    <div>My nav bar</div>
    <Nuxt :nuxt-child-key="somekey"/>
    <div>My footer</div>
  </div>
</template>
```

`:nuxt-child-key` 将传递给 `<RouterView>`，以便在动态页面正确处理过渡效果。

### NuxtChild

用于显示子路由(嵌套路由)。

```text
-| pages/
---| parent/
------| child.vue
---| parent.vue
```

```html
<template>
  <div>
    <h1>I am the parent view</h1>
    <NuxtChild :foobar="123" />
  </div>
</template>
```

### keep-alive

作用于 `<NuxtChild>` 和 `<Nuxt>`，进行组件缓存。

```html
<template>
  <div>
    <Nuxt keep-alive :keep-alive-props="{ exclude: ['modal'] }" />
  </div>
</template>

<!-- will be converted into something like this -->
<div>
  <KeepAlive :exclude="['modal']">
    <RouterView />
  </KeepAlive>
</div>
```

`keep-alive-props` 是原 keep-alive 上的一些属性，包括：exclude、include等。

### NuxtLink

路由跳转。

```html
<template>
  <NuxtLink to="/">Home page</NuxtLink>
</template>
```

在 nuxt 中 NuxtLink 是默认**预取**的，也就是说在浏览器空闲的时候，将会预取并加载资源，以便用户点击链接时他们已经准备好。

当然你也可以关闭 prefetch。

局部关闭：

```html
<NuxtLink to="/about" no-prefetch>About page not pre-fetched</NuxtLink>
<NuxtLink to="/about" :prefetch="false">About page not pre-fetched</NuxtLink>
```

全局关闭：

```js
export default {
  router: {
    prefetchLinks: false
  }
}
```

局部开启：

> Nuxt v2.10.0 之后

```html
<NuxtLink to="/about" prefetch>About page pre-fetched</NuxtLink>
```

### linkActiveClass

nuxt 默认链接激活类名是：`nuxt-link-active`

当然你也可以自定义：

```js
export default {
  router: {
    linkActiveClass: 'my-custom-active-link'
  }
}
```

### client-only

显示仅在客户端渲染的组件。

```html
<template>
  <div>
    <sidebar />
    <client-only placeholder="Loading...">
      <!-- this component will only be rendered on client-side -->
      <comments />
    </client-only>
  </div>
</template>
```

使用插槽：

```html
<template>
  <div>
    <sidebar />
    <client-only>
      <!-- this component will only be rendered on client-side -->
      <comments />

      <!-- loading indicator, rendered on server-side -->
      <template #placeholder>
        <comments-placeholder />        
      </template>
    </client-only>
  </div>
</template>
```

> 有时在服务器渲染的页面中，即使使用 $nextTick，`<client-only>` 中的 $refs 也可能还没有准备好。技巧可能是多次调用 $nextTick：
>
> ```js
> mounted(){
>   this.initClientOnlyComp()
> },
> methods: {
>   initClientOnlyComp(count = 10) {
>     this.$nextTick(() => {
>       if (this.$refs.myComp) {
>         //...
>       } else if (count > 0) {
>         this.initClientOnlyComp(count - 1);
>       }
>     });
>   },
> }
> ```

> 在 Nuxt < v2.9.0 中可以使用 `<no-ssr>` 替代 `<client-only>`

## 非路由组件

### 自动导入

nuxt 中支持自动引入，在全局配置的 component 属性进行配置。

```js
export default {
  components: true
}
```

### 组件命名

非路由组件的命名是基于文件夹名和文件名生成的。

```bash
| components/
--| base/
----| foo/
------| Button.vue

/** 最终的组件名 */
<BaseFooButton />
```

> 建议还是将最终的组件名提前根据文件夹命名一致。

忽略某个文件夹对命名的影响，可通过 component 下的 dirs 进行配置：

```bash
components: {
  dirs: [
    '~/components',
    '~/components/base'
  ]
}
```

### 懒加载

nuxt 中懒加载直接在组件前面加上 Lazy 即可。

懒加载的组件在打包的过程中将会被单独分块儿打包，这将优化打包的大小。

```html
<template>
  <div>
    <h1>Mountains</h1>
    <LazyMountainsList v-if="show" />
    <button v-if="!show" @click="show = true">Show List</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: false
    }
  }
}
</script>
```

## 过渡

作用于路由，过渡动效。

有三种写法：string、object、function。

默认的全局 transition-name 为 page。

### string

string 类型主要配置 transition 的 name，后续根据 .name-enter-active 等类名编写 css 动效。

```vue
<script>
export default {
  transition: 'home'
}
</script>

<transition name="home"></transition>

<style>
  .home-enter-active, .home-leave-active { transition: opacity .5s; }
  .home-enter, .home-leave-active { opacity: 0; }
</style>
```

### object

对象配置的属性相对 string 就多了一些，比如：name、mode、duration等等，以及可以调用一些[钩子函数](https://v2.vuejs.org/v2/guide/transitions.html#JavaScript-Hooks)。

```js
export default {
  transition: {
    name: 'home',
    mode: 'out-in'
  }
}

export default {
  transition: {
    afterLeave(el) {
      console.log('afterLeave', el)
    }
  }
}
```

### function

能够获取到路由信息：to、from。

```js
export default {
  transition(to, from) {
    if (!from) {
      return 'slide-left'
    }
    return +to.query.page < +from.query.page ? 'slide-right' : 'slide-left'
  }
}
```

### 全局配置

nuxt.config.js 的全局过渡配置主要有俩：layoutTransition、pageTransition。

写法和上述一致。

```js
export default {
  layoutTransition: 'my-layouts'
  // or
  layoutTransition: {
    name: 'my-layouts',
    mode: 'out-in'
  }
}

export default {
  pageTransition: 'my-page'
  // or
  pageTransition: {
    name: 'my-page',
    mode: 'out-in',
    beforeEnter (el) {
      console.log('Before enter...');
    }
  }
}
```

## 中间件

nuxt 中可定义一些自定义函数作为中间件，在（一组）页面渲染前执行。

共享的中间件应当放置在 `/middleware` 下，文件名则是中间件的名称。

每一个中间件函数可接收一个 context 参数。

通常情况下，中间件将在首次服务端渲染（第一次请求 nuxt app 或者 刷新页面）时以及客户端路由跳转时执行。

多个中间件的执行顺序：

1. nuxt.config.js 中的全局中间件
2. layouts 中的中间件
3. pages 中的中间件

### Router Middleware

类似路由守卫，需要在 nuxt.config.js 中 router 模块下配置。

```js
/** middleware/stats.js */
import http from 'http'

export default function ({ route }) {
  return http.post('http://my-stats-api.com', {
    url: route.fullPath
  })
}

/** nuxt.config.js */
export default {
  router: {
    middleware: 'stats' // middleware: ["stats"],
  }
}
```

### 匿名中间件

局部中间件，直接在特殊的页面中直接通过 middleware 配置当前页面的中间件。

```html
<template>
  <h1>Secret page</h1>
</template>

<script>
  export default {
    middleware({ store, redirect }) {
      // If the user is not authenticated
      if (!store.state.authenticated) {
        return redirect('/login')
      }
    }
  }
</script>
```

