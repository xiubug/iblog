---
title: MySQL数据库自增主键归零的几种方法
tags:
  - rzero
categories:
  - mysql
date: 2018-11-02 15:01:48
---

MySQL自增主键归零的方法：

### 一、如果曾经的数据都不需要的话，可以直接清空所有数据，并将自增字段恢复从1开始计数：
```js
truncate table table_name;
```
### 二、当用户没有truncate的权限时且曾经的数据不需要时：
* 1、删除原有主键：

```js
alter table 'table_name' drop 'id';
```
* 2、添加新主键：

```js
alter table 'table_name' add 'id' int(11) not null first;
```
* 3、设置新主键：

```js
alter table 'table_name' modify column 'id' int(11) not null auto_increment, add primary key(id);
```
### 三、当用户没有权限时：
* 1、可以直接设置数据表的 AUTO_INCREMENT 值为想要的初始值，比如10000：
```js
alter table 'table_name' auto_increment = 10000;
```
