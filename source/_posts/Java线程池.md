---
title: Java线程池
date: 2017-10-26 18:54:01
tags:
- Java
- 线程池
- ThreadPoolExecutor
comments: true
categories: 原创
---
&nbsp;&nbsp;&nbsp;&nbsp;Java在JDK1.5之后加入了java.util.concurrent包，这包中有一个接口Executor，它是Java线程池的顶级接口。但在严格意义上讲它并不是一个线程池，而是一个执行线程的工具。真正的线程池接口是ExrcutorService。<br>
&nbsp;&nbsp;&nbsp;&nbsp;那么我们为什么要使用线程池呢，我们都知道，每当我们创建一个线程的时候，都会耗费相应的时间和空间。而线程池的主要作用就是减少时间和空间的<!-- more -->开销。即：<br>
- **减少创建和销毁线程的次数**，每个工作线程都可以被重复利用，可执行多个任务。
- 可以根据系统的承受能力，**调整线程池中工作线程的数目，防止消耗过多的内存**（JDk1.5以后每个线程堆栈大小为1MB）。

<br>

# 创建线程池
&nbsp;&nbsp;&nbsp;&nbsp;创建一个线程池比较复杂，Executor类里面提供了一些静态工厂，生成一些常用的线程池。<br>
1. **newSingleThreadExecutor**：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。使用LinkedBlockingQueue（无界队列）。
2. **newFixedThreadPool**：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。使用LinkedBlockingQueue（无界队列）。
3. **newCachedThreadPool**：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。使用SynchronousQueue。
4. **newScheduledThreadPool**：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

---

# ThreadPoolExecutor类
&nbsp;&nbsp;&nbsp;&nbsp;java.util.concurrent.ThreadPoolExecutor类是线程池中最核心的一个类，他是类ExecutorService的默认实现，想要更深的了解Java中的线程池，必须先了解这个类。<br>


## ThreadPoolExecutor类的主要参数
&nbsp;&nbsp;&nbsp;&nbsp;在ThreadPoolExecutor类中，有几个极为重要的参数：
- **corePoolSize**：线程池中保存的线程数，包括运行线程和空闲线程。当池中的线程达到corePoolSize的时候，就会把新到达的任务放在缓存队列当中。
- **maximumPoolSize**：线程池的最大线程数，他决定了线程池最多能创建多少个线程。当corePoolSize和maximumPoolSize相同时，将创建一个固定大小的线程池。
- **keepAliveTime**：表示空闲线程保留的最长时间，达到keepAliveTime时，线程则会终止。  在默认情况下，当线程池中的线程数量大于corePoolSize设置的值时，keepAliveTime才会起作用，直到线程池中线程数量不大于corePoolSize。
- **unit**：参数keepAliveTime的时间单位。有7种取值，在TimeUnit类中有7种静态属性，分别为：天（DAYS）、小时（HOURS）、分（MINUTES）、秒（SECONDS）、毫秒（MILLISECONDS）、微秒（MICROSECONDS）、纳秒（NANOSECONDS）。
- **workQueue**：一个阻塞队列，用来存储等待执行的任务。此队列仅保持由execute方法提交的Runnable任务。这里的阻塞队列有以下几种选择：ArrayBlockingQueue、LinkedBlockingQueue和SynchronousQueue。一般使用LinkedBlockingQueue和SynchronousQueue。如果使用无界queue，那么maximumPoolSize是没有意义的。
- **threadFactory**：线程工厂，主要用于创建新线程。
- **handler**：由于超出线程范围和队列容量而使执行被阻塞时 所使用的处理程序。

&nbsp;&nbsp;&nbsp;&nbsp;**注意**：<br>
&nbsp;&nbsp;&nbsp;&nbsp;如果运行的线程少于corePoolSize，则Executor首选添加新的线程，而不选择排队。也就是说，当运行线程数小于corePoolSize，新的线程直接开始运行，不会存放到workQueue队列中。

&nbsp;&nbsp;&nbsp;&nbsp;如果运行的线程多于corePoolSize，则Executor首选将请求添加进队列，**不添加新的线程**。

&nbsp;&nbsp;&nbsp;&nbsp;如果无法添加进队列（队列已满），则创建新的线程。若创建此线程将超出maximumPoolSize，则交由handler处理，handler一般有4种处理方法，分别为：<br>
&nbsp;&nbsp;&nbsp;&nbsp;1、直接抛出异常，这是默认策略；<br>
&nbsp;&nbsp;&nbsp;&nbsp;2、用调用者所在的线程来执行任务；<br>
&nbsp;&nbsp;&nbsp;&nbsp;3、忽略该任务，直接丢弃；<br>
&nbsp;&nbsp;&nbsp;&nbsp;4、删除阻塞队列中最靠前的任务，并将此任务添加进阻塞队列。<br>

---
<br>

## ThreadPoolExecutor中的方法
&nbsp;&nbsp;&nbsp;&nbsp;首先说说几个类的关系。在ThreadPoolExecutor的源码中，有：

```
public class ThreadPoolExecutor extends AbstractExecutorService {...}
```
说明类ThreadPoolExecutor继承自类AbstractExecutorService，接下来，我们再看看类AbstractExecutorService：

```
public abstract class AbstractExecutorService implements ExecutorService {...}
```
他实现了接口ExecutorService，而ExecutorService又继承自Execute。**ThreadPoolExecutor类是Executor类的底层实现**。自此，和ThreadPoolExecutor有关的几个类和接口的关系基本上清楚了。

&nbsp;&nbsp;&nbsp;&nbsp;现在，说说类ThreadPoolExecutor中的几个核心方法：<br>

- **execute**：该方法实际上是Execute类中的方法，在ThreadPoolExecutor中进行了具体的实现。它是ThreadPoolExecutor的**核心方法**，通过这个方法向线程池提交一个任务，交由线程池去处理。
- **submit**：submit()方法是在ExecutorService中声明的方法，在AbstractExecutorService时就有了具体的实现。这个方法也是用来向线程池提交任务的，但是它与execute不同，它能返回任务执行的结果。
- **shutdown**和**shutdownNow**：根据jdk帮助文档：shutdown是启动有序关闭，先前提交的任务会被执行，但是不会接受新的任务。而shutdownNow是停止所有任务，并且返回正在等待执行的任务列表。从此方法返回时，这些任务将从任务列表中删除。

---
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)