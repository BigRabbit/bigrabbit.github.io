---
layout: post
title:  "《CCNA学习指南》读书笔记：2. 思科IOS系统初探"
date:   2015-09-05 22:33:00
categories: CCNA
excerpt: 《CCNA学习指南》读书笔记
---
* content
{:toc}

##思科IOS系统介绍##

IOS，即互联网络操作系统（Internetwork Operating System），是思科路由器和所有Catalyst交换机的内核，提供路由选择、交换、网络互联和远程通信功能。具体功能如下表：

- 运行网络协议及提供功能；
- 在设备间高速传输数据；
- 控制访问和禁止未经授权的网路使用，从而加强安全性；
- 提供可扩展性和冗余性；
- 提供连接到网络资源的可靠性。

##路由器和交换机的主要组件##
 
	| 组件 | 描述 | 
	|--------|-------|
	| 引导程序 | 存储在ROM中的微代码，用于引导初始化阶段启动路由器。|
	| POST（加电自检） | 存储在ROM中的微代码，检查路由器硬件的基本功能及判断路由器包含哪些接口|
	| ROM监视器 | 存储在ROM中的微代码，用于制造、测试和排除故障，并在无法从闪存加载IOS时运行微型IOS |
	| 微型IOS | RXBOOT或引导加载程序，启用接口及将思科IOS加载到闪存中 |
	| RAM | 存储分组缓冲区、ARP缓存、路由选择表及路由器正常运行所需的软件和数据结构。运行配置存储在RAM中，大多数还在启动时将闪存中的IOS加载到RAM中 |
	| ROM | 启动和维护路由器。存储POST、引导程序和微型IOS|
	| 闪存 | 默认存储IOS |
	| NVRAM（非易失RAM） | 存储路由器和交换机的配置 |
	| 配置寄存器 | 控制路由器的启动方式 |

#路由器和交换机的启动过程##

1. IOS设备执行POST对硬件进行检查，确保设备不缺少必要的组件，且组件运行正常。然后测试各个接口，


##连接到思科IOS设备##

- 控制台端口：8针的RJ-45接头，位于设备背面。
- 辅助端口：与控制台端口一样，但可以配置调制解调器命令，允许调制解调器连接到路由器，进而可以远程带外（out-of-band）配置。
- Telnet/SSH程序：带内方式，通过网络配置设备。


##2.6种模式 ##

- .用户执行模式（User EXEC Mode）：仅执行基本的监视命令，访问其它网络和主机，但不能看到和更改路由器的设置内容。

		Switch>
- 特权执行模式（Privileged EXEC Mode）：可执行所有的用户命令，还可以看到和更改路由器的设置内容。

		Switch>enable
		Switch#
- 全局配置模式（Global Configuration Mode）：可以执行影响整个系统的命令

		Switch#configure terminal
		Enter configuration commands, one per line.  End with CNTL/Z.
		Switch(config)#	
- 端口模式：路由器端口的配置
- 线路模式
- 路由引擎模式

##3. 密码设置##

- 启用密码（enable）

	控制用户进入特权模式，在用户执行enable命令时所要求提供的密码。在10.3之前的老系统上设置启用密码，如果设置了启用加密密码，该密码不起作用。
		
		Switch(config)#enable ?
		last-resort  Define enable action if no TACACS servers respond
		password     Assign the privileged level password
		secret       Assign the privileged level secret
		use-tacacs   Use TACACS to check enable passwords
		Switch(config)#enable password rabbit
		Switch(config)#
- 启用加密密码(enable secret)

	和启用密码一样，控制用户进入特权模式，在用户执行enable命令时所要求提供的密码。较新的加密密码，口令以一种更加强健的加密形式被存储下来。如果设置了，优先启用。

		Switch(config)#enable secret pirate
		Switch(config)#
- 控制台密码

		Switch(config)#line console 0
		Switch(config-line)#password rabbit
		Switch(config-line)#login
- 辅助端口密码
	
	记住：交换机没有辅助端口。	

		Switch(config)#line aux ?
		<0-0>  First Line number
		
		Switch(config)#line aux 0
		Switch(config-line)#password rabbit
		Switch(config-line)#login
- Telnet(VTY)密码

	通过Telnet登录到路由器所需要的密码。

		Switch(config)#line vty 0 ?
			<1-871>  Last Line number
			<cr>

		Switch(config)#line vty 0 871
		Switch(config-line)#password rabbit
		Switch(config-line)#login

	若未设置该密码，路由器默认拒绝所有Telnet连接请求。可以通过`no login`命令绕过，从而允许在没有设置Telnet密码时也允许建立Telnet连接。

		Switch(config)#line vty 0 871
		Switch(config-line)#no login

	默认情况下，只有启用加密密码是加密的，也就是说执行命令`show running-config`时，可以查看到除启用加密密码外的其他所有密码。若要对其他密码加密，必须进行手工配置。

		Switch#configure terminal
		Enter configuration commands, one per line.  End with CNTL/Z.
		Switch(config)#service password-encryption
		Switch(config)#exit
		Switch#show running-config