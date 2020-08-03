---
layout: post
title: Idea代码飘红的问题解决方式
subtitle: Idea代码飘红问题解决方式
author: ymkNK
date: 2019-08-07 10:08:30 +0800
categories: Tech
tag: Tech
img: 9.jpg
---
## 前言
在安装使用Idea Intelli的时候，导入新项目的时候，总会出现代码权限飘红的问题。本篇文章，就是介绍出现这种问题的几种情况和分别的解决方式。
![导入项目代码全线飘红](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190807-102252@2x.png)


## 情况一：maven二方库settings没有设置
一般公司内部都会有自己的二方库建设，新人在git下项目之后，总是会出现代码偏红的问题，各种库都没有导入下来，这时候首先应该排查的就是maven的配置是否正确，是否正确连接到了公司内部的二方库。
### 解决方式
以Idea Intelli为例:
**preference -> build,execution,deployment -> build tools -> maven -> User settings file: 勾选上override，改写为二方库settings.xml文件的路径**

![设置maven settings](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190807-105505.png)

## 情况二：Lombok插件没有安装
> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.[lombok官网](https://www.projectlombok.org/)

Lombok可以让我们的Java代码更加简洁，不用再去写那些Getter和Setter。然而这个强大的功能，在初次安装idea的时候，并不会内置的给我们安装Lombok插件，因此这个时候，同样也会出现大片代码报错的问题。

### 解决方式
安装Lombok的插件：
** preference -> Plugins -> Install lombok -> restart **
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190807-111529.png)

## 问题三：注解未开启
在安装好了 Lombok之后，发现运行还是报错
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190807-112144@2x.png)

### 解决方式
1. 关闭项目。 
2. Preference > Build, Execution, Deployment > Compiler > Annotation Processors. Check 'Enable annotation processing'. 
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190807-112533.png)
3. Open your project. File > Invalidate Caches / Restart... > Invalidate and Restart 然后等待这个过程完成结束,将会发现一切都将回归平淡，云淡风轻.