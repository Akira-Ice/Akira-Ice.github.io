---
title: map 转换成对象数组

categories:
  - [BugRoad]
tags: 
  - BugRoad
---

# map 转换成对象数组

## 🤔发现问题

`map => key-value`

通常使用到的 `Array.from` 转换，会得到`[[key, value],....]`

## 🙅‍♂️拒绝妥协，坚持解决

> `Array.from` 有一个回调函数，可以进一步把控数据的流向以及类型。

`Array.from(map, ([key, value]) => ({ key, value }));`