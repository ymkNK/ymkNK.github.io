---
layout: post
title: 单例模式的实现方式
subtitle: singleton
author: ymkNK
date: 2020-07-04 00:34:38 +0800
categories: Spring
tag: Spring
img: 2.jpg
---
## 什么是单例模式？
在整个程序运行过程中只会实现一次的对象，例如Runtime类。  
需要满足的条件有以下几点：
- 构造器私有化
- 自行创建，并且用静态变量保存
- 向外提供这种实例
- 为了强调这是单例模式，我们可以使用final修饰

## 常见形式
### 饿汉式
懒汉式就是直接就实例化了，在程序运行的时候，直接创建，不管是否需要这个对象.不存在线程安全问题
#### 饿汉式-直接实例化饿汉式
```$xslt
public class Singleton(){
    public static final INSTANCE=new Singleton();
    private Singleton(){
        // 构造器私有化
    }
}
```
#### 饿汉式-枚举式
和上面的效果一模一样
```$xslt
public enum Singleton(){
    INSTANCE,
    ;
} 

```

#### 静态代码块饿汉式
这种情况和第一种效果也是一样的，但是如果需要配置的话，就需要使用这种方式了
```$xslt
public class Singleton(){
    public static final INSTANCE;
    private String info;
    static {
        Properties pro=new Properties();
        pro.load( Singleton.class.getClassLoader().getResourceAsStream("src/下面的资源文件"))
        INSTANCE=new Singleton(pro.getProperty("info"));
    }

    private Singleton(String info){
        this.info=info;
        // 构造器私有化
    }
}
```

### 懒汉式
延迟创建，在需要的时候才创建对象
#### 线程不安全的懒汉式
这种情况就是会有线程安全的问题
```$xslt
public class Singleton(){
    private static final INSTANCE;
    private Singleton(){
        // 构造器私有化
    }
    public Singleton getInstance(){
        if(null==INSTANCE){
            INSTANCE=new Singleton();
        }
        return INSTANCE;
    }
}
```
#### 线程安全的懒汉式
加上锁，保证多线程的时候，不会有问题
```$xslt
public class Singleton(){
    private static final INSTANCE;
    private Singleton(){
        // 构造器私有化
    }
    public Singleton getInstance(){
        if(null==INSTANCE){
            synchronized(Singleton.class){
                INSTANCE=new Singleton();
            }
        }
        return INSTANCE;
    }
}
```
#### 静态内部类形式（适用于多线程）
静态内部类只有在被使用的时候，才会开始实例化。因此这个也将是线程安全的，并且只有再被需要的时候才会被加载

```$xslt
public class Singleton{
    
    private Singleton(){
        // 构造器私有化
    }

    private static class Inner{
        private static final INSTANCE=new Singleton();
    }
    public Singleton getInstance(){
       return Inner.INSTANCE;
    }
}
```