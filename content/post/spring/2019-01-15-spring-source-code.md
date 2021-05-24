---
author: ymkNK
categories: Spring
date: "2019-01-15T17:06:53Z"
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/8.jpg
subtitle: 《Spring源码深度解析》读后感(一)
tag: Spring
title: 《Spring源码深度解析》读后感(一)Spring的容器Bean
---
# 前言
这个系列主要是用来记录自己读这本书的一些笔记和想法的记录，不过买书好像买错了，应该买spring boot的相关解读的书籍，这是spring的深入解读的书。吃了一亏，不过这也是spring boot的前身，其中的源码也是有价值的。  
首先开始的，是整个spring当中最核心的用法，但是却十分重要的一个部分--容器。大道至简，大音希声。
# 容器的基本用法
bean是spring中最核心的东西，因为spring就像一个大水桶，而bean就像是水桶中的水，水桶脱离了水便没有了什么用处。而spring 的目的，就是让我们的bean，能够成为纯粹的POJO（plain ordinary java object）。
需要：  
- 一个成员属性
- 这个属性的getter
- 这个属性的setter
然后在配置文件xml当中添加相关的bean

        <bean id="myTestBean" class="bean.MyTestBean"/>

然后就可以在测试代码中使用我们所测试的bean了
# 容器的基础XmlBeanFactory
代码如下

        BeanFactory bf= new XmlBeanFactory(new ClassPathResource("beanFactoryTest.xml"));

## 实现流程
### 首先调用了ClassPathResource的构造函数
来构造了Resource资源文件的实例对象，然后后续的资源处理就可以使用Resource提供的各种服务来操作了。然后当我们有了Resource之后，就可以进行XmlBeanFactory的初始化了。  
与之同理，对于不同来源的资源文件都有相应的Resource实现。有了Resource接口之后便可以对所有资源文件进行统一的处理。例如

        Resource resource=new ClassPathResource("beanFactoryTest.xml");
        InputStream inputStream=resource.getInputStream();

### XmlBeanDefinition
然后，当Resource的相关类完成了对配置文件进行封住后配置文件的读取工作就全权交给XmlBeanDefinition来处理了。  
（ignoreDependencyInterface（）函数的主要功能是忽略给定接口的自动装配功能）  (时序图书 p20)
- 封装资源文件。
- 获取输入流
- 通过构造的InputSource实例和Resource实例继续调用函数doLoadbeanDefinitions

### 支撑着整个Spring容器部分的实现基础
然后就是3个步骤，支撑着整个Spring容器部分的实现基础
- 获取对XML文件的验证模式
- 加载XML文件，并得到对应的Document
- 根据返回的Document注册Bean信息


### doRegisterBeanDefinitions(root)
 
然后，又通过一系列的处理之后，终于可以到了核心逻辑的底部doRegisterBeanDefinitions(root)  
终于开始真正的解析了。
- 处理profile（profile可以让我们在配置文件中部署配置两套配置来适用于测试环境和开发环境，在springboot中变成了bootstrap.yml中的一种属性）
- 模板模式的使用（解析前后的函数preProcessXml（）和postProcessXml（）并没有实现，顺便复习一下面向对象的三大特性：封装多态继承，五大原则：单一功能原则，开闭原则，里氏代换原则，依赖原则，接口分离原则，迪米特原则）
- pariseBeanDefinitions方法来对bean进行解析，然后并注册BeanDefinition  

因为在Spring中的xml配置中有两大类Bean的声明：默认的和自定义的。在以后的文章中将会提及。
