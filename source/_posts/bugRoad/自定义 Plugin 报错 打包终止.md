---
title: Bug Road
categories:
  - [BugRoad]
tags: 
  - plugin
---

# 自定义 Plugin 报错 打包终止

## 🤔发现问题

```bash
(node:9776) [DEP_WEBPACK_COMPILATION_ASSETS] DeprecationWarning: Compilation.assets will be frozen in future, all modifications are deprecated.BREAKING CHANGE: No more changes should happen to Compilation.assets after sealing the Compilation.Do changes to assets earlier, e. g. in Compilation.hooks.processAssets.Make sure to an appropriate stage from Compilation.PROCESS_ASSETS_STAGE_*. Use `node --trace-deprecation ...` to show where the warning was created)
```

## 🙅‍♂️拒绝妥协，坚持解决

  意思是： `Compilation.assets will be frozen in future`  就是 `Compilation.assets` 将来的`webpack`的版本中会被冻结，建议在 `compilation` 的 `seal` 阶段去处理资源。（也就是说可以在，资源压缩之后，冻结之前，对资源进行处理，但是`compilation`可能会触发多次）

`compiler.hooks.emit.tapAsync("pluginName", (compilation) => {})`

  打包终止原因是 异步钩子，需要调用回调，才会继续往下执行

- 改用 `tap` 定义钩子
- 或者添加 `callback`，并调用

  之后就可以打包成功了，但可能打包之后仍然会有警告，这里可以暂时忽略