---
layout: post
title: Yaml和Java对象之间的相互转换
subtitle: yamlToJava
author: ymkNK
date: 2019-02-22 21:46:09 +0800
categories: Tech
tag: Tech
img: 3.jpg
---
# 前言
最近需要做一个将java对象转换成yaml文件格式的工作，由于手动进行操作的话效率会很低下，因此再找有没有合适的包可以使用。最后找到的工具就是yamlBeans,本文主要是翻译整理yamlBeans中的一部分信息
>[yamlBeans官网](http://yamlbeans.sourceforge.net/)  
>[yaml文档的学习](https://www.jianshu.com/p/97222440cd08)

# 添加依赖
将下列代码添加到pom文件当中

    <dependency>
       <groupId>com.esotericsoftware.yamlbeans</groupId>
       <artifactId>yamlbeans</artifactId>
       <version>1.12</version>
    </dependency>

# Yaml反序列化为常见数据结构
所谓的反序列化，就是将yaml格式的文件，转换为java的常见数据结构
主要使用的类是YamlReader，然后可以将yaml文档转换为HashMap，ArrayList,还有Strings类型。  
yaml：

    name: Nathan Sweet
    age: 28
    address: 4011 16th Ave S
    phone numbers:
     - name: Home
       number: 206-555-5138
     - name: Work
       number: 425-555-2306

code:

    YamlReader reader = new YamlReader(new FileReader("contact.yml"));
    Object object = reader.read();
    System.out.println(object);
    Map map = (Map)object;
    System.out.println(map.get("address"));

# Yaml反序列化为POJO
将yaml文档反序列化为pojo(plain ordinary java object)

Yaml and java class:

    name: Nathan Sweet
    age: 28

    public class Contact {
    	public String name;
    	public int age;
    }

Code:

    YamlReader reader = new YamlReader(new FileReader("contact.yml"));
    Contact contact = reader.read(Contact.class);
    System.out.println(contact.age);

# 序列化为Yaml文档
这个地方就是与上方的反着来的，就是将java的常见的数据结构（HashMap,Sring,ArrayList都是可以的）或者pojo转换为yaml的文档，这里主要使用到的类是YamlWriter()

example:

    String yaml="";
    StringWriter stringWriter = new StringWriter();
    YamlWriter writer = new YamlWriter(stringWriter);
    try {
        writer.write(greyTask);
        writer.close();
    } catch (YamlException e) {
        log.error("Grey Yaml Error",e);
    }
    yaml = stringWriter.toString();

    return yaml;

上方的是我自己写的例子，YamlWriter的构造函数所需要的参数，是一个writer（可以是fileWriter也可以是StringWriter，当然其他类型的writer也是可以的）

输出的图像如下(fileWriter的)：
![YamlWriter](https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/yamlBeans/yamlWriter.png)

#### 设置tag
有的时候tag会特别长，这个时候，就可以将自定义设置tag

code:

    YamlWriter writer = new YamlWriter(new FileWriter("output.yml"));
    writer.getConfig().setClassTag("contact", Contact.class);
    writer.write(contact);
    writer.close();

result:

    !contact
    name: Nathan Sweet
    age: 28

# 锚点Anchor
当出现指向同一个对象的时候，就会使用锚点

yaml:

    oldest friend:
        &1 !contact
        name: Bob
        age: 29
    best friend: *1

上面的yaml文件中，&1 ！contact相当于定义了contact为1，然后使用*1来进行引用。默认来讲，yamlWriter在进行转换的时候，如果遇到了相同的对象会自动的输出锚点。

    Contact contact = new Contact();
    contact.name = "Bob";
    contact.age = 29;
    Map map = new HashMap();
    map.put("oldest friend", contact);
    map.put("best friend", contact);
