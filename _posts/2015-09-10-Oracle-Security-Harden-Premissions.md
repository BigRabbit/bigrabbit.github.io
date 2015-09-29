---
layout: post
title:  "Oracle 安全加固-2. Oracle权限管理 "
date:  2015-09-13 22:11:00
category: Oracle
tags: [Oracle, "Security Harden", Permission]
---

权限（privilege）是Oracle用户数据库预定义好的执行某些操作的能力。角色（role）是权限管理的一种解决方案，是一组相关权限的集合。用户（user）是能够访问（即连接、登录和操作）数据库的人员。


##一. 权限分类

按照权限所针对的控制对象，可以分为：

- 系统权限（system privilege）：在系统级控制用户使用数据库的存取和使用的机制，即执行某种SQL语句的能力。如是否能够启动、停止数据库，是否能修改数据库参数，是否能连接到数据库等。
- 对象权限（object privilege）：在对象级控制数据库的存取和使用的机制，即访问其他用户的Scheme对象的能力。如用户可以存取哪个用户Scheme中的哪个对象，是否能够对该对象进行查询、插入或更新等（是针对表或视图而言的）。

其中用户权限信息保存在数据字典中。

权限可以通过如下两种方式授予用户：

- 直接授予，即直接将权限授予用户。
- 间接授予，及先将权限授予角色，然后将角色授予用户。

而且权限的授予是可以传递的，即已具有权限的用户可以将自己的权限或其中一部分权限再授予或传递给其他用户。


###1. 系统权限

一般情况下，系统权限只授予值得信任的用户，避免系统权限被滥用，而直接危及数据库的安全性。数据字典视图system_privilige_map中包含了Oracle数据库中的所有系统权限。可以通过`select count * from system_privilige_map`查询所有的系统权限或`select count * from system_privilige_map where name '%ROLE%'`查询与角色有关的系统权限。

**系统权限分类**：

根据类型不同，Oracle内置的系统权限大致分为以下几类：

- 数据库权限
- 概要文件权限
- 角色权限
- 同义词权限
- 表权限
- 表空间权限
- 专用（或管理权限）

	- SYSDBA：系统管理员权限
	- SYSOPER：系统操作员权限
- 其他权限
- ....

根据所操作的对象不同，系统权限可以划分为：

- 针对Scheme对象的系统权限。
	
	- 具有ANY关键字的系统权限：表示可以在任何用户的Scheme中进行操作，如`CREATE ANY TABLE`意味着可以在任何用户的Scheme中创建表格。
	- 不具有ANY关键字的系统权限：表示只在自己的方案中Scheme中进行操作，如`CREATE  TABLE`意味着可以在自己的Scheme中创建表格。
- 针对非Scheme对象的系统权限，只与数据库管理中全局的、唯一的数据库维护操作有关，如`CREATE TABLESPACE`。 

###2. 对象权限

Oracle数据库的Scheme对象主要是指：表、索引、视图、同义词、过程、函数、包、触发器。

创建对象的用户拥有该对象的所有对象权限，而不需要特别的授予。所以，对象权限的设置实际上是为对象所有者给其他用户提供操作该对象的某种权利的一种方法。

其中，Oracle数据库一共有9种不同的对象权限，如下表所示：

    对象 | alter | delete | execute | index | insert | read | reference | select | update  
    -------|-------|-------|-------|-------|-------|-------|-------|-------|-------
    directory | | | | |  | √ | | | | 
    function | | | √  | |  | | | | | 
    procedure | | | √  | |  | | | | | 
    package | | | √  | |  | | | | | 
    db object | | | √  | |  | | | | | 
    library | | | √ | | | | | | | 
    operator | | | √ | | | | | | | 
    sequence | √| | | | | | | | 
    table | √ | √ | | √ | √ | | √ | √ | √ 
    type | | | | |  | √ | | | | 
    view | | √ | | | √ | | | | √ | √
  
