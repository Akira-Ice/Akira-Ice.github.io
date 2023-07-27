---
title: VueRouter 传递 state 参数无效
date: 2022/9/20
updated: 2022/9/20
categories:
  - [BugRoad, Vue3]
tags: 
  - VueRouter
---

# VueRouter 传递 state 参数无效

## 什么是 state ？

在浏览器中，`history.state` 是 `history` 对象的一个属性，用于访问历史记录中当前页面的状态对象。当使用浏览器的前进或后退按钮时，可以通过 `history.state` 获取前一个页面或后一个页面的状态对象，从而实现前端页面之间状态的传递和管理。

自从 `Vue3` 中的 `VueRouter` 中将 `params` 大削弱。`state` 便走上了饭桌。

```js
const router = useRouter();
router.push({name: 'xxx', state: { user: {...} } })

history.state.user /
```

## 为什么会无效？

`报错：Error with push/replace State DOMException: Failed to execute 'pushState' on 'History': <Object> could not be cloned.`

通常是传递的数据是 响应式数据 造成的，也就是 `proxy` 类型。

通过 `toRaw` 还原即可。
