---
title: Java多线程（3）——线程间通信
date: 2017-09-18 10:34:14
tags:
- Java
- 多线程
- 并发
- 性能
comments: true
categories: 转载
---
&nbsp;&nbsp;&nbsp;&nbsp;本文主要学习线程间相互通信的内容。线程见需要通信，才能协同完成工作，虽然这增加的这里的复杂度，也很容易出错，但是线程间通信是很重要也很不可缺少的功能。

---
# 等待与通知

## wait、notify介绍
&nbsp;&nbsp;&nbsp;&nbsp;如果看一眼java最基础的一个类Object的源码的话，会发现Object类有两个方法，wait、notify。所有的类都是默认继承Object类的，所以我们创建的所有的类都有这两个方法。
<!-- more -->
&nbsp;&nbsp;&nbsp;&nbsp;举个例子，我们去麦当劳、或者奶茶店，买好了之后会得到小票，这时候店家会开始准备我们的餐饮，我们可以不用一直顶着店家看的，我们就站在一边玩会手机（这其实就是多线程，你在等待取餐的时间内去干了别的），等到东西做好了，服务员会叫号，说xxxx号好了，这时候我们一听，是我们的东西好了，于是放下手机去取餐。

&nbsp;&nbsp;&nbsp;&nbsp;这个例子就是等待通知。我们买好餐之后，拿到小票，就在一边等着，这时候就相当于调用了wait方法，当餐好了，服务员叫我们，这相当于调用了notify方法，我们收到之后就停止wait去取餐了。

