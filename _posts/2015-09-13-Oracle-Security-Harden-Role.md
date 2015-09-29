---
layout: post
title: "Oracle 安全加固-2. Oracle角色管理"
category: Oracle
tags: [Oracle, "Security Harden", Role]
---

##一. 角色概述

角色（role）是权限管理的一种解决方案，是一组相关权限的命名的集合。角色可以简化对权限的管理，先根据实际业务规则来划分权限，创建不同的角色，然后分别授予不同的系统权限、对象权限，最后将不同的角色分配给需要不同权限的用户。

角色特点如下：

- 角色名具有唯一性。
- 角色不属于任何用户或Scheme。
- 角色的描述信息存储在数据字典中。
- 除了角色本身之外，可以将角色授予任何数量的用户或角色。
- 可以给角色授予任何数量的系统权限、对象权限。
- 如果创建角色时指定了需要验证，则在启用该角色时需要提供验证信息，如口令。

使用角色的优势如下：

- 减少权限管理的工作量。
- 实现权限的动态管理。
- 有选择地使用权限。
- 可以使用操作系统来管理权限。此时，授予用户的数据库角色不再起任何作用。

数据库字典视图`DBA_ROLES`中包含了数据库中所有（预定义、自定义）角色。

	SELECT * FROM DBA_ROLES; #查询所有角色；
	SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE='CONNECT' ORDER BY PRIVILEGE; #查询CONNECT角色的系统权限；

##二. 预定义角色

Oracle安装过程中，安装程序会调用desc.bsq和catexp.bsq脚本创建以下几个默认角色：

- CONNECT：使用户连接、登录到数据库，即创建一个会话（session）。应将这个角色授予任何需要访问数据库的用户或应用程序。
- RESOURCE：使用户在其相关的Scheme中，创建、更改和删除某些类型的Scheme对象。一般应将该角色授予开发人员和那些必须创建Scheme对象的其他用户。
- DBA：拥有所有的系统权限，以及给其他用户授予所有系统权限的能力，但不包括启动和关闭数据库的权限。（默认授予了sys和system用户）。
- EXECUTE\_CATALOG\_ROLE：具有EXECUTE对象权限。
- SELECT\_CATALOG\_ROLE：具有在所有数据字典上的SELECT对象权限。
- DELETE\_CATALOG\_ROLE：具有在所有数据字典上的DELETE对象权限。
- EXP\_FULL\_DATABASE：具有执行数据库导出（export）权限。
- IMP\_FULL\_DATABASE：具有执行数据库导入（import）权限。

##三. 角色管理

###1. 创建角色

创建角色的语法：

	CREATE ROLE role_name { [ IDENTIFIED BY password ] | [ IDENTIFIED EXTERNALLY ]};

- `IDENTIFIED BY password`要求在启用该角色之前先验证口令。
- `IDENTIFIED EXTERNALLY`要求在启用该角色之前先通过一个外部服务进行身份验证。

###2. 角色的权限管理

角色的权限管理与用户权限管理的内容、步骤、方法基本相同，但需要注意以下几点。

- 角色授予对象权限时，不允许带有`WITH GRANT OPTION`和`WITH HIERARCHY OPTION`选项（只能为select对象权限指定该选项）。
- 不能使用`ALTER ROLE`语句更改角色的名称。必须先删除角色，再添加新名称的角色，然后授予相应的权限。

###3. 用户的角色管理

用户的角色可以分为默认角色、启用角色、禁用角色。授予、回收用户的角色可以理解生效，而不需要重新登录。

**授予用户角色**

	GRANT role1 [, role2] ...
	To {user|role|PUBLIC} [, {user|role|PUBLIC} ] ......
	[WITH ADMIN OPTION];

- `WITH ADMIN OPTION`选项，则获得该角色的用户可以将该角色授予其他用户或角色，或从其他用户那里回收该角色，甚至可以修改或删除该角色。

创建角色的用户隐含得到了该角色的`WITH ADMIN OPTION`。

**回收用户的角色**

	ROVKE role1 [, role2] ...
	FROM {user|role|PUBLIC} [, {user|role|PUBLIC} ] ......;

回收角色不会级联，与回收系统权限相同，但与回收对象权限不同。

**更改用户的默认角色**

当为用户授予某个角色后，初始时，该角色就成为了该用户的默认角色，即当该用户登录数据库后由Oracle自动启用的角色。

可以使用`ALTER USER`语句更改用户所使用的默认角色。

	ALTER  USER user
	DEFAULT ROLE {role1, [, role2]... | ALL [EXCEPT rolex [, roley ...] | NONE};

- ALL：表示除了在`EXCEPT`子句中指定的rolex和roley等之外的角色都是默认角色。
- EXCEPT：表示子句后面的rolex和roley等表示的角色不是默认角色。
- NONE：表示授予用户的所有角色都不是默认角色。

###4. 启用与禁用角色

在当前会话中，可以启用或禁用该会话（或该用户）的角色，即临时地使某些角色起作用或不起作用，而在下一个会话（或重新连接）中，仍恢复为用户的默认角色。

可以使用`SET ROLE`与`DBMS_SESSION.SET_ROLE`过程来启用或禁用角色。但不能在PL/SQL过程中使用`SET ROLE`命令。

	SET ROLE {role1 [ IDENTIFIED BY password1 ]
	[, role2 [ IDENTIFIED BY password1 ] ] ...
	| ALL [ EXCEPT rolex [, roley]......] | NONE ];

- ALL：表示除了在`EXCEPT`子句中指定的rolex和roley等之外的角色。
- EXCEPT：表示子句后面的rolex和roley等表示禁用的角色。
- NONE：表示禁用所有授予用户的角色。

###5. 删除角色

	DRO ROLE role;

##6. 角色查询

与角色有关的数据字典表与视图如下表所示：

	视图 | 说明
	-------|-------
	DBA\_ROLES | 数据库中存在的所有角色。
	DBA\_ROLE\_PRIVS | 授予给用户或角色的角色。
	USER\_ROLE\_PRIVS | 授予给当前用户的角色。
	ROLE\_ROLE\_PRIVS | 授予给其他角色的角色。
	ROLE\_SYS\_PRIVS | 授予给角色的系统权限的信息。
	ROLE\_TAB\_PRIVS | 授予给角色的对象权限的信息。
	DBA\_SYS\_PRIVS | 授予给用户和角色的所有系统权限。
	DBA\_TAB\_PRIVS | 授予给用户和角色的所有对象权限。
	SESSION_ROLES | 当前用户（当前会话）启用的角色。

查询某个角色所具有的系统权限

	SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE='MYROLE1' ORDER BY PRIVILEGE;

查询某个角色所具有的对象权限

	SELECT * FROM DBA_TAB_PRIVS WHERE GRANTEE='MYROLE1' ORDER BY PRIVILEGE;

##四. 小结

一般情况下，创建角色、给角色授权是由DBA来完成的，如果其他用户完成这些事，必须具有`CREATE ROLE` 和`GRANT ANY ROLE`系统权限。