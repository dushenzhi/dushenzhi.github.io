---
layout:     post
title:      将ThreadLocal传递到子线程
subtitle:   Java线程局部变量传递
date:       2016-12-11
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - Java
    - 多线程
    - ThreadLocal
---

在项目开发的过程中，我们常常会把一些常用的线程上下文信息放到ThreadLocal中(如Spring中的RequestContextHolder)，方便在程序中随时调取。但是在使用多线程时父线程中的ThreadLocal通常无法直接传递到子线程中去，容易造成程序bug。 
这种情况通常有两种方式将父线程中的ThreadLocal传递到子线程中。


## 方法一
最常规的想法是在编写子线程任务时，每次都手动的将子线程需要用到的ThreadLocal数据传递到子线程中，这样子线程也能过随时获取到线程上下文信息。

## 方法二

自定义一个ThreadPoolExecutor代替系统的ThreadPoolExecutor，每次用线程池提交线程任务时，线程池会自动将父线程的ThreadLocal自动传递到子线程中，避免每次手动传递ThreadLocal到子线程。

代码如下所示：
```java
public class TraceThreadPoolExecutor extends ThreadPoolExecutor {

    public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory);
    }

    public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, handler);
    }

    public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
    }

    /**
     * 覆盖execute方法,将一些上下文信息传递到子线程,如登陆用户信息等
     */
    @Override
    public void execute(final Runnable command) {
        final LoginUser loginUser = getLoginUser();
        Runnable task = new Runnable() {
            @Override
            public void run() {
                SessionInfoContextHolder.setLoginInfo(loginUser);
                try{
                    command.run();
                } catch (Exception e){
                //dosomething
                } finally {
                    clearLoginInfo();
                }
            }
        };
        super.execute(task);
    }

}
```


csdn博客文章地址:[https://blog.csdn.net/dushenzhi/article/details/52770379](https://blog.csdn.net/dushenzhi/article/details/52770379)
