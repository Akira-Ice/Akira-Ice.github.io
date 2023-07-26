---
title: 阿里云培训 - Linux基础
date: 2023/02/22
categories:
  - [work,Eccom]
tags: 
  - 阿里云培训
  - Linux
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

### 删除文本重复行

#### uniq

用于检查及删除文本文件中重复出现的行列

`uniq testfile` 删除重复行

`uniq -c testfile` 删除并统计重复的行数

`uniq -d testfile`只显示重复的行数

### 文本分析

#### awk

处理文本文件的语言，强大的文本分析工具。

查看个挂载点的可用空间：`df -h | awk 'print 1234'`

查看每个用户的shell：` awk -F ':' '{print $1, $7}' /etc/pawword`

查看长度大于80字节的日志信息：`awk 'length > 80' /var/log/messages`

### 文件查找

#### find

`find . -name 'c'`

`find . -type f`

`find . -ctime -20`

### 查看当前目录

#### pwd

显示工作目录

`pwd [-L]`

`pwd -p`

#### pwdx

查看进程运行的目录

`pwdx PID`

### 挂载文件系统

### mount

挂在Linux系统外的文件。

`mount` 查看当前所有的挂载

`mount /dev/sdb1/mnt`

`mount -o ro /dev/sdb1/mnt`

`mount -o loop /tmp/`

### 进程状态

#### ps

显示当前进程的状态

`ps -ef | grep http`

`ps aux | grep http`

`ps -A`

`ps -u root`

#### top

实时查看进程状态

`top -p PID`
实时查看某一个进程

### 进程管理

#### kill

`kill PID`

`kill -9 PID`

强制结束进程

#### nohup

后台运行进程

`nohup ./start.sh &`

### 查看打开文件

#### lsof

查看进程打开的文件，打开文件的进程，进程打开的端口（TCP、UDP）

`lsof abc.txt`

`lsof -c abc`

`lsof -i 8080`

### 配置主机名

#### hostname

`hostname xxx`

临时修改

### 系统运行时间

#### uptime

查看机器启动运行多久、负载

### 系统监测

#### tsar

`tsar --swap --load`

监控虚拟内存

`tsar --mem`

监控内存

`tsar --io`

监控io

`tsar --traffic`

监控网络

`tsar --check --cpu --io`

监控警告信息

## NTP

网络时间协议

### NTPServer的时间来源

1. GPS、北斗等卫星系统
2. 原子钟
3. 移动基站
4. 恒温晶振

### 安装NTP

#### 检查是否安装

`rpm -q ntp`

#### 安装NTP

`yum install ntp`

### 配置NTP客户端

 配置文件：`/etc/ntp.conf`

### NTP服务管理

服务状态查看：`service ntpd status`

服务启用：`service ntpd start`

服务重启：`service ntpd restart`

服务开启启动：`service ntpd on`

### NTP常用命令

#### 查询NTP服务器时间

`ntpdate -q ntp1.aliyun.com`

#### 检查NTP时间同步情况

`ntpq -p`

## 软件管理

### yum

自动处理依赖关系。

#### 配置文件

`/etc/yum.repos.d/*.repo`

- [名称] - yum源名称
- name - yum源的描述信息
- baseurl - yum源地址
- gpgcheck - 是否检查GPGKEy
- enable - 是否可用

#### 常用命令

- 查看可用包版本
  
  `yum list packname`

- 安装
  
  `yum install packname`

- 搜索
  
  `yum search xx`

- 删除
  
  `yum remove packname`

### 源码编译安装

## DNS 解析

### dig

#### 基本用法

`dig aliyun.com`

#### 查看反向解析

`dig -x 104.205.34.3`

#### 从根服务器开始追踪一个域名的解析过程

`dig +trace aliyun.com`

### nslookup

#### 直接查询

`nslookup aliyun.com`

#### 反向查询

`nslookup ip`

### DNS问题排查思路

- 用户不能访问，运维可访问
  
  - 用户DNS设置问题

- 用户与运维都不能访问
  
  - 检查DNS服务是否能 ping 通
  
  - 查看DNS服务器的53端口是否可用
    
    `telnet <IP> 53`

## 网络配置

### 查看网络

#### ifconfig

用于显示或设置网络设备。

查看活动网卡：`ifconfig`

查看指定网卡：`ifconfig eth0`

查看所有网卡：`ifconfig -a`

#### netstat

查看路由表：`netstat -rn 、 route -n`

查看某端口：`netstat -antulp | grep ：80`

查看所有监听端口：`netstat -antulp | grep LISTEN`

#### 网卡配置文件

文件目录：`/etc/sysconfig/network-scripts/ifcfg-*`

- IPADDR —— IP地址
- NETMASK —— 子网掩码
- GATEWAY —— 网关

#### 全局网络配置

文件目录：`/etc/sysconfig/network`

- HOSTNAME —— 主机名
- GATEWAY —— 默认网关

## 日志查看

### 系统登录日志

登录成功日志：`last [userName]`

登录失败日志：`lastb`

命令记录：`history`

### 常用日志文件

系统启动后的信息和错误日志：`/var/log/message`

安全相关的日志：`/var/log/secure`

定时任务相关的日志：`/var/log/corn`

守护进程启动和停止相关的日志：`/var/log/boot.log`

用户登录、注销及系统启动、停机的事件日志：`/var/log/wtmp`
