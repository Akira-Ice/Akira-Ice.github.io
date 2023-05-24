---
title: 阿里云培训 - MySQL基础
categories:
  - [work,Eccom]
tags: 
---

# MySQL基础

## MySQL介绍

**关系型数据库**管理系统

体积小、速度快、总体拥有成本低

## MySQL存储引擎

1. MyISAM

   MySQL 5.0 以前默认的数据库引擎，最常用，拥有较高的插入、查询速度，蛋不支持事务。

2. InnoDB

   事务型数据库的首选引擎，支持ACID事务，支持行级锁定，MySQL 5.5 起成为默认数据库引擎。

3. MEMORY

   MEMORY存储引擎提供 “内存表”，也不支持事务、外键。

4. ARCHIVE

   ARCHIVE存储引擎是被设计用来存储企业中的大量流水数据的存储引擎。

## MySQL系统命令

### MySQL服务管理

1. 服务状态查询

   `service mysqld status`

2. 启动服务

   `service mysqld start`

3. 停止服务

   `service mysqld stop`

4. 检查MySQL端口

   `netstat -antulp | grep mysqld`

###  常用指令

1. 获取帮助

   `help(\?)`

2. 选择数据库

   `use(\u)`

3. 执行 SQL 文件

   `source(\.)`

4. 清空当前的输入语句

   `clear(\c)`

### 元数据查询

1. 服务器版本信息

   `SELECT VERSION()`

2. 当前数据库名

   `SELECT DATABASE()`

3. 当前用户名

   `SELECT USER()`

4. 服务器配置变量

   `SHOW VARIABLES`

5. 服务器状态

   `SHOW STATUS`

### MySQL配置文件

1. MySQL配置文件默认路径

   `/etc/my.cnf`

2. 常用配置参数

   `basedir` —— MySQL根目录

   `datadir` —— 数据库文件目录

   `socket` —— 为 MySQL 客户端与服务器之间本地通信指定的一个套接字文件

   `character-set-server` —— 新数据库或数据库的默认字符集

## SQL

`Structured Query Language`

### 语法组成

1. DML

   查询、插入、删除和修改数据库中的数据。

   SELECT、INSERT、UPDATE、DELETE 等。

2. DCL

   用来控制存取许可、存取权限等。

   GRANT、REVOKE 等。

3. DDL

   用来建立数据库、数据库对象和定义其列。

   CREATE TABLE、DROP TABLE、ALTER TABLE 等。

4. 功能函数

   日期函数、数据函数、字符函数、系统函数等。

### 数据库操作

1. 显示数据库

   `show databases;`

2. 选择操作数据库

   `use <dbname>;`

3. 创建数据库

   `create database <dbname> [charset=utf8];`

4. 显示数据库创建语句

   `show create database <dbname>;`

5. 删除数据库

   `drop <dbname>;`
   
6. 显示数据库中创建的所有表

   `show tables;`

7. 显示表结构

   `desc[rive] <tbname>;`

   `show colums from <tbname>;`

8. 现实数据表创建语句

   `show create table <tbname>;`

### MySQL数据类型

#### 数值类型

#### 自增类型 - auto_increment

1. 字段值每次自动增长1。

2. 自增型的整数字段

   只适用于整数，在数据类型后加 auto_increment 关键字表示。如：`smallint unsigned auto_increment;`

注意：
1. mysql 中每个表只能设置一个自增字段。

2. 该列必须是 NOT NULL

3. 该列必须定义唯一索引，如逐渐 primary key 或唯一键 unique key，以避免重复。

4. 该列的最大值受其数据类型约束。如 tinyint 型的最大只为 127，加上 unsigned，则为 255.一单打到上限，auto_increment 就会失效。

#### 小数类型 - decimal

1. 小数可通过 unsigned 设置为非负数。
2. `decimal(m,d)` 表示述职中共有 m 位数，其中整数 m-d 位，小数 d 位。

如： `decimal(5,2)`，取值范围为：-999.99 ~ 999.99。

#### 字符串类型

