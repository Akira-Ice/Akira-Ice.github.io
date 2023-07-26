---
title: NPM & YARN 常用命令
date: 2023/7/25
categories:
  - [BugRoad]
tags: 
  - BugRoad
---

# NPM & YARN 常用命令

| Command                        | Description  |
|:------------------------------ |:------------ |
| `npm view packageName version` | 查看包的可安装版本    |
| `npm init -y`                  | yarn init -y |
| `npm i xxx -d `                | yarn add     |
| `npm i xxx -s`                 | yarn         |
| `npm i xxx -g`                 | 全局安装依赖       |
| `npm i xxx --force`            | 忽略上游冲突，覆盖依赖  |
| `npm i xxx --legacy-peer-deps` | 忽略上游冲突，不覆盖依赖 |
| `npm config get prefix`        | 查看全局安装路径     |