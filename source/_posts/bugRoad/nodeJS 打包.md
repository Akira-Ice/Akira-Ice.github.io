---
title: nodeJS 打包
categories:
  - [BugRoad]
tags: 
  - BugRoad
---

# nodeJS 打包

## 全局安装

`npm i pkg -g`

## 配置 package.json

(1) bin 指定启动文件

(2) script 配置命令

(3) `pkg .` 寻找指定目录下的`package.json`文件，然后在找`bin`字段作为入口文件

(4) `-t` 指定打包平台

(5) `-o` 指定输出文件名

```json
{
...,
"bin": "./app.js",
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "build": "pkg . -t node12-win-x64 -o server -d"
},
...
}
```

## 打包后路径区别

![](https://s2.loli.net/2023/04/01/odRy1kBJiPm4zGZ.png)

## node环境下载慢

pkg 打包需要特殊的 node 环境，会去 node pkg 缓存中去找，找不到就去 github 下载。

每次打包的时候会显示下载的版本，可以去 github 先下载好，然后放到缓存去，缓存大致位置在：`C:\Users\lenovo\.pkg-cache`，可以用 `everything` 搜 `.pkg-cache`。

[pkg资源下载](https://github.com/vercel/pkg-fetch/releases)

下载版本的名字可能会有不同，下载的是 `node-v8.17.0-win-x64`，但是需要的可能是`fetch-v8.17.0-win-x64`,修改一下名字即可。