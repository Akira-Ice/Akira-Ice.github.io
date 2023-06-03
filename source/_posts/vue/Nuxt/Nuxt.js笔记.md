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

约定俗成的目录结构 -> 良好的起点

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
