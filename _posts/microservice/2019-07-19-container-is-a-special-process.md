---
layout: post
title: 容器就是一个特殊的进程
subtitle: 容器就是一个特殊的进程
author: ymkNK
date: 2019-07-19 11:56:44 +0800
categories: MicroService
tag: MicroService
img: 1.jpg
---
# 前言
程序的静态表现就是程序，平常的安安静静的躺在硬盘上；但是一旦运行起来，他就变成了计算机里的数据和状态的总和，这就是它的动态表现。
而容器的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出了一个边界。
- Cgroups是用来约束的主要手段。
- Namespace是用来修改视图的主要方法。

# docker run实验
安装好docker，输入以下命令
```
docker run -it busybox /bin/sh
```
这个命令就是告诉docker：帮我启动一个容器  
接下来我们可以使用ps指令，来看一看容器和宿主机中的进程信息。
宿主机里的内容：
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190719-120700@2x.png)
容器里的内容
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190719-120743@2x.png)

# Namespace机制
上面的小实验，这就是Linux里的namespace机制实现的，
我们可以看到，我们容器里pid为1的/bin/sh进程，在宿主机中的pid为13193，这就是容器使用了namespace使用了障眼法，让容器里的视角，以为自己就是这个机器的1号员工（pid=1），可是实际上，他还是原来的100号进程。

namespace其实只是Linux在创建新的进程的时候的一个参数，在Linux系统中创建线程的系统调用是clone()

```
int pid=clone(main_function,stack_size,SIGCHLD,NULL);
```

这个系统调用，创建了一个新进程，并且返回一个pid。而我们在创建一个新进程的时候，可以指定CLONE_NEWPID 参数。

```
int pid=clone(main_function,stack_size,CLONE_NEWPID | SIGCHLD,NULL);
```

这个新创建的进程就将“看见”一个全新的进程空间，看见它自己在这个进程空间中，他自己的pid为1。这其实只是一个障眼法，在宿主机上，它的pid该是多少，还是多少。

# Namespace种类
Namespace的种类很多，上文中提到的只是PID Namespace，除此之外，还有如下的Namespaces,这些namespaces用来实现对进程上下文的“障眼法操作”
- MOUNT Namespace， 用来隔离文件系统的挂载点，使得子进程拥有自己独立的挂载点信息。
- UTS（UNIX Time-sharing System ）Namespace提供了主机名和域名的隔离,能够使得子进程有独立的主机名和域名(hostname),而不仅仅只是宿主机上的一个进程.
- IPC Namespace,使划分到不同ipc namespace的进程组通信上隔离，无法通过消息队列、共享内存、信号量方式通信，但没有对所述所有IPC通信方式隔离。
- Network Namespace，是实现网络虚拟化的重要功能，它能创建多个隔离的网络空间，它们有独自的网络栈信息。不管是虚拟机还是容器，运行的时候仿佛自己就在独立的网络中。
- User Namespace,是Linux3.8新增的一种namespace，用于隔离安全相关的资源，包括user IDs and group IDs，keys,和capabilities。同样一个用户的userID和groupID在不同的user namespace中可以不一样(与PID namespace类似)。换句话说，一个用户可以在一个user namespace中是普通用户，但在另一个user namespace中是超级用户。

通过Namespace这样的方式，容器就只能看见当前Namespace所限定的资源、文件、设备、状态、配置。而对于宿主机和其他不想关的程序，对这个容器（一个进程）来说，就“雨我无瓜了”

# 结语
这个原理，就是为什么容器要比虚拟机效能更高的原因，本文是读了张磊前辈的文章后的学习笔记。