---
title: keep-alive 缓存滚动条
categories:
  - [BugRoad]
tags: 
  - BugRoad
---

# keep-alive 缓存滚动条

## 常规使用

通常是用来存储组件状态。

在 Vue3 中，可以使用 `keep-alive` 组件来缓存组件的状态和数据，从而实现在页面切换时保留数据的效果。

`keep-alive` 组件可以将需要缓存的组件包裹起来，并提供一个 `include` 属性来指定哪些组件需要缓存。例如，如果需要缓存一个名为 `MyComponent` 的组件，可以这样写：

```html
// include 通常是组件的名称
<keep-alive :include="['MyComponent']">
  <router-view></router-view>
</keep-alive>
```

这样，当 `MyComponent` 组件被切换出去时，其状态和数据将会被缓存起来，并在切换回来时自动恢复。如果需要缓存所有组件，可以将 `include` 属性设置为 `true`。

需要注意的是，`keep-alive` 组件只会缓存被包裹的组件的状态和数据，而不会缓存路由的状态和数据。如果需要缓存整个路由的状态和数据，可以考虑使用 `vuex` 来管理应用的状态。

> 注意在 `Vue3` 中这种方式会 `warning`，需要通过 `v-slot` 的形式使用

```vue
<keep-alive>
  // Component 是 router-view 自带的属性
  <router-view :include="['MyComponent']" v-slot="{ Component }">
    <component :is="Component"></component>
  </router-view>
</keep-alive>
```

`keep-alive` 组件除了 `include` 属性之外，还有一个对立的属性 `exclude`，可以用来指定不需要缓存的组件。

`exclude` 属性接受一个字符串或一个数组作为参数，用来指定不需要缓存的组件名称。

## 能否缓存滚动条呢？

`keep-alive` 组件是用来缓存组件的状态和数据的，它并不会缓存页面的滚动条位置。因此，在使用 `keep-alive` 缓存组件时，如果需要在切换路由时保持滚动条位置不变，需要手动记录和恢复滚动条位置。

一般来说，可以在组件的 `activated` 和 `deactivated` 钩子函数中记录和恢复滚动条位置。例如，在 `activated` 钩子函数中记录滚动条位置：

```javascript
export default {
  activated() {
    // 记录滚动条位置
    this.$scrollPosition = document.documentElement.scrollTop || document.body.scrollTop;
  }
}
```

在这个示例中，`activated` 钩子函数中记录了滚动条位置，将其保存在组件的 `$scrollPosition` 属性中。

在 `deactivated` 钩子函数中恢复滚动条位置：

```javascript
export default {
  deactivated() {
    // 恢复滚动条位置
    document.documentElement.scrollTop = document.body.scrollTop = this.$scrollPosition || 0;
  }
}
```

在这个示例中，`deactivated` 钩子函数中恢复了滚动条位置，将滚动条位置设置为组件的 `$scrollPosition` 属性保存的值。

需要注意的是，由于不同的浏览器可能会有不同的滚动条实现方式，因此在记录和恢复滚动条位置时需要考虑浏览器的兼容性。同时，需要在组件的 `beforeDestroy` 钩子函数中清理保存的滚动条位置，以避免对下次渲染产生影响。
