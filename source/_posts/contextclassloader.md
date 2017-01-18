---
title: Java 线程上下文类加载器(ContextClassLoader)
date: 2017-01-18 11:24:04
tags: classloader
---
### 背景

&emsp;&emsp;最近使用款中间件的容器，它会自动加载所有的中间件，并使用不同的classLoader将各个中间件隔离加载，提供给应用提供服务。

&emsp;&emsp;但是这里就出现一个问题，HBase初始化客户端的是在应用ClassLoader中调用的，在Hbase客户端的初始化的时候会加载默认的配置文件，此处就加载失败了；原因是加载资源配置的是容器的modelClassLoader而在客户端调用的时候却在thread的ContextClassLoader中寻找，导致无法加载到。

&emsp;&emsp;有2种方式解决：
* 将需要的资源文件配置到应用的路径下，而不是用HBase jar包中默认的配置，
这样就能在应用的ClassLoader中寻找到。
* 在加载资源的上下文中，设置Thread的ContextClassLoader为class.getClassLoader()，
这样就能够HBase在调用Thread.getContextClassLoader()的时候，就是加载此类的modelClassLoader了,代码如下：

```
Thread currentThread = Thread.currentThread();
ClassLoader oldCl = currentThread.getContextClassLoader();
try {
currentThread.setContextClassLoader(getClass().getClassLoader());
// Load Resource
} finally {
currentThread.setContextClassLoader(oldCl);
}
```


### 线程上下文类加载器(ContextClassLoader)

&emsp;&emsp;类 java.lang.Thread中的方法 getContextClassLoader()和 setContextClassLoader(ClassLoader classLoader)用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader classLoader)方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。
最早线程上下文类加载器最主要的意义是用在SPI上的，解决双亲委派无法解决的类加载问题。

&emsp;&emsp;SPI类加载遇到最主要的问题是，SPI 的接口是 Java 核心库的一部分，是由引导类加载器来加载的；SPI 实现的 Java 类一般是由系统类加载器来加载的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。所以此时应该由谁去加载SPI的具体实现就不清楚了，因为引导类加载器已经是顶层类了，双亲委派肯定是不行的。
这个是用上下文加载器正好能解决这个问题，ServiceLoader只需要使用用线程上下文加载器去加载这个类就行不用关心，具体是哪个加载器，由负责调用的线程去设置已经加载具体实现类的ClassLoader到线程上下文就行。

&emsp;&emsp;我们具体看一下ServiceLoader的代码，也确实是这么实现的:

```
 public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
```

### 总结
  &emsp;&emsp;现在各种各样的需求存在，对自定义加载顺序的诉求也越来越多，前面的[文章](https://xytutu.github.io/2016/12/28/overrideclassloader/)提到了几种思路，线程上下文类加载器(ContextClassLoader)也是一种较好的思路。
  &emsp;&emsp; _在SPI中ContextClassLoader最本质的特征，在抽象的层次可以指定任意的ClassLoader的去加载类或者资源，这也是在ClassLoader实现抽象的一种方式_



