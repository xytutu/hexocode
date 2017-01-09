---
title: Java自定义Class加载顺序
date: 2016-12-28 20:47:26
tags: Java
---

在Java中默认的Class加载顺序都是父类委托加载，但是有些时候需要自定义加载顺序比如OSGI和阿里的Pandora。
其中最常见的问题是有2个一模一样的类，到底加运行哪一个。

### 打破双亲加载

* 父类委托加载的基本实现思路，loadClass首先委托给parent中查找class要是找不到，再使用本身的classLoader去寻找，所以默认的顺序基本上是从上向下找。

* 普通自定义classLoader只是重写findClass去加载自己路径下的class。但是要想打破父类委托机制去自定义前后顺序去加载class就要完全自己去实现loadClass方法控制加载顺序，不是首先委派给parent而是在自己返回或者查找持有的其他classLoader中的类。

### 设计自己的ClassLoader
* 基本思路就是，自定义classLoader，然后将自定义的classLoader设置为extClassLoader以下所有classLoader的parent，然后利用双亲加载（除去jdk class）其他的class就会加载自己的class，此ClassLoader已经是appClassLoader中最顶层的ClassLoader了，所有冲突的类都会运行其中实现，防止了类冲突的问题。