&nbsp;&nbsp;&nbsp;&nbsp;所以这里的一个线程，就是我们点餐取餐，示例代码如下。
```bash
String lock = "你的餐";
synchronized (lock) {
    // 点餐 付款
    lock.wait();
    // 取餐 吃饭
}
```
还有一个线程，是服务员，示例代码如下。
```bash
String lock = "你的餐";
synchronized (lock) {
    // 备餐
    lock.notify();
    // 确认你的小票
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这里我们付款之后，wait等待（阻塞当前线程），这里的线程是点餐取餐的过程，而我们自己相当于一个cpu，这时候可以分享cpu去执行其他线程的工作（比如玩手机）。服务员备餐，当备餐好了之后，调用notify，然后看看你的小票，确认没问题把餐给你他就走了。这时候你从阻塞状态恢复回来，取餐然后去吃饭。

&nbsp;&nbsp;&nbsp;&nbsp;这里特别说明一下，**wait调用之后，会释放锁（同时阻塞），这样其他线程才能获得锁去工作。而notify执行之后不会立刻释放锁，需要同步语句块所有内容执行完之后才会释放锁，wait后面的代码才能开始执行**。所以上面的例子中，服务员叫你的号了，但是你不能立刻把餐拿走，需要先确认你的小票是不是这个号，确认完了他的工作才算结束，你才能拿餐。

## 等待通知必须在同步语句块中
&nbsp;&nbsp;&nbsp;&nbsp;可以是同步语句块，也可以是synchronized修饰的方法中，总是需要处于同步状态下，而这个等待、通知的主体也必须是同步状态下监视的对象。如果不在同步状态下，会抛出IllegalMonitorStateException异常。

&nbsp;&nbsp;&nbsp;&nbsp;而且向我们上面的例子，同步监控的是lock字符串，那么wait、notify的主体也就必须是lock；对于synchronized修饰的方法而言，由于锁的是当前对象，所以主体应该是this。

## 线程状态介绍
&nbsp;&nbsp;&nbsp;&nbsp;线程的状态有new、runnable、running、blocked/time wait/sleeping、terminated。

&nbsp;&nbsp;&nbsp;&nbsp;其中新建一个线程就是new，然后调用start方法，线程会进入runnable（可运行）状态，但是这时候线程可能还没开始运行，因为他要争抢cpu资源，所以不一定你调用了start方法，这个线程就可以启动了。

&nbsp;&nbsp;&nbsp;&nbsp;当线程的争抢到cpu资源了，那么他就会进入running（运行中）状态。当然runnable和running可能会互相转换的，如果有更高优先级的线程争抢到了cpu资源，那么这个线程可能会进入到runnable状态。线程进入runnable状态有如下可能：

&nbsp;&nbsp;&nbsp;&nbsp;a、调用sleep之后经过的时间超过了指定的sleep时间（sleep结束之后重新进入runnable状态争抢cpu资源）；
&nbsp;&nbsp;&nbsp;&nbsp;b、线程调用的阻塞IO已经返回，阻塞方法执行完毕；
&nbsp;&nbsp;&nbsp;&nbsp;c、线程成功的获得了试图同步的监视器；
&nbsp;&nbsp;&nbsp;&nbsp;d、线程正在等待某个通知，其他线程发出了通知；
&nbsp;&nbsp;&nbsp;&nbsp;e、处于挂起（suspend）状态的线程调用了resume恢复方法。

&nbsp;&nbsp;&nbsp;&nbsp;blocked是阻塞的意思，time wait是处于等待的状态，sleeping是处于sleep状态。这三个状态通常统称为一种状态，他们比较相似。blocked比如遇到了一个IO操作，需要等待，而其他线程争抢到了cpu资源，这时候当前线程就处于blocked状态了。time wait比较好理解吧，就是处于wait阻塞的状态。sleeping就是主动调用的sleep方法处于等待状态。所以总结起来，可能引起进入blocked状态的情况有下面5种：

&nbsp;&nbsp;&nbsp;&nbsp;a、线程调用sleep方法主动放弃；
&nbsp;&nbsp;&nbsp;&nbsp;b、调用wait方法等待通知；
&nbsp;&nbsp;&nbsp;&nbsp;c、调用了阻塞IO，等待返回（这里比如发起一个http请求，需要等待服务员响应）；
&nbsp;&nbsp;&nbsp;&nbsp;d、线程试图获得某个对象监视器，但是这个同步对象正在被别的线程持有；
&nbsp;&nbsp;&nbsp;&nbsp;e、调用了suspend主动挂起（这种方法已经被废弃，容易引起死锁，建议弃用）。

&nbsp;&nbsp;&nbsp;&nbsp;当run里面的内容都运行完了，线程的工作也就结束了，这时候他就可以销毁了，进入了terminated状态。

&nbsp;&nbsp;&nbsp;&nbsp;这里额外说一点，**wait和sleep其实有这一点相似，就是在阻塞状态时，调用了线程的interrupt方法，会出现InterruptedException异常**。

## 其他补充
&nbsp;&nbsp;&nbsp;&nbsp;当一个对象监视器有多个线程正在wait的时候，这时候某个线程调用了notify之后，所有wait状态的线程开始争抢cpu资源，其中只有一个线程可以从wait状态进入运行状态，所以要想都唤醒，那就需要调用n遍notify方法，或者，调用notifyAll方法，这里还是要看具体的业务场景了。

&nbsp;&nbsp;&nbsp;&nbsp;wait方法还有个带参的方法，参数可以是个long，意思就是等待n毫秒，如果还没有被唤醒，那么就自己醒。当然如果在n毫秒以内也可以被notify唤醒。

&nbsp;&nbsp;&nbsp;&nbsp;假设你设计的程序中一个工作中只会执行一次wait、notify的话，那么一定要注意他们是不是一定会按照顺序执行的，假如先执行了notify，那么在执行wait的话就没人唤醒他了。

---
# 线程通信具体方式

## wait、notify最简单的使用
&nbsp;&nbsp;&nbsp;&nbsp;这种方式也很简单，就不细说了。就是一个线程等待获取另一个线程的数据的时候，首先在需要调用数据的前面执行wait，另外一个线程写入数据，写入之后执行notify方法通知等待的线程获取数据。这样就完成了线程间最简单的通信了。

&nbsp;&nbsp;&nbsp;&nbsp;其实再复杂一点的话，就是读取数据这边，在读取完了之后又要通知写数据的线程写数据，数据两个线程其实在读写数据前后都需要wait和notify方法了。而这里都要对数据这块进行加锁。这就是复杂之处了，由于写数据的线程不会只有一个，读数据的线程也不会只有一个。所以有可能这个写线程的notify唤醒的是另一个写线程的wait，这就出错了，而且也可能导致所有读线程的wait都得不到唤醒而产生死锁。这里有个简单的解决办法，就是调用notifyAll，这样读线程也会读取了，而写的时候判断一下是否被写过，读的时候判断下是否被读过，这里的判断有很多办法，交给大家自己去实验一下了（其实加变量就可以了）。

## 管道通信，字节流、字符流
&nbsp;&nbsp;&nbsp;&nbsp;Java中提供了很多输入输出流，Stream，可以方便我们对数据进行操作。JDK提供了两组类来实现线程间通信，分别是PipedInputStream与PipedOutputStream、PipedReader与PipedWriter。 关于管道的知识这里就不详细介绍了，大家有兴趣可以自己搜索学习下。

---
# join方法

## join的作用
&nbsp;&nbsp;&nbsp;&nbsp;join最重要的一个作用就是等待当前线程执行完毕。举个例子。
```bash
public static void main(String[] args) {
    TestThread t = new TestThread();
    t.start();
    t.join();
    System.out.println("我想最后说");
}
```
&nbsp;&nbsp;&nbsp;&nbsp;正常情况下，如果没有这个join的调用的话，这个打印语句有可能是先于或者在线程运行中执行的，而调用了join方法之后，这个打印语句就会在t这个线程彻底执行完之后再打印了。

&nbsp;&nbsp;&nbsp;&nbsp;其实这有很多作用，这里的main方法实际上是主线程，而t是子线程。如果我们不加这个join，实际上主线程会先于子线程结束。有时候我们需要等子线程执行完，比如修改一个字符串的内容，主线程再去获取修改后的内容，所以一定要等子线程执行完才可以。

## join的原理
&nbsp;&nbsp;&nbsp;&nbsp;知道了join的作用之后，就需要知道他的原理了。实际上很简单，join的内部是通过wait实现的。所以join的一个特性我们就知道了，会被interrupt打断，和wait一样。

## join其他内容
&nbsp;&nbsp;&nbsp;&nbsp;join内部调用的wait，所以他也有一个join(long)的方法，这个方法和wait一样，也就是join多少毫秒，如果在这时间之后，线程还是没有结束，那么就不等了。

&nbsp;&nbsp;&nbsp;&nbsp;join(long)和sleep(long)区别是什么呢，首先一个是如果在时间内线程执行结束，join等待的时间更少。另外一个就是join由于内部使用的是wait，所以在调用join之后，实际上是调用了对象的wait方法，所以会释放当前对象的锁，其他线程就可以获取锁了，可操作的内容就更多了。

&nbsp;&nbsp;&nbsp;&nbsp;另外，由于join内部调用的是wait，所以当被notify是，他同样需要和其他当前对象正在wait状态的线程进行锁争抢，所以有的时候也可能产生意外。

---
# ThreadLocal类
&nbsp;&nbsp;&nbsp;&nbsp;不知道大家有没有了解过这个类，其实我在最开始学mvc的时候就学到过，这个类就是一个线程共享变量的类。什么叫做线程共享变量呢？

&nbsp;&nbsp;&nbsp;&nbsp;对于我们使用public static修饰的，是个静态变量，大家都可以调用，而且就这一份。而线程共享变量，就是一个线程独享的变量。就算这个变量的声明就那一份，但是每个线程对这个变量的访问是互不干扰的。

&nbsp;&nbsp;&nbsp;&nbsp;所以回到我最初说的，就这一个变量，我们在controller里面去写个值，在service、dao层去访问这个变量都是可以得到的，而多个用户的访问又是互不相干的。而一次访问映射的其实就是一个线程，controller去调用service、service调用dao，再到底层实现，都是一个在一个线程里的。

## get、set
&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocal主要的两个方法就是get和set了，很简单，set放值，get取值。而ThreadLocal支持泛型，所以可以存取任何类型的对象。下面是示例。
```bash
public class ThreadTool {
    public static ThreadLocal<String> tl = new ThreadLocal<String>();
}
 
