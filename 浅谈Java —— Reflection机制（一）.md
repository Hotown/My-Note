# 浅谈Java —— Reflection机制（一）

## 概述

反射(Reflection)是Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

这就意味着，Java语言在运行时，拥有自观能力，通过这种能力可以了解自身，以便为下一步操作做准备。

反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

接下来从几个方面来探讨一下Java的反射机制

+ 反射的用途

+ 反射的基本运用


## 反射的用途

 Spring的核心部分，IOC的实现就是通过反射机制实现的。
    
在实例化一个类的时候，Spring会通过反射机制调用类的set方法将事先保存在HashMap中的类属性注入到类中。（借鉴自[t_man的专栏](http://blog.csdn.net/it_man/article/details/4402245)）
    
因为初次接触，这里就不再例举反射机制的其他用途，留待后续补充……

## 反射的基本运用

`Java的反射机制借助于4个类：class，Constructor，Field，Method`

在Java运行环境中，对于任何一个类，可以获取到这个类的属性和方法。这样动态获取类信息以及动态调用对象的方法的功能源于`Java Refleciton`

Java反射机制主要提供了
```
在运行时判断任意一个对象所属的类(class)
在运行时构造任意一个类的对象(Constructor)
在运行时判断任意一个类所具有的成员变量和方法(Field)
在运行时调用任意一个对象的方法(Method)
```

1. 得到构造器

    ```
    Constructor getConstructor(Class[] params) -- 获得使用特殊的参数类型的公共构造函数， 
     
    Constructor[] getConstructors() -- 获得类的所有公共构造函数 
     
    Constructor getDeclaredConstructor(Class[] params) -- 获得使用特定参数类型的构造函数(与接入级别无关) 
     
    Constructor[] getDeclaredConstructors() -- 获得类的所有构造函数(与接入级别无关)
    ```

2. 获得字段信息

    ```
    Field getField(String name) -- 获得命名的公共字段
    Field[] getFields() -- 获得所有公共字段
    Field getDeclaredField(String name) -- 获得类声明的命名的字段
    Field[] getDeclaredFields() -- 获得类声明的所有字段
    ```

3. 获得方法信息

    ```
    Method getMethod(String name, Class[] params) -- 使用特定的参数类型，获得命名的公共方法 
     
    Method[] getMethods() -- 获得类的所有公共方法 
     
    Method getDeclaredMethod(String name, Class[] params) -- 使用特写的参数类型，获得类声明的命名的方法 
     
    Method[] getDeclaredMethods() -- 获得类声明的所有方法
    
    ```
    
    




