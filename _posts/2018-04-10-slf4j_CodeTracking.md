---
layout:     post
title:      最牛日志框架slf4j如何设计和实现
subtitle:   slf4j日志框架代码浅析
date:       2018-04-10
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - slf4j
    - 日志
    - Java
---

## slf4j日志框架简介

每一个Java程序员都知道日志对于任何一个Java应用程序尤其是服务端程序是至关重要的，`java.util.logging`、`Apache log4j`、`logback`等常用的日志框架相信很多同学也已经有接触过或者用过其中的一种或几种。

SLF4J(Simple logging Facade for Java)不是一个真正的日志实现，而是一个抽象层（ abstraction layer），它允许你在后台使用任意一个日志类库。使你的代码独立于任意一个特定的日志API，这是对于API开发者的很好的思想。正式这个特性使得SLF4J迅速流行开来，事实上成为Java世界的日志标准。

官方网址：[https://www.slf4j.org/](https://www.slf4j.org/)

官方用户手册：[https://www.slf4j.org/manual.html](https://www.slf4j.org/manual.html)

下图展示了slf4j日志框架是如何对多种日志系统进行适配和支持的

![/img/slf4j/concrete-bindings.png](/img/slf4j/concrete-bindings.png)



## 代码跟踪学习

slf4j日志框架在我们实际工程中通常是使用`LoggerFactory.getLogger()`获取到日志对象`logger`，然后再根据需要使用`logger`打印不同等级的对象

```java
Logger logger = LoggerFactory.getLogger(PhoneRechargeServiceImpl.class);
logger.error("log info...");
```

slf4j日志框架实际上只是一层皮，仅仅提供了统一的日志打印接口层的封装，但是并没有具体实现打印日志的功能，如果需要真正的具有日志打印功能需要

那么，slf4j是如何绑定到一个具体日志的系统呢？是不是很好奇，接下来我们通过追踪源代码来看看这个到底是如何做到的。

```java
public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }
public static Logger getLogger(Class<?> clazz) {
        Logger logger = getLogger(clazz.getName());
        if (DETECT_LOGGER_NAME_MISMATCH) {
            Class<?> autoComputedCallingClass = Util.getCallingClass();
            if (autoComputedCallingClass != null && nonMatchingClasses(clazz, autoComputedCallingClass)) {
                Util.report(String.format("Detected logger name mismatch. Given name: \"%s\"; computed name: \"%s\".", logger.getName(),
                                autoComputedCallingClass.getName()));
                Util.report("See " + LOGGER_NAME_MISMATCH_URL + " for an explanation");
            }
        }
        return logger;
    }    
```



`getLogger()`支持`Class`和`String`两种类型的参数，事实上`Class`类型参数的方法也是通过`clazz.getName()`调用了`String`参数类型的`getLogger()`方法，底层都是通过`getILoggerFactory()`拿到具体的日志实现，下面来看看该方法代码实现。

```java
public static ILoggerFactory getILoggerFactory() {
        if (INITIALIZATION_STATE == UNINITIALIZED) {
            synchronized (LoggerFactory.class) {
                if (INITIALIZATION_STATE == UNINITIALIZED) {
                    INITIALIZATION_STATE = ONGOING_INITIALIZATION;
                    performInitialization();
                }
            }
        }
        switch (INITIALIZATION_STATE) {
        case SUCCESSFUL_INITIALIZATION:
            return StaticLoggerBinder.getSingleton().getLoggerFactory();
        case NOP_FALLBACK_INITIALIZATION:
            return NOP_FALLBACK_FACTORY;
        case FAILED_INITIALIZATION:
            throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
        case ONGOING_INITIALIZATION:
            // support re-entrant behavior.
            // See also http://jira.qos.ch/browse/SLF4J-97
            return SUBST_FACTORY;
        }
        throw new IllegalStateException("Unreachable code");
    }
```



slf4j日志初始化过程是在`performInitialization()`中进行的，初始化成功则返回该绑日志实现的`LoggerFactory`，下面来看看最重要的`performInitialization()`过程是如何初始化日志的。

其实初始化过程非常的简单，只有简单的两步：

第一步，进行日志绑定，我们继续看`bind()`方法

第二步，进行版本校验，参见`versionSanityCheck()`方法

```java
private final static void performInitialization() {
        bind();
        if (INITIALIZATION_STATE == SUCCESSFUL_INITIALIZATION) {
            versionSanityCheck();
        }
    }
```





`bind()`方法代码如下所示：

```java
private final static void bind() {
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            // skip check under android, see also
            // http://jira.qos.ch/browse/SLF4J-328
            if (!isAndroid()) {
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding
            StaticLoggerBinder.getSingleton();
            INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
            reportActualBinding(staticLoggerBinderPathSet);
            fixSubstituteLoggers();
            replayEvents();
            // release all resources in SUBST_FACTORY
            SUBST_FACTORY.clear();
        } catch (NoClassDefFoundError ncde) {
            String msg = ncde.getMessage();
            if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
                INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
                Util.report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
                Util.report("Defaulting to no-operation (NOP) logger implementation");
                Util.report("See " + NO_STATICLOGGERBINDER_URL + " for further details.");
            } else {
                failedBinding(ncde);
                throw ncde;
            }
        } catch (java.lang.NoSuchMethodError nsme) {
            String msg = nsme.getMessage();
            if (msg != null && msg.contains("org.slf4j.impl.StaticLoggerBinder.getSingleton()")) {
                INITIALIZATION_STATE = FAILED_INITIALIZATION;
                Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
                Util.report("Your binding is version 1.5.5 or earlier.");
                Util.report("Upgrade your binding to version 1.6.x.");
            }
            throw nsme;
        } catch (Exception e) {
            failedBinding(e);
            throw new IllegalStateException("Unexpected initialization failure", e);
        }
    }
```



重点关注到`findPossibleStaticLoggerBinderPathSet()`方法是来寻找和绑定可能的`StaticLogger`集合

同时请注意到`StaticLoggerBinder.getSingleton();`这行，不知道大家有没有发现`StaticLoggerBinder`这个类在`slf4j-api.jar`中是没有定义的，神奇的是代码怎么编译通过的？请继续往下看，我们将揭晓这个谜底。



```java
private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class";
static Set<URL> findPossibleStaticLoggerBinderPathSet() {
        // use Set instead of list in order to deal with bug #138
        // LinkedHashSet appropriate here because it preserves insertion order
        // during iteration
        Set<URL> staticLoggerBinderPathSet = new LinkedHashSet<URL>();
        try {
            ClassLoader loggerFactoryClassLoader = LoggerFactory.class.getClassLoader();
            Enumeration<URL> paths;
            if (loggerFactoryClassLoader == null) {
                paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
            } else {
                paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
            }
            while (paths.hasMoreElements()) {
                URL path = paths.nextElement();
                staticLoggerBinderPathSet.add(path);
            }
        } catch (IOException ioe) {
            Util.report("Error getting resources from path", ioe);
        }
        return staticLoggerBinderPathSet;
    }
```

代码中可以看到`slf4j`是通过`ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH)`载入具体日志实现类的

看到这里`STATIC_LOGGER_BINDER_PATH`这个静态变量对应的值为`org/slf4j/impl/StaticLoggerBinder.class`了吧。没错！绑定的就是这个class，也就是上面`slf4j-api.jar`中没有定义的类。而这个类定义在哪里呢？答案是在具体实现日志绑定的jar中，我们可以打开常见的日志框架绑定slf4j的jar包看看。

`slf4j-log4j.jar`的`StaticLoggerBinder`实现

![/img/slf4j/slf4j-log4j.jpg](/img/slf4j/slf4j-log4j.jpg)

`logback-classic.jar`的`StaticLoggerBinder`实现

![/img/slf4j/logback-classic.jpg](/img/slf4j/logback-classic.jpg)





目前支持的日志实现框架有*`slf4j-log4j12`*、*`slf4j-jdk14`*、*`slf4j-nop`*、*`slf4j-simple`*、*`slf4j-jcl`*、*`logback-classic`*等，当然只要你愿意，也可以写一个自己的实现。

实际项目中，存在多个日志实现框架时，只能选择其中一个具体实现，需要将其他实现的`slf4j`适配支持的包排除掉，并将其他的具体实现日志系统直接打印的日志桥接到`slf4j`上，这样系统就可以统一在一个日志框架下打印日志。

如何将其他具体日志框架直接打印的日志桥接到slf4j上呢？我们可以参见下图：

![/img/slf4j/redirectToSlf4j.png](/img/slf4j/redirectToSlf4j.png)





## 正确姿势

1、最常见的不当姿势：

```java
logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
```

2、改进姿势：

```java
if(logger.isDebugEnabled()) {
  logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
}
```

3、较优雅的姿势是采用`{}`占位符参数化变量：

```java
Object entry = new SomeObject();
logger.debug("The entry is {}.", entry);
```

原因：采用`方式1`打日志时，当系统的日志级别不需要打印debug日志时还是会进行字符串拼接操作，而`String`为不可变类型，采用`+`进行字符串连接会增大系统开销，这在频繁打印日志的系统中，改进姿势2打日志每次都要额外进行日志级别判断，非常不方便，**强烈建议采用`{}`占位符的方式打印日志**。

如果占位变量是一个复杂对象，会在实际打印日志时调用对象的`toString()`方法。



## 如何实现一套兼容SLF4J的日志系统

1、org.slf4j.Logger接口适配实现

自定义一个`org.slf4j.Logger`接口的实现类，可以参见slf4j-jcl, slf4j-jdk14和slf4j-log4j12中的实现实例进行适配。

2、org.slf4j.ILoggerFactory接口适配实现

自定义一个`org.slf4j.ILoggerFactory`接口实现一个工厂类`MyLoggerFactory`，这个工厂会返回一个`MyLoggerAdapter`实例。

3、修改StaticLoggerBinder类

StaticLoggerBinder类会返回一个日志工厂类(`ILoggerFactory`)，修改StaticLoggerBinder类中的`loggerFactoryClassStr`变量

`Marker`和`MDC`支持可以参考`NOP`中现成的实现

总结起来，创建一个支持slf4j的日志系统大致需要如下几步：

1. 拷贝一份现有的适配实现
2. 创建一个适配器（adapter），让日志系统与org.slf4j.Logger接口适配
3. 创建一个步骤2中适配器（adapter）的工厂类
4. 修改StaticLoggerBinder类，会用到步骤3中创建的工厂类

## 参考

参考：

<https://blog.csdn.net/kxcfzyk/article/details/38613861>

<https://blog.csdn.net/winwill2012/article/details/71786004>

<https://blog.csdn.net/u011375296/article/details/41170637>





