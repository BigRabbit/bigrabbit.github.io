---
layout: post
title:  "《CCNA学习指南》读书笔记-2. 思科IOS系统"
date:   2015-09-05 22:33:00
category: CISCO
tags: [CISCO, CCNA]
---

##一. 思科IOS系统介绍

IOS，即互联网络操作系统（Internetwork Operating System），是思科路由器和所有Catalyst交换机的内核，提供路由选择、交换、网络互联和远程通信功能。具体功能如下表：

- 运行网络协议及提供功能；
- 在设备间高速传输数据；
- 控制访问和禁止未经授权的网路使用，从而加强安全性；
- 提供可扩展性和冗余性；
- 提供连接到网络资源的可靠性。

###1. 路由器和交换机的主要组件
 
    组件 | 描述  
    -------|-------
    引导程序 | 存储在ROM中的微代码，用于引导初始化阶段启动路由器。
    POST（加电自检） | 存储在ROM中的微代码，检查路由器硬件的基本功能及判断路由器包含哪些接口
    ROM监视器 | 存储在ROM中的微代码，用于制造、测试和排除故障，并在无法从闪存加载IOS时运行微型IOS 
    微型IOS | RXBOOT或引导加载程序，启用接口及将思科IOS加载到闪存中
    RAM | 存储分组缓冲区、ARP缓存、路由选择表及路由器正常运行所需的软件和数据结构。运行配置存储在RAM中，大多数还在启动时将闪存中的IOS加载到RAM中
    ROM | 启动和维护路由器。存储POST、引导程序和微型IOS
    闪存 | 默认存储IOS
    NVRAM（非易失RAM） | 存储路由器和交换机的配置
    配置寄存器 | 控制路由器的启动方式

###2. 路由器和交换机的启动过程

1. IOS设备执行POST对硬件进行检查，确保设备不缺少必要的组件，且组件运行正常。然后测试各个接口，它存储在只读存储器（ROM）中。
2. ROM中的引导程序找到并加载思科IOS。默认情况下，所有思科设备都从闪存加载IOS软件。
3. 接下来，IOS软件在NVRAM中查找有效的配置文件，即启动配置。仅当管理员将运行配置复制到NVRAM时才存在。
4. 如果在NVRAM中找到启动配置文件，路由器或交换机将把它复制到RAM中，并将文件命名为运行配置。如果NVRAM中没有启动配置文件，路由器或交换机将通过所有能检测到载波的接口发送广播，以查找可提供配置的TFTP服务器。

##二.  连接到思科IOS设备

- 控制台端口：8针的RJ-45接头，位于设备背面。
- 辅助端口：与控制台端口一样，但可以配置调制解调器命令，允许调制解调器连接到路由器，进而可以远程带外（out-of-band）配置。
- Telnet/SSH程序：带内方式，通过网络配置设备。


##三. 6种模式 

###1. 用户执行模式（User EXEC Mode）

仅执行基本的监视命令，访问其它网络和主机，但不能看到和更改路由器的设置内容。

	Switch>
###2. 特权执行模式（Privileged EXEC Mode）

可执行所有的用户命令，还可以看到和更改路由器的设置内容。

	Switch>enable
	Switch#

从特权执行模式返回用户执行模式：
	
	Switch#disable
	Switch>
###3. 全局配置模式（Global Configuration Mode）

可以执行影响整个系统的命令

	Switch#configure terminal
	Enter configuration commands, one per line.  End with CNTL/Z.
	Switch(config)#	
###4. 端口模式

解耦库口配置是无疑是最重要的路由器配置之一，要与其他设备进行通信，接口必须绝对精确。接口只能在端口模式下进行配置。在**全局配置模式**下，针对路由器或交换机端口的进行配置，使用`interface type slot/port`或`interface type slot/subslot/port`来选择需要配置的接口。

首先通过以下命令列出路由器/交换机的所有接口：
	
	Router#show ip interface brief
	Interface                  IP-Address      OK? Method Status                Protocol
	Ethernet0/0                unassigned      YES unset  administratively down down
	Ethernet0/1                unassigned      YES unset  administratively down down
	Ethernet0/2                unassigned      YES unset  administratively down down
	Ethernet0/3                unassigned      YES unset  administratively down down
	Serial1/0                  unassigned      YES unset  administratively down down
	Serial1/1                  unassigned      YES unset  administratively down down
	Serial1/2                  unassigned      YES unset  administratively down down
	Serial1/3                  unassigned      YES unset  administratively down down
	
