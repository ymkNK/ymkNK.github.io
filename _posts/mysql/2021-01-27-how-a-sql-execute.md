---
layout: post
title: 基础架构：一条SQL更新语句是如何执行的？
subtitle: how-a-sql-executes
author: ymkNK
date: 2021-01-27 14:46:13 +0800
categories: mysql
tag: mysql
img: 6.jpg
---

>[本文是学习笔记，来自极客时间](https://time.geekbang.org/column/article/68319)


`mysql> select * from T where ID=10；` 这么一条查询语句是如何执行的呢？

MySQL基础架构
![](https://lllovol.oss-cn-beijing.aliyuncs.com/oss/mysql%E5%9F%BA%E6%9C%AC%E6%9E%B6%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.png?OSSAccessKeyId=LTAI4G4Hqhts9opvvGjEcW7y&Expires=1611735777&Signature=bqglE2mtVF8TVGGjL8v6kALjymU%3D)