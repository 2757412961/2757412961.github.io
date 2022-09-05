---
title: java 动态代理
date: 2022-04-08
categories: 
- java
tags:
- 动态代理

---



[TOC]

# 一、代理模式简介

## 1.1、代理模式

提供了间接对目标对象进行访问的方式，即通过代理对象访问目标对象。

## 1.2、代理模式的好处

在实现目标对象功能的基础上，增加额外的功能，扩充目标对象的功能。符合**开闭原则**：即不改变既有代码的前提下，对功能进行扩展。

例如，需要在所有类的方法执行前后打印日志。

# 二、JVM 创建对象

## 对象创建过程
<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig001.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;">对象创建过程</div>


## 关于 Class 对象和实例对象

**Class 对象**：每个类运行时的类型信息由 Class 对象来表示，Class 对象中包含与这个类有关的信息。

**实例对象**：实例对象是由 Class 对象创建的。
<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig002.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;"> Class 对象和实例对象</div>

Java 程序在开始运行之前并非被完全加载，其各个类都是在必需时才加载的 —— 懒加载

## 获取 Class 对象的方式

> Class.forName("类的全限定名")

> 实例对象.getClass()

> 类名.class —— 类字面常量
>
> 如：
>
> Class clazz = int.class

如果字段被 `static final` 修饰，称为“编译时常量”。调用该字段时，不会对类进行初始化。

因为这个字段在编译期就把结果放入常量池中了。

# 三、静态代理

静态代理的逻辑：

<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig003.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;">静态代理的逻辑</div>


## 静态代理缺陷

对于每一个目标类都需要写对应的 XXProxy 代理类，如果目标类很多的话，写成百上千个代理类是不切实际的。

# 四、动态代理

## 动态代理的思路
<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig004.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;">动态代理的思路</div>

## 动态代理的优势

在程序运行时，JVM 才对被代理对象（目标对象）生成代理对象。涉及到反射。

*注意，动态代理的时候目标对象必须得实现接口，否则动态代理无法使用，原因很明显，因为代理对象本身不用实现接口，它用到的是目标对象的引用，所以如果目标对象不实现接口的话，根本没法进行代理。*

如何从 interface 创建 Class 对象？

JDK 提供：`java.lang.reflect.InvocationHandler`接口 + `java.lang.reflect.Proxy` 类

Proxy 类有一个静态方法： 

```java
public static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces)
```

入参是 ClassLoader 类加载器以及一组接口 Class 对象。出参是 Class 类型，也就是一个 Class 对象，实际就是一个代理 Class，通过这个 Class 可以创建对应入参的代理对象。
<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig005.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;"> </div>

## 动态代理的实现

```java
import java.lang.reflect.*;

public class DynamicProxy  {
  public static void main(String[] args) throws Throwable {
    // 通过 Dog 接口创建了一个代理 Class
    Class proxyClass = Proxy.getProxyClass(Dog.class.getClassLoader(), Dog.class);
    Constructor constructor = proxyClass.getConstructor(InvocationHandler.class);
    
    // 用构造器创建代理实例对象，需要传入 InvocationHandler
    Dog dog = (Dog)constructor.newInstance(new InvocationHandler() {
      
      // 调用代理对象的方法，都会调用 invoke()
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 代理需要目标对象的引用
        // 手动创建目标对象，硬编码，不推荐
        // 推荐的做法是，将目标对象作为参数传进来
        WhiteDog whiteDog = new WhiteDog();
        Object result = method.invoke(whiteDog, args);
        return result;
      }
    });
    
    dog.canRun();
    dog.canBark();
  }
}

// 重构之后
import java.lang.reflect.*;

public class DynamicProxy  {
  
  public static void main(String[] args) throws Throwable {
    WhiteDog target = new WhiteDog();
    Dog dog = (Dog) getProxy(target);

    dog.canRun();
    dog.canBark();
  }
  
  // 传入目标对象的引用 target  
  private static Object getProxy(final Object target) throws Throwable {
    Class proxyClass = Proxy.getProxyClass(Dog.class.getClassLoader(), Dog.class);
    Constructor constructor = proxyClass.getConstructor(InvocationHandler.class);
    // 入参是目标对象 target， 出参是代理对象 proxy， 类型是 Object 
    return constructor.newInstance(new InvocationHandler() {
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return method.invoke(target, args);
      }
    });
  }
}

```

实际使用的是 `Proxy.newProxyInstance()`

```java
// 传入目标对象的引用 target  
private static Object getProxy2(final Object target) throws Throwable{
return Proxy.newProxyInstance(
        target.getClass().getClassLoader(), // 类加载器
        target.getClass().getInterfaces(),	// 让代理实例对象和原对象实现同一个接口
        // 这么写其实是 匿名内部类 的写法
        new InvocationHandler() {			// 代理对象的方法最终会导向 invoke 方法
          @Override
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(target,args);
          }
        });
}
```

## 动态代理逻辑图
<center>
<img style="border-radius: 0.3125em;
box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
src="http://msy-test.oss-cn-hangzhou.aliyuncs.com/share/docs/java/dynamicProxy/Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86%20fig006.png">
<br>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
display: inline-block;
color: #999;
padding: 2px;">动态代理逻辑图</div>


# 五、Cglib 代理

## Cglib 代理的实现

前面说的动态代理需要目标类实现接口，但是如果某个类根本不实现接口怎么办呢？这时候就需要 Cglib 代理了，也称 **子类代理**。

逻辑是在内存中构建一个子类对象集成目标对象，实现目标对象的功能扩展。

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxy {

  public static void main(String[] args) {
    // 目标： 通过 Cglib 代理 BlackDog
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(BlackDog.class);
    
    enhancer.setCallback(new MethodInterceptor() {
      @Override
      public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        // 此处一定要使用 proxy 的 invokeSuper 方法来调用目标类的方法  
        return methodProxy.invokeSuper(o, objects);
      }
    });
    // 生成代理实例  
    BlackDog dog = (BlackDog)enhancer.create();
    dog.canBark();
    dog.canRun();
  }

}
```

## Cglib 代理总结

在 Spring 的 AOP 编程中: 面向切面编程

如果加入容器的目标对象**有实现接口, 用 JDK 的动态代理**

如果目标对象**没有实现接口,用 Cglib 代理**。

# 六、参考材料

[Java 动态代理作用是什么？](https://www.zhihu.com/question/20794107)

[Java中三种代理模式](https://www.cnblogs.com/jie-y/p/10732347.html)

[Java中Class对象详解](https://blog.csdn.net/dufufd/article/details/80537638)