通过`show ip interface brief`命令，获知该路由器有4个以太网接口和4个串行WAN接口。然后，假设我们需要对`Serial1/0  `串行接口进行配置，便需要进入端口模式：

	Router#configure terminal
	Enter configuration commands, one per line.  End with CNTL/Z.
	Router(config)#interface Serial 1/0
	Router(config-if)#

所有交换机端口默认都是被启用，而所有路由器端口默认都被禁用。要禁用端口，可以使用接口配置命令`shutdown`；要启用接口，可以使用命令`no shutdown`：

	Router(config)#interface s1/0
	Router(config-if)#no shutdown
	*Mar  1 01:38:20.279: %LINK-3-UPDOWN: Interface Serial1/0, changed state to up
	*Mar  1 01:38:21.283: %LINEPROTO-5-UPDOWN: Line protocol on Interface Serial1/0, changed state to up
	Router(config-if)#do show interface s1/0
	Serial1/0 is up, line protocol is up
	  Hardware is M4T

配置串口接口时，如果串行接口连接到CSU/DSU设备，这种设备会为路由器线路提供时钟频率。而如果采用的是背对背配置（DCE与DTE），电缆的一端-数据通信设备（DCE）必须提供时钟频率。默认情况下，思科路由器的串行接口都是数据终端设备（DTE），意味着要让串行接口充当DCE，必须对其进行配置，使其提供时钟频率。

	

###5. 线路模式

	Router(config)#line ?
	  <0-134>  First Line number
	  aux      Auxiliary line
	  console  Primary terminal line
	  tty      Terminal controller
	  vty      Virtual terminal
	  x/y      Slot/Port for Modems
	
	Router(config)#line console 0
	Router(config-line)#


###6. 路由引擎模式

##四. 密码设置

###1. 启用密码（enable）

控制用户进入特权模式，在用户执行enable命令时所要求提供的密码。在10.3之前的老系统上设置启用密码，如果设置了启用加密密码，该密码不起作用。
	
	Switch(config)#enable ?
	last-resort  Define enable action if no TACACS servers respond
	password     Assign the privileged level password
	secret       Assign the privileged level secret
	use-tacacs   Use TACACS to check enable passwords
	Switch(config)#enable password rabbit
	Switch(config)#
###2. 启用加密密码(enable secret)

和启用密码一样，控制用户进入特权模式，在用户执行enable命令时所要求提供的密码。较新的加密密码，口令以一种更加强健的加密形式被存储下来。如果设置了，优先启用。

	Switch(config)#enable secret pirate
	Switch(config)#
###3. 控制台密码

	Switch(config)#line console 0
	Switch(config-line)#password rabbit
	Switch(config-line)#login
###4. 辅助端口密码

记住：交换机没有辅助端口。	

	Switch(config)#line aux ?
	<0-0>  First Line number
	
	Switch(config)#line aux 0
	Switch(config-line)#password rabbit
	Switch(config-line)#login
###5. Telnet(VTY)密码

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

##五. 管理思科配置

**查看当前配置**

查看DRAM中的当前配置：

	Switch#show running-config

**查看启动配置**

查看NVRAM中存储的启动配置：

	Switch#show startup-config

**当前配置保存为启动配置**


	Switch#copy running-config startup-config

**当前配置保存至TFTP服务器**

	Switch#copy running-config tftp

**恢复思科设备的配置**

如果恢复到启动配置文件中的版本：

	Switch#copy startup-config running-config

如果恢复TFTP服务器上保存的配置版本：

	Switch#copy tftp running-config 

**删除配置**

	Switch#erase startup-config
	Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]
	[OK]
	Erase of nvram: complete
	Switch#
	*Mar  1 02:45:18.411: %SYS-7-NV_BLOCK_INIT: Initialized the geometry of nvram	
	SW1#reload
	Proceed with reload? [confirm]

删除交换机或路由器的NVRAM中的内容。如果此时在特权模式下执行命令`reload`并选择不保存所做的修改，交换机或路由器将重启并进入**设置模式**。