![image-20230523143815654](https://s2.loli.net/2023/05/23/muMqIx8z27laOZv.png)

1. CHAR：定长字符串
2. VARCHAR：变长字符串
3. TEXT：长文本数据

#### 日期和时间类型

![image-20230523144009500](https://s2.loli.net/2023/05/23/tgZNQjeXYnVP4K7.png)

1. DATE：日期值
2. DATETIME：混合日期和时间值

### 数据表创建

```sql
create table tbname (
	id int auto_increment primary key,
  name varchar(20) not null,
  sex tinyint not null,
  birthday date not null
);
```

### 临时表

保存一些临时数据时使用的表

`create TEMPORARY tbname (xxx);`

特性：

1. 临时表只在**当前**连接可见
2. 当关闭连接时，MySQL会**自动删除**表并释放所有空间。

### 数据操作

**CRUD**

#### 新增数据

`INSERT [INFO] <表名>(<列名>) VALUES(<值列表>)；`

#### 查询数据

```
SELECT [DISTINCET | DISTINCTROW | ALL] select_expression [
	FROM table_references //指定查询数据的表
		[WHERE where_definition] //查询数据的过滤条件
		[GROUP BY col_name,...] // 对匹配 where 子句的查询结果进行分组
		[HAVING where_definition] //对分组后的结果进行条件限制
		[ORDER BY {unsigned_integer | col_name | formula} [ASC | DESC],...] // 对查询结果进行排序
		[LIMIT [offset] rows] //对查询的显示结果限制数目
		[PROCEDURE procedure_name] //查询存储过程返回的结果集数据
]
```

##### SQL集函数

主要集函数：

1. 记数函数：`count(列名)` 计算元素的个数。
2. 求和函数：`sum(列名)` 对某一列的值求和，但属性必须是整数。
3. 计算平均值：`avg(列名)` 对某一列的值计算平均值。
4. 求最大值：`max(列名)` 找出某一列的最大值。
5. 求最小值：`min(列名)` 找出某一列最小值。

##### WHERE

谓语：

1. `BETWEEN AND`：在两数之间。
2. `NOT BETWEEN AND`：不在两数之间。
3. `IN <值表>`：是否在特定的集合里面（枚举）。
4. `NOT IN <值表>`：与上面相反。
5. `LIKE`： 是否匹配与一个模式。
6. `IS NULL（为空）或 IS NOT NULL（不为空） REGEXP`：检查一个值是否匹配一个常规表达式。

#### 多表查询

1. 连接查询

   同时涉及多个表的查询称为连接查询

   用来连接两个表的条件称为连接条件

2. 连接查询的方式

   1. 内连接
   2. 外连接
      1. 左外连接
      2. 右外连接
   3. 查询结果合并
   4. 子查询

##### 内连接（INER JOIN）



```sql
SELECT S.SName,C.score 
From Score AS C 
INNER JOIN Students AS S 
ON C.Student_id = S.student_id
## or
SELECT S.SName,C.score 
From Score AS C, Students AS S
Where c.Student_id = S.Student_id;
```

##### 外连接 - 左连接

以一张表为中心，往外找，找到返回，未找到返回`null`。

做外连以前面的表作为主表，返回所有主表中的所有行。

```sql
SELECT sname,number 
FROM student AS S
LEFT JOIN score AS C
ON S.sid = C.student_id;
```

##### 外连接 - 右连接

```sql
SELECT sname,number 
FROM student AS S
RIGHT JOIN score AS C
ON S.sid = C.student_id;
```

##### 合并查询数据记录

```sql
SELECT * FROM t1
UNION
SELECT * FROM t2
```

##### 子查询

通俗说，就是嵌套查询。

```sql
SELECT sname FROM student
WHERE age > (SELECT age FROM student WHERE sname='赵六');
```

#### 修改数据

语法：`UPDATE <表名> SET <列名 = 更新值> [WHERE <更新条件>]`

#### 数据删除

语法：`DELETE FROM <表名> [WHERE <更新条件>]`

> **生产环境禁止使用 UPDATE 和 DELETE**，且在使用时需要些 WHERE，否则将修改整张表的数据，且无法修复。

## 日志查看

1. 错误日志

   记录 MySQL 服务器启动、关闭和运行时出错等信息。

2. 查询日志

   记录 MySQL 服务器的启动和关闭信息、客户端的连接信息、更新数据库记录 SQL 语句和查询数据库记录 SQL 语句。

3. 慢查询日志

   记录执行时间超过指定时间的查询语句，通过工具分析慢查询日志开一定位 MySQL 服务器性能瓶颈所在。

4. 二进制日志

   以二进制形式记录数据库的各种操作，但不记录查询语句。

### 错误日志

日志路径：`mysql> show variables like 'log_error'`

默认路径为：`/var/log/mysqld.log`

### 慢查询日志

日志路径：`show variables like 'slow_query_log%'`

![image-20230524152158771](https://s2.loli.net/2023/05/24/dj58HGVAXLJFlZo.png)

## 备份还原

### 数据备份

`mysqldump` 是 mysql 用于转存储数据库的实用程序。

产生一个 SQL 脚本。

语法：`mysqldump -u root -p Password [databaseName]`

> 值备份所有表，不包括数据库本身。

### 数据还原

1. mysql 命令导入

   `mysql -u root -p password < sql文件`

2. source

   `source sql文件`

