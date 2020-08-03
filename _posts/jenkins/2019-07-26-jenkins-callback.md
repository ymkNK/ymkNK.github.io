---
layout: post
title: 使用Jenkins回调获取状态
subtitle: 使用Jenkins回调获取状态
author: ymkNK
date: 2019-07-26 11:01:52 +0800
categories: Tech
tag: Jenkins
img: 3.jpg
---
## 前言
最近的工作需要使用Jenkins执行一个特定脚本，但是使用FeignClient调用Jenkins的执行接口后，并不能获取到脚本执行之后的结果（成功和失败），本文就将介绍这种问题的其中一种解决方式。
>Jenkins是开源CI&CD软件领导者，提供超过1000个插件来支持构建、部署、自动化，满足任何项目的需要。[Jenkins官网](https://jenkins.io/zh/)

## 关键点
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190726-115622@2x.png)
通过调研后，Jenkins在完成构建后，可以添加一个构建后操作。这就可以通过这个构建后操作，将本次构建脚本执行的结果，通过这个构建后操作，传给另外一个job，然后通过这个job将执行的结果通过回调的方式，传给应用服务器。

## 整个流程
1. 服务器端创建一条记录，获取到记录的id，与需要执行的job所需要的参数还有回调所需要用到的apiHost一同存入到map当中，转换为JSON格式，通过FeignClient调用Jenkins对应的脚本。
>服务器端应用可能会部署在不同的环境下，连接不同的数据库，因此可以在application.yml中配置当前apiHost的值，通过ConfigServer可以更改为当前环境所对应的值。

2. 在Jenkins端进行对应的job执行。
3. 通过构建后操作的trigger parameterized build on other projects功能。
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190726-150927@2x.png)
4. 根据本次构建的成功与否，分别执行不同的构建后操作，传递给回调job对应的参数。
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190726-145102@2x.png)
![](qeh76ukrx.bkt.clouddn.com/assets/img/pics/WX20190726-145131@2x.png)

5. 回调job将获取到的信息，通过一个shell脚本，通过curl传递给服务器端。
```
echo "${ID}"
echo "${API_HOST}"
echo "${BUILD_NO}"
echo "${RESULT}"
curl "$API_HOST/api/xxx/xxx/xxx/hook" -X POST -d "Id=$ID&buildNo=$BUILD_NO&result=$RESULT" 
```
6. 服务器端拿到数据进行更新和处理。

## 结语
有时候只是想要一条小小的消息，却不知道要辗转多少次，才能到达该去的地方