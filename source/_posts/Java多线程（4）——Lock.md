---
title: Java多线程（4）——Lock
date: 2017-09-18 10:59:41
tags:
- Java
- 多线程
- 并发
- 性能
comments: true
categories: 转载
---
&nbsp;&nbsp;&nbsp;&nbsp;java的多线程有两种方式，我们先前面学习了基础的synchronized方式，大家忘记了建议先去回忆一下再来看这部分。Lock其实对应着synchronized的方式加锁，但是更加灵活，本节讲的时候会对照着synchronized相关的知识来说。

---
# ReentrantLock类
&nbsp;&nbsp;&nbsp;&nbsp;Java中实现并发控制锁的一个关键类。我们可以使用synchronized关键字来实现线程间的同步互斥<!-- more -->，也可以通过ReentrantLock来实现。

## ReentrantLock与synchronized区别
&nbsp;&nbsp;&nbsp;&nbsp;首先我们想一下，synchronized的实现，有两种方式，一种是修饰方法的关键字，一种是选择一个监控对象以代码块的形式来实现同步互斥，在遇到异常时自动退出同步互斥状态。这种方式比较简单，我们不需要去操心锁会不会得不到释放，只要代码能够正常执行完，锁就会得到释放。但是我们如果想以更灵活的一些方式去实现这个流程，不好意思，synchronized比较固定，你没法控制。

&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock就很灵活了。他有两个关键方法，lock、unlock。什么情况获得锁，什么情况释放锁，都是可以灵活控制的。而除了这点的灵活性以外，后面部分还会介绍更加灵活的内容。我们首先需要知道的是，ReentrantLock相比synchronized的灵活性强很多。但是灵活性势必带来复杂性，如果不小心就很容易出错。

## ReentrantLock同步示例
功能测试类如下：
```bash
public class AService {
    private Lock lock = new ReentrantLock();
    public void a() {
        try {
            lock.lock();
            // 打印时间戳
            Thread.sleep(1000);
            // 一些操作
        } catch {
            e.printSatckTrack();
        } finally {
            lock.unlock();
        }
    }
 
    public void b() {
        try {
            lock.lock();
            // 打印时间戳
            Thread.sleep(1000);
            // 一些操作
        } catch {
            e.printSatckTrack();
        } finally {
            lock.unlock();
        }
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;现在有两个线程接收一个对象作为参数，内部run分别调用这个对象的a和b方法。打印结果回事顺序打印两个时间戳，时间相差1秒。大家可以实际实验一下，结论就是lock可以实现同步的互斥。

---
# Condition实现等待通知
&nbsp;&nbsp;&nbsp;&nbsp;我们在之前synchronized的时候，使用wait、notify来实现等待和通知，那么在用Lock的时候，肯定他也得有这样对应的实现啊。Condition就是他这样的实现。而且要比synchronized里面的形式更加灵活。

我们先来看个例子：
```bash
public class AService {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    public void a() {
        try {
            lock.lock();
            // 打印时间戳
            condition.await();
            Thread.sleep(1000);
            // 一些操作
        } catch {
            e.printSatckTrack();
        } finally {
            lock.unlock();
        }
    }
 
    public void signal() {
        try {
            lock.lock();
            condition.signal();
        } finally {
            lock.unlock();
        }
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这里展示了Condition的两个重要方法，以及怎么生成Condition的方法。由于所有类都是继承自Object的，所以wait与notify已经都被占了，所以Condition使用的是await与signal，当然，他也会有个signalAll方法与notifyAll对应。另外Condition和synchronized形式的wait、notify一样，也必须在同步状态下，否则也是会报错的。

&nbsp;&nbsp;&nbsp;&nbsp;我们看到，Condition的对象是通过调用lock.newCondition获取的，所以实际上是可以获取多个Condition对象的，所以这就是Lock比synchronized灵活的另外一个地方了，它可以有多个不同的Condition监视来做wait、notify。而**每组await、signal只影响相同的Condition的，不同的Condition的await、signal互不相干**。

---
# 公平锁与非公平锁
&nbsp;&nbsp;&nbsp;&nbsp;Lock的另外一个灵活的地方是可以选择这个锁是个公平锁还是个非公平锁。公平锁意思是线程获取锁的顺序是按照线程启动顺序来分配的，类似于FIFO先进先出。而非公平锁线程间是通过抢占机制，随机获得锁的，所以在非公平锁的情况下有些线程可能永远拿不到锁。

&nbsp;&nbsp;&nbsp;&nbsp;是否是公平锁，这个的设置很简单，就是在创建ReentrantLock的时候，传入一个布尔型的参数即可，这里就不列出代码了,大家可以自己实验一下。在默认的情况下，ReentrantLock类使用的是非公平锁。

---
# ReentrantReadWriteLock类
&nbsp;&nbsp;&nbsp;&nbsp;Lock里面还可以进一步提高效率。synchronized和ReentrantLock都会独占锁，也就是同一时间只有一个线程可以监控lock，操作某一关键对象。但是这种情况性能很差，等于并发变成顺序执行了。

&nbsp;&nbsp;&nbsp;&nbsp;而我们想一个场景，当一个关键数据读多写少的时候（事实上我们多数的需求也都是这样，读多写少），我们让所有的数据读取都顺序执行，势必降低了运行效率，如果在数值不变的情况下，完全大家可以一起去并发访问这个数据，而没有必要顺序访问。这就是ReentrantReadWriteLock的作用。

&nbsp;&nbsp;&nbsp;&nbsp;实际上这个类就类似于使用了两个锁，一个读锁一个写锁。我们想想一下如果要让读写锁分开的话，什么情况是允许的呢？

&nbsp;&nbsp;&nbsp;&nbsp;读，大家是可以共享锁的，不需要阻塞互斥，而写与写的操作是必须互斥的，不然可能导致数据错乱。那么读写呢，肯定也需要互斥，不然可能读到脏数据。所以只要有写的操作，就必须其他都不能获取锁，而没有写的时候，所有的读都可以获取到锁。

&nbsp;&nbsp;&nbsp;&nbsp;道理很简单，总结起来就是读读共享、写写互斥、读写互斥、写读互斥。只要有写就互斥。

&nbsp;&nbsp;&nbsp;&nbsp;关键方法就是lock.readLock().lock()与lock.readLock().unlock()，lock.writeLock().lock()与lock.writeLock().unlock()。使用的时候必须成对出现。

&nbsp;&nbsp;&nbsp;&nbsp;到此，所有有关Lock的内容就都说完了，其实Lock并没有什么难的，api是有点多，关键还是实战经验了。

---
转载自：[赵伊凡's Blog](http://irfen.me/java-multi-thread-4/)