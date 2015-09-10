---
layout: post
title:  "树莓派无法启动-unable to mount root fs on unkown block "
date:   2015-09-04 22:11:00
category: Raspberry
tags: [Raspberry]
---
* content
{:toc}


##1. 现象与原因##

前端时间
树莓派`unable to mount root fs on unkown-block`  `fsck -t ext4 /dev/sdb1`

##2. 问题解决##

开始怀疑是树莓派板子损坏和SD卡损坏。

##3. 统备份与还原##

Windows平台下有个写入树莓派系统映像的[Win32DiskImager](http://sourceforge.net/projects/win32diskimager/ "Win32DiskImager")，其实细心的读者应该发现这个软件上面有个选项是Read。打开Win32DiskImager，如下图所示，Device选择SD卡所在盘符，Image File指定要备份的文件路径，需要注意的是备份的文件大小会相当于SD卡实际的容量，比如16GB的SD卡，备份下来文件也有14.7GB左右，请保证有足够的空间。准备妥当后直接点击Read，等完成后即可。


##4. 挂载SD卡文件系统并浏览查看LINUX系统文件##

因为在Windows下默认不能直接挂载Linux的分区，所以我们需要借助于另外一款软件：[DiskInternals Linux Reader](http://www.diskinternals.com/download/)，打开这个软件，连接上你的包含Raspberry Pi系统的SD卡，直接点击Drives -> Refresh Drive List就不仅可以查看Linux的分区，也可以对其中的文件进行操作。

##5. 浏览查看IMG镜像文件##

如何浏览并查看img镜像文件呢？还是用到刚才我们提到的DiskInternals Linux Reader工具，运行该软件，点击菜单 Drives -> Mount Image，选择Raw Disk Images，然后在弹出的文件选择框内选择树莓派的img文件，稍等片刻就可以看到镜像被成功加载了。

镜像成功挂载将会出现两个盘，其中一个是Linux根盘，这个是我们通常在Windows系统中看不到的，另外一个是FAT16的引导盘，这个绝大多数情况下可以看到。

1. [修复fs解决树莓派系统损坏无法启动](http://tieba.baidu.com/p/3552129837 "修复fs解决树莓派系统损坏无法启动")
2.  [树莓派Raspberry Pi备份SD卡系统、浏览挂载IMG分区镜像文件](http://wangye.org/blog/archives/938/ "树莓派Raspberry Pi备份SD卡系统、浏览挂载IMG分区镜像文件")