##二. 授予和回收权限

可以将系统权限、对象权限授予指定的用户、角色。执行授权的用户必须本身具有这种权限，或在获得权限时得到了`WITH ADMIN OPTION`选项（对于系统权限），或得到了`WITH GRANT OPTION`选项（对于对象选项），或该用户本身具有`GRANT ANY PRIVILEGE`系统权限。

当授予了相关的权限后，这些权限（除专用权限sysdba和sysoper之外）立即生效，而不需要关闭、重启数据库，也不需要获得特权的用户先，然后再重新登录。

###1. 授予系统权限
	
数据库的系统权限首先是被授予sys用户、system用户。在Oracle数据库安装过程中，安装程序会调用dsec.bsq脚本，该脚本会创建三个角色（connect、resource和dba）及其分配相应权限。

使用GRANT局域可以将系统权限分配授予指定的用户、角色、PUBLIC公共用户组，语法是：

	GRANT { system_privilege | ALL [PRIVILEGES] }
		[,  { system_privilege| ALL [PRIVILEGES] } ] ......
	To {user|role|PUBLIC} [, {user|role|PUBLIC} ] ......
	[WITH ADMIN OPTION];
其中：

- system_privilege 表示系统权限。
- user 表示被授权用户。
- role 表示被授权的角色。
- PUBLIC 表示公共用户组。

`PUBLIC`公共用户组是一个在创建数据库时就被自动创建的用户组。该用户组有什么权限，数据库中所有用户就有什么权限。

如果授权时带有`WITH ADMIN OPTION`选项，则被授权的用户可以将这些系统权限传递给其他用户。

###2. 查询系统权限

与系统权限有关的数据字典与视图如下表所示：

    视图 | 说明
     -------|-------
    DBA\_SYS\_PRIVS | 授予所有用户和角色的系统权限。
    USER\_SYS\_PRIVS | 授予当前用户的系统权限。
    ORLE\_SYS\_PRIVS | 包含授予角色的系统权限的信息。它提供的只是该用户可以访问的角色的信息。
    SESSION\_PRIVS | 当前会话可以使用的系统权限（包括直接授予的和通过角色授予的系统权限）。
    V$PWFILE\_USERS | 所有被授予sysdba或sysoper专用系统权限的用户信息。
    SYSTEM\_PRIVILEGE\_MAP | 所有系统权限，包括sysdba或sysoper专用系统权限。

###3. 回收系统权限

	REVOKE { system_privilege | ALL [PRIVILEGES] }
		[,  { system_privilege| ALL [PRIVILEGES] } ] ......
	FROM {user|role|PUBLIC} [, {user|role|PUBLIC} ] ...... ;
	
