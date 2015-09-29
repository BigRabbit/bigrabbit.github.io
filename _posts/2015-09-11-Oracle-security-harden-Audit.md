---
layout: post
title: "Oracle Security Harden Audit"
category: 
tags: []
---

##1、什么是审计

　　审计（Audit)用于监视用户所执行的数据库操作，并且Oracle会将审计跟踪结果存放到OS文件（默认位置为$ ORACLE_BASE/admin/$ORACLE_SID/adump/）或数据库（存储在system表空间中的SYS.AUD$表中，可通过视图 dba_audit_trail查看）中。默认情况下审计是没有开启的。
　　不管你是否打开数据库的审计功能，以下这些操作系统会强制记录：用管理员权限连接Instance；启动数据库；关闭数据库。

##2、和审计相关的两个主要参数

###Audit_sys_operations

默认为false，当设置为true时，所有sys用户（包括以sysdba,sysoper身份登录的用户）的操作都会被记录，audit trail不会写在aud$表中，这个很好理解，如果数据库还未启动aud$不可用，那么像conn /as sysdba这样的连接信息，只能记录在其它地方。如果是windows平台，audti trail会记录在windows的事件管理中，如果是linux/unix平台则会记录在audit_file_dest参数指定的文件中。

###Audit_trail：

- None：是默认值，不做审计；
- DB：将audit trail 记录在数据库的审计相关表中，如aud$，审计的结果只有连接信息；
- DB,Extended：这样审计结果里面除了连接信息还包含了当时执行的具体语句；
- OS：将audit trail 记录在操作系统文件中，文件名由audit_file_dest参数指定；
- XML：10g里新增的。

*注：这两个参数是static参数，需要重新启动数据库才能生效。*

##3、审计级别

　　当开启审计功能后，可在三个级别对数据库进行审计：Statement(语句)、Privilege（权限）、object（对象）。
　　Statement：
　　按语句来审计，比如audit table 会审计数据库中所有的create table,drop table,truncate table语句，alter session by cmy会审计cmy用户所有的数据库连接。
　　Privilege：
　　按权限来审计，当用户使用了该权限则被审计，如执行grant select any table to a，当执行了audit select any table语句后，当用户a 访问了用户b的表时（如select * from b.t）会用到select any table权限，故会被审计。注意用户是自己表的所有者，所以用户访问自己的表不会被审计。
　　Object：
　　按对象审计，只审计on关键字指定对象的相关操作，如aduit alter,delete,drop,insert on cmy.t by scott; 这里会对cmy用户的t表进行审计，但同时使用了by子句，所以只会对scott用户发起的操作进行审计。注意Oracle没有提供对schema中所有对象的审计功能，只能一个一个对象审计，对于后面创建的对象，Oracle则提供on default子句来实现自动审计，比如执行audit drop on default by access;后，对于随后创建的对象的drop操作都会审计。但这个default会对之后创建的所有数据库对象有效，似乎没办法指定只对某个用户创建的对象有效，想比 trigger可以对schema的DDL进行“审计”，这个功能稍显不足。
　　4、审计的一些其他选项
　　by access / by session：
　　by access 每一个被审计的操作都会生成一条audit trail。
　　by session 一个会话里面同类型的操作只会生成一条audit trail，默认为by session。
　　whenever [not] successful：
　　whenever successful 操作成功(dba_audit_trail中returncode字段为0) 才审计,
　　whenever not successful 反之。省略该子句的话，不管操作成功与否都会审计。
　　5、和审计相关的视图
　　dba_audit_trail：保存所有的audit trail，实际上它只是一个基于aud$的视图。其它的视图dba_audit_session,dba_audit_object, dba_audit_statement都只是dba_audit_trail的一个子集。
　　dba_stmt_audit_opts：可以用来查看statement审计级别的audit options，即数据库设置过哪些statement级别的审计。dba_obj_audit_opts,dba_priv_audit_opts视图功能与之类似
　　all_def_audit_opts：用来查看数据库用on default子句设置了哪些默认对象审计。
　　6、取消审计
　　将对应审计语句的audit改为noaudit即可，如audit session whenever successful对应的取消审计语句为noaudit session whenever successful;
　　7、10g中的审计告知一切
　　Oracle 数据库 10g 审计以一种非常详细的级别捕获用户行为，它可以消除手动的、基于触发器的审计。
　　假定用户 Joe 具有更新那张表的权限，并按如下所示的方式更新了表中的一行数据：
　　update SCOTT.EMP set salary = 12000 where empno = 123456;
　　您如何在数据库中跟踪这种行为呢？在 Oracle 9i 数据库及其较低版本中，审计只能捕获“谁”执行此操作，而不能捕获执行了“什么”内容。例如，它让您知道 Joe 更新了 SCOTT 所有的表EMP，但它不会显示他更新了该表中员工号为 123456 的薪水列。它不会显示更改前的薪水列的值 — 要捕获如此详细的更改，您将不得不编写您自己的触发器来捕获更改前的值，或使用 LogMiner 将它们从存档日志中检索出来。
　　细粒度审计(FGA) ，是在 Oracle 9i 中引入的，能够记录 SCN 号和行级的更改以重建旧的数据，但是它们只能用于 select 语句，而不能用于 DML ，如 update 、insert 和delete 语句。因此，对于 Oracle 数据库 10g 之前的版本，使用触发器虽然对于以行级跟踪用户初始的更改是没有吸引力的选择，但它也是唯一可靠的方法。