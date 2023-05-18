---
title: 阿里云培训 - Linux基础
categories:
  - [work,Eccom]
tags: 
hidden: true
---

# Linux基础

## 常见命令

### 查看文件

#### cat

用于连接文件并打印到标准输出设备上

`cat /etc/issue`

#### more

类似 cat 命令，以分页的方式显示文件内容

`more /etc/issue`

#### less

与 more 类似，但使用 less 可以随意浏览文件

`less /etc/issue`

#### head

用于显示文件的开头至标准输出中

`head -n 20 /etc/fstab`

#### tail

用于显示文件的末端至标准输出中

`tail-n 20 /etc/fstab`

### vim文本编辑器

![image-20230518145505809](https://s2.loli.net/2023/05/18/PGNk7BQWReILlZr.png)

#### 常用模式

- 命令模式
- 插入模式
- ex模式
- 可视模式

### 文本统计

#### wc

用于计算字数

`wc file`

`wc -l file`

`w|wc -l`