回收系统权限不会级联。如果用户A将系统权限a授予用户B（并带有WITH ADMIN OPTION`选项），用户B又将系统权限a授予用户C。那么，当删除B或从用户B回收系统权限a后，用户C仍然保留着系统权限a。

###4. 授予对象权限

授予对象权限的工作一般情况下是由对象的所有者完成。DBA用户（sys用户、system用户）可以访问任何用户的任何对象，并且可以将这些对象上的对象权限授予其他用户。

	GRANT { object\_privilege | ALL [PRIVILEGES] [ (column\_1 [, column\_2]......)  ] }
		[,  { object\_privilege | ALL [PRIVILEGES] [ (column\_1 [, column\_2]......)  ] } ] ......
	ON [scheme.] object
	To {user|role|PUBLIC} [, {user|role|PUBLIC} ] ......
	[WITH GRANT OPTION]
	[WITH HIERARCHY OPTION];

其中：

- object_privilege 表示对象权限
- column_1, column_2, column_m, column_n 表示列权限对应的列的列表。
- scheme表示方案名。
- object 表示对象。
- user 表示被授权的用户。
- role表示被授权的角色。
- PUBLIC表示公共用户组。

如果授权时带有`WITH GRANT OPTION`选项，则被授权的用户可以将这些对象权限传递给其他用户。

授予对象权限时需要注意如下几个方面：

- 如果是自己创建的对象，则具有该对象所有对象权限，而无须被授予。
- 如果要授予对象权限，则该对象必须是自己Scheme中的对象，或得到该对象权限时也得到了`WITH GRANT OPTION`选项，或拥有`grant any privilege`系统权限。
- `WITH GRANT OPTION`选项不能被授予角色。

###5. 查询对象权限

与对象权限有关的数据字典与视图如下表所示：

    视图 | 说明
     -------|-------
    DBA\_TAB\_PRIVS | 数据库所有用户或角色的对象权限
    ALL\_TAB\_PRIVS | 当前用户或PUBLIC的对象权限
    USER\_TAB\_PRIVS | 当前用户的对象权限
    DAB\_COL\_PRIVS | 数据库中所有用户或角色的列对象权限
    ALL\_COL\_PRIVS | 当前用户或PUBLIC是其所有者、授予者或被授予者的所有的列对象权限
    USER\_COL\_PRIVS | 当前用户是其所有者、授予者或被授予者的所有的列对象权限
    ALL\_COL\_PRIVS\_MADE | 当前用户是其所有者、授予者的所有的列对象权限
    USER\_COL\_PRIVS\_MADE | 当前用户是其授予者的所有的列对象权限
    ALL\_COL\_PRIVS\_RECD | 当前用户或PUBLIC是被授予者的所有的列对象权限
    USER\_COL\_PRIVS\_RECD | 当前用户是被授予者的所有的列对象权限
    ALL\_TAB\_PRIVS\_MADE | 当前用户所做的所有对象授权或者是在当前用户所拥有的对象上的授权
    USER\_TAB\_PRIVS\_MADE | 当前用户所做的所有的列对象权限
    ALL\_TAB\_PRIVS\_RECD | 当前用户或PUBLIC用户是被授予者的对象权限
    USER\_TAB\_PRIVS\_RECD | 当前用户是被授予者的的对象权限
    ROLE\_TAB\_PRIVS | 包含了授予角色的对象权限。提供的只是该用户可以访问的角色的信息。

###6. 回收对象权限


	REVOKE { object_privilege | ALL [PRIVILEGES] [ (column_1 [, column_2]......)  ] }
		[,  { object_privilege | ALL [PRIVILEGES] [ (column_1 [, column_2]......)  ] } ] ......
	ON [scheme.] object
	FROM {user|role|PUBLIC} [, {user|role|PUBLIC} ] ...... 
	[CASCADE CONSTRAINTS];

回收对象权限时要注意如下几个方面：

- 授权者只能才能够自己授权的用户那里回收对象权限。
- 用户不能有选择性地回收权限，必须首先回收该对象权限，然后再将合适的列权限授予用户。
- 假如基于一个对象权限创建了过程、视图，那么回收该对象后，这些过程、视图将变为无效。
- 回收对象权限会级联。如果用户A将对象权限a授予用户B（并带有WITH GRANT OPTION`选项），用户B又将对象权限a授予用户C。那么，当删除B或从用户B回收系统权限a后，用户C将不再具有对象权限a，并且用户B和用户C中与该对象权限有关的对象都变成无效。
- 如果用户A将对象权限a授予用户C，用户B也将对象权限a授予了用户C。那么删除用户A或删除用户B后，用户C仍然保留着对象权限a。

`CASCADE CONSTRAINTS` 选项表示在回收对象权限时，同时删除利用REFERENCES对象权限在对象上定义的参照完整性约束。

##三. 小结

通过向用户授予或回收权限，便可以控制用户在数据库中能做什么和不能做什么。所以，通过用户名和口令控制是否能够登录数据库只是第一道防线，Oracle数据库的安全机制主要还是通过权限第二道防线来控制的。
