---
layout: post
title:  "Oracle 安全加固-1. Oracle网络配置及加固建议 "
date:   2015-09-04 22:11:00
category: Oracle
tags: [Oracle, Security]
---
* content
{:toc}

一、权限分类：

系统权限：系统规定用户使用数据库的权限。（系统权限是对用户而言)。
实体权限：某种权限用户对其它用户的表或视图的存取权限。（是针对表或视图而言的）。

二、系统权限管理：

1、系统权限分类：

DBA: 拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。
RESOURCE:拥有Resource权限的用户只可以创建实体，不可以创建数据库结构。
CONNECT:拥有Connect权限的用户只可以登录Oracle，不可以创建实体，不可以创建数据库结构。
对于普通用户：授予connect, resource权限。
对于DBA管理用户：授予connect，resource, dba权限。

2、系统权限授权命令：

[系统权限只能由DBA用户授出：sys, system(最开始只能是这两个用户)]

授权命令：`SQL> grant connect, resource, dba to 用户名1 [,用户名2]…;`

[普通用户通过授权可以具有与system相同的用户权限，但永远不能达到与sys用户相同的权限，system用户的权限也可以被回收。]
例：

	SQL> connect system/manager
	SQL> Create user user50 identified by user50;
	SQL> grant connect, resource to user50;

查询用户拥有哪里权限：

	SQL> select * from dba_role_privs;
	SQL> select * from dba_sys_privs;
	SQL> select * from role_sys_privs;

删除用户：

	SQL> drop user 用户名 cascade;  //加上cascade则将用户连同其创建的东西全部删除
 

3、系统权限传递：

增加WITH ADMIN OPTION选项，则得到的权限可以传递。
SQL> grant connect, resorce to user50 with admin option;  //可以传递所获权限。

4、系统权限回收：系统权限只能由DBA用户回收

命令：SQL> Revoke connect, resource from user50;

说明：
1）如果使用WITH ADMIN OPTION为某个用户授予系统权限，那么对于被这个用户授予相同权限的所有用户来说，取消该用户的系统权限并不会级联取消这些用户的相同权限。
2）系统权限无级联，即A授予B权限，B授予C权限，如果A收回B的权限，C的权限不受影响；系统权限可以跨用户回收，即A可以直接收回C用户的权限。