---
title: MariaDB创建新用户、新数据库并允许特定用户远程访问
date: 2017.3.11 13:43
categories: Linux
tags:
- MariaDB
- New DB
- Remote
---

### 创建新的可远程访问的数据库用户
```bash
MariaDB [(none)]> create user 'user'@'%' identified by 'pass';
```
### 创建新的数据库
```bash
MariaDB [(none)]> create database newdb;

```
### 把新创建的数据库授权给新创建的用户
```bash
MariaDB [(none)]> grant all privileges on newdb.* to 'user'@'%';
MariaDB [(none)]> flush privileges;
```
