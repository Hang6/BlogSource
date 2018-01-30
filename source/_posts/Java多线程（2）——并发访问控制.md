---
title: Java多线程（2）——并发访问控制
date: 2017-09-18 09:58:09
tags:
- Java
- 多线程
- 并发
- 性能
comments: true
categories: 转载
---
&nbsp;&nbsp;&nbsp;&nbsp;这章主要介绍一下synchronized关键字相关的用法，顺带也介绍一下volatile关键字。这两个关键字在java的并发访问控制中都很重要。

---
# synchronized使用范围及加锁规则
&nbsp;&nbsp;&nbsp;&nbsp;synchronized这个关键字可以有很多用法，每种用法所加的锁都有不同的锁范围，下面一一介绍。
<!-- more -->
&nbsp;&nbsp;&nbsp;&nbsp;a、加在实例方法上作为关键字
&nbsp;&nbsp;&nbsp;&nbsp;b、加在静态方法上作为关键字
&nbsp;&nbsp;&nbsp;&nbsp;c、同步语句块，这块分两种，一种是使用对象，一种是使用class

## 1、synchronized加在实例方法上
&nbsp;&nbsp;&nbsp;&nbsp;举个简单的例子吧。
```bash
public class A {
    public synchronized void xxx(){
        // do something
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这样就是把关键字加在实例方法上，为什么要特别强调实例呢，因为我们对应的还有静态方法。实例方法需要new出对象来调用，而静态方法可以直接类名调用（这块就当废话）。

&nbsp;&nbsp;&nbsp;&nbsp;由于调用这个xxx方法需要实例化出来一个对象，所以，多个线程调用这同一个对象的xxx方法，他们的调用就会是同步的了。下面看段代码实例。
```bash
public class AThread extends Thread {
    private A a;
    public AThread(A a) {
        this.a = a;
    }
    @Override
    public void run() {
        a.xxx();
    }
}
```
```bash
public static void main(String[] args) {
    A a = new A();
    AThread t1 = new AThread(a);
    AThread t2 = new AThread(a);
    t1.start();
    t2.start();
}
```
&nbsp;&nbsp;&nbsp;&nbsp;正常情况，这两个线程应该是并发异步执行的（即t2的run的内容不需要等t1结束在运行），但是由于线程调用了A的xxx方法，这个方法被synchronized关键字修饰了，这时候这个xxx方法变成了同步方法，所以t2的run在调用a的xxx的时候，会被阻塞，知道t1里面的内容执行完。

&nbsp;&nbsp;&nbsp;&nbsp;**另外提两句：如果t1和t2两个线程锁传入的对象，是两个不同的对象的话（例如new出两个A，a1、a2）则不会产生这个阻塞；如果synchronized这个关键字同时修饰了A类的两个实例方法xxx与yyy，t1里面调用的还是xxx，而t2里面调用的是yyy，那么仍然是同步的，和当前的运行结果没有什么差异。**

&nbsp;&nbsp;&nbsp;&nbsp;所以结论就是：**synchronized关键字修饰在实例方法上，会对实例出来的对象加锁。**

## 2、synchronized关键字加在静态方法上
&nbsp;&nbsp;&nbsp;&nbsp;这里只举例一下修饰的例子，就不详细介绍调用的示例了。
```bash
public class A {
    public synchronized static void xxx(){
        // do something
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这块其实很简单，就是把synchronized加在里一个静态方法上面。这种情况下，就是对这个A类的所有静态方法加锁了。当然同时也锁了下面要讲的一个synchronized同步语句块的一个情况的锁。

&nbsp;&nbsp;&nbsp;&nbsp;**这里还要额外补充一点，实例方法的锁，会锁同一实例的所有加了synchronized关键字的实例方法；静态方法的锁，会锁同一类（class）所有加了synchronized关键字的静态方法。**

&nbsp;&nbsp;&nbsp;&nbsp;对于1和2两部分，下面在举个例子，某个类X有这四个方法。
```bash
synchronized void a();
synchronized void b();
synchronized static void sa();
synchronized static void sb();
```
&nbsp;&nbsp;&nbsp;&nbsp;现在有X类的两个实例x、y，对于下面的四种情况，我们分别说一下结果。

&nbsp;&nbsp;&nbsp;&nbsp;1）x.a与x.b，这种情况就是我们1里面说过的，由于是同一个对象，所以是同步访问。
&nbsp;&nbsp;&nbsp;&nbsp;2）x.a与y.a，由于实例方法的锁是针对对象的，所以这里两个线程的访问会是异步非阻塞的。
&nbsp;&nbsp;&nbsp;&nbsp;3）x.sa与y.sb（其实应该这么写X.sa与X.sb），这里由于是修饰的静态方法，所以这个锁是针对class的，所以他们会阻塞，是同步的。
&nbsp;&nbsp;&nbsp;&nbsp;4）x.a与X.sa，这里大家可以去实验一下，会是异步的。我们可以这么理解，对象锁和class的类锁是互不相干的，他们只管自己的事。

## 3、synchronized同步语句块
&nbsp;&nbsp;&nbsp;&nbsp;接下来我们来介绍同步语句块，为什么可以修饰在方法的关键字上之后，还要同步语句块呢？首先synchronized修饰在方法上其实易用性很强，我们不用管太多东西，只要方法结束或者方法中间抛出异常，这个同步锁就会解开结束。缺点是什么呢，不灵活、效率低。由于这个关键字加在了方法上，所以锁的是整个方法。加入一个方法a需要运行2s，那么同时过来3个线程，就是6s。

&nbsp;&nbsp;&nbsp;&nbsp;假如有个投票的方法，这个方法会加票、写库、然后各种记录操作、扣钱（假设投票需要虚拟币）、通知前端、发消息什么的。一堆操作肯定很耗时，但是为了保持我们的投票数据准确不能出现脏读的情况，所以我们还必须加锁。假设这个投票方法要运行2s，那么在投票的快要结束的时间，同时1000个人来投票就要2000s时间来处理啊，半个多小时。

&nbsp;&nbsp;&nbsp;&nbsp;实际上我们真正需要加锁的地方在哪，并不是上面提到的所有的情况都要锁起来，我们只需要在增加投票数那一块锁起来，后面的一些无关的操作并不一定需要是同步的。所以synchronized在方法上修饰就没那么灵活了。

&nbsp;&nbsp;&nbsp;&nbsp;同步语句块要解决的就是这么个情况，可以在方法的中间需要加锁的地方加锁，只锁那一块。下面举个例子。
```bash
public class A {
    public void xxx(){
        synchronized (this) {
            // 加票
        }
        // 记录明细、扣钱...
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;上面的代码就只对加票这块做了同步处理，可能加票这部操作只需要1ms，就要快了很多。

&nbsp;&nbsp;&nbsp;&nbsp;下面在说一下同步语句块的两种情况，一种是以对象为锁，一种是加类锁。上面的例子实际上是对象锁，this也是个对象嘛。当然这里也可以写成synchronized(A.class)，这样就是在A这个类上加锁。对于我们这次的业务需求来说都一样没啥差别（实际上应该加类锁，但是我们这块一般都会做成单例去处理这样的业务，就问题不大了）。

&nbsp;&nbsp;&nbsp;&nbsp;其实同步语句块与上面的synchronized修饰于方法上面还是有互斥的，对应的情况就是如果同步语句块的参数是this的话，就是代表这个实例对象，所以会和1中所讲情况产生同步；如果是class的话，代表类，会与2中所讲内容产生同步。

&nbsp;&nbsp;&nbsp;&nbsp;当然同步语句块的参数还可以使用其他对象，一般为了处理像是投票那种比较独立的需求，我们可以这样加锁。
```bash
public class A {
    private Object lock = new Object();
    public void xxx(){
        synchronized (lock) {
            // 加票
        }
        // 记录明细、扣钱...
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;单独声明一个对象作为锁，这样锁的是这个实例对象，当在别的地方需要用到这个锁的时候也在这个实例对象上加锁就行，不会和A这个类的对象锁和类锁冲突。

&nbsp;&nbsp;&nbsp;&nbsp;最后说一点，就是这个参数还可以是String字符串，但是一般尽量不要使用，由于String字符串在Java中存在常量池的问题，所以有时候虽然是两个变量，但是只要内容一样就会产生同步锁。

## 4、锁重入、继承问题
&nbsp;&nbsp;&nbsp;&nbsp;最后再说一个锁重入与继承所产生的问题。锁重入，就是synchronized代码块中又有一个synchronized代码块，或者同步方法中调用了同步方法。

&nbsp;&nbsp;&nbsp;&nbsp;同步代码块能否进锁就还是看能不能获取锁，有没有相同的锁内容正在执行。

&nbsp;&nbsp;&nbsp;&nbsp;同步方法调用同步方法这里，只要是自己对象的锁，那么可以无限重新进锁。举上面2对比的那个例子，a里面如果调用了b，那么b中的方法也是可以执行的，并不会因为a方法持有了锁，而到里面的b会出现问题。但是直接调用b方法肯定还是会产生锁的。

&nbsp;&nbsp;&nbsp;&nbsp;继承这块就两个重点，一个是继承就当是子类全部继承就好了，没有父子类关系。另外一个就是重写的时候，synchronized关键字也需要重新声明，否则重写方法不加synchronized关键字这个方法就不会是同步方法了。

## 5、死锁
&nbsp;&nbsp;&nbsp;&nbsp;多线程的锁，很容易产生死锁问题，下面举个例子。
```bash
public synchronized static void a() {
    // ...
    b();
    // ...
}
 
public synchronized static void b() {
    // ...
    a();
    // ...
}
```
&nbsp;&nbsp;&nbsp;&nbsp;这里是个简单明了的例子，两个线程同时进入a、b方法（他们肯定不能是一个类的了，不然b都进不去），这时候，a在等待进入b的锁，b在等待进入a的锁，就会产生互相等待，也就是死锁了。

&nbsp;&nbsp;&nbsp;&nbsp;分析一个Java进程有没有死锁，可以通过运行*jstack -l* 进程ID来发现是否存在死锁。

---
# volatile关键字
&nbsp;&nbsp;&nbsp;&nbsp;其实对于同步来说，见的最多的双重校验单例的实现。里面其实也有用到了volatile关键字。这个关键字一般还是挺少用的，他有两个作用，一个是可见性，一个是禁止指令重排序。

&nbsp;&nbsp;&nbsp;&nbsp;可见性这个问题，在一般情况下不容易见到，但是当运行server版Java进行的话，就会出现。当然也可以通过jvm增加-server参数来实现。线程内都会保存一个变量的内存副本，这个内存副本只会在初始化的时候读取，之后就是在线程内做了修改，回去写入和更新。但是如果外部去修改了这个变量，那么线程内的副本是不会主动更新的，这就是可见性的问题。所以如果给变量增加了volatile的关键字，那么就可以保证这个变量每次都去主内存中读写变量，不管内存副本了。

&nbsp;&nbsp;&nbsp;&nbsp;指令重排序，这个大家可能会比较陌生，由于jvm有自己对代码的优化，比如一段代码的4行内容，他们互不影响，那么在jvm实际执行的时候有可能并不是按照顺序执行的，可能是1、3、2、4这样的顺序执行，这就是指令重排序。当然这和我们这次讲的锁没有关系，只是提一下，volatile还有禁止指令重排序的作用。

---
# 小结
&nbsp;&nbsp;&nbsp;&nbsp;其实多线程的问题有点类似于我们一开始学习数据库，脏读啊，不可重复读什么的，都是并发引起的。当然这里就涉及到锁了。本次介绍了synchronized关键字的处理锁和同步的问题，其实Java中还有更灵活的方式也就是lock来处理锁和同步的问题，这个我们之后会讲到。

---
转载自：[赵伊凡's Blog](http://irfen.me/java-multi-thread-2/)