public class Run {
    public static void main(String[] args) {
        ThreadTool.tl.set("a");
        System.out.println(ThreadTool.tl.get());
    }
}
```

## 默认方法的重写
&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocal有个可重写的方法，一个是initialValue方法，这个方法返回默认值。如果我们没有set值的话，返回的会是null，而重写了这个方法，可以返回我们指定的默认值，当然一般情况下我们是没有必须要重写的。

## InheritableThreadLocal
&nbsp;&nbsp;&nbsp;&nbsp;通过这个类名我们应该也可以知道，这个类其实和继承有关（其实不是）。其实是这个类可以让子线程可以继承父线程的内容。这样，子线程和父线程通过这个类，就可以共享变量了。这个比较简单，使用方法和TheadLocal一样，就不举例了。

&nbsp;&nbsp;&nbsp;&nbsp;但是这个就会出现坑了，由于正常情况下我们的ThreadLocal是在一个线程中使用的，并发问题都是出在多线程中的，所以ThreadLocal并不会出现并发访问问题，而InheritableThreadLocal可能会出现在多个线程中了（父线程可以起多个子线程），所以线程多了，还是可能出现脏读之类的问题的，这点要注意。

&nbsp;&nbsp;&nbsp;&nbsp;到此我们对于线程通信的内容就介绍完了。

---
转载自：[赵伊凡's Blog](http://irfen.me/java-multi-thread-3/)