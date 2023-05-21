---
layout: post
title: 【Django using MySQL on Windows Server 2016】部署项目过程中遇到的一点问题
# author: 焜_8899
date: 2020-04-21 19:53:42 +0800
last_modified_at: 2020-04-21 20:50:49 +0800
tags: [Django]
toc:  true
---

## 1. 说明

使用
 - Python 3.7.4
 - Django 3.0.5
 - MySQL 8.0
 - Windows Server 2016 数据中心版 64位中文版
 - 腾讯云服务器

## 2. MySQL部分

### 2.1 新建数据库

指定了字符集为utf8，排序规则为utf8_general_ci。

```
mysql> CREATE DATABASE 数据库名 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

### 2.2 导入数据

已提前将所需数据从已有数据库导出到了.sql文件中。

```
mysql> USE 数据库名;
mysql> SOURCE 路径/文件名.sql
```

**注意需要将路径中的“\”改为“/”。**

### 2.3 查看数据

此时，当查看数据时，中文可能还会是乱码。

解决方法：
```
mysql> set names gbk;
```

虽然先前指定了字符集为utf8，但这里仍须设为gbk。

目前还不清楚原因。

## 3. Django部分

在尝试
```
python manage.py runserver
```
的时候，出现报错
```
django.db.utils.OperationalError: (2059, <NULL>)
```

查到原因说
>最新的mysql8.0对用户密码的加密方式为caching_sha2_password，django暂时还不支持这种新增的加密方式。

所以
>只需要将用户加密方式改为老的加密方式即可。

解决方法：
1. 登陆MySQL后查看当前的加密方式
```
mysql> use mysql;
mysql> select user, plugin from user where user='root';
```
2. 修改加密方式
```
mysql> alter user ‘root’@‘localhost’ identified with mysql_native_password by '密码';
```
3. 重新加载权限表使配置生效
```
mysql> flush privileges;
```
4. 修改后可再次执行1.查看修改后的结果。

再次启动Django服务器就没问题了。
