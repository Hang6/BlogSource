---
title: Java多线程（1）——基础
date: 2017-09-18 09:15:51
tags:
- Java
- 多线程
- 并发
- 性能
comments: true
categories: 转载
---
# 进程与线程
&nbsp;&nbsp;&nbsp;&nbsp;**进程**是什么，想必学计算机的同学都不会陌生，打开windows任务管理器，或者linux服务器上top命令锁展示的结果，就是一个个的进程。

&nbsp;&nbsp;&nbsp;&nbsp;*进程*（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在早期面向进程设计的计算机结构中，进程是程序的基本执行实体；在当代面向线程设计的计算机结构中<!-- more -->，进程是线程的容器。程序是指令、数据及其组织形式的描述，进程是程序的实体。——引自百度百科

&nbsp;&nbsp;&nbsp;&nbsp;但是一个应用只有一个进程吗，NO，一个应用可能会开除多个子进程来进行需要的工作。举个例子，php解析的服务进程，可以配置要使用多少个子进程进行工作，所以在top命令中，会看到多个同名进程，而他们的创建者都是php主进程创建的。

&nbsp;&nbsp;&nbsp;&nbsp;**线程**大家肯定也听过，一般一个进程下会有多个线程。

&nbsp;&nbsp;&nbsp;&nbsp;*线程*，有时被称为轻量级进程，是程序执行流的最小单元。在单个程序中同时运行多个线程完成不同的工作，称为多线程。——引自百度百科

&nbsp;&nbsp;&nbsp;&nbsp;另外，一个进程至少会有一个线程，如果只有一个线程，那就是程序本身了。一般程序有多个线程的话都会有一个主线程去执行入口程序，之后再发起其他线程去进行其他的工作。

&nbsp;&nbsp;&nbsp;&nbsp;看起来还是不太清晰的话，我来举个例子解释一下。比如一个应用，需要干两件事情，A是发送一个请求等待响应之后打印出来（这个请求加上响应的时间可能是2s），B也是发送一个请求等待响应之后打印出来（这个时间是3s）。如果没有线程就需要顺序执行了，这个总的执行时间我们可以预估到是5s。如果使用了多线程，线程1发起请求A，这时候再起一个线程2发起请求B，线程1等待2s收到响应打印，线程2等待3s收到响应打印。总的程序执行时间就是3s了。

&nbsp;&nbsp;&nbsp;&nbsp;其实多线程，是利用CPU空闲去干多件事。就比如上面的例子，A在等待的时候，CPU是闲着的，这时候这2s的时间内就可以去干别的事情。其实多线程只是看起来是同时执行了多个任务，实际上只是利用的空闲时间去相互插空而已，只是我们感觉不到。

---
# Java中怎么实现多线程
&nbsp;&nbsp;&nbsp;&nbsp;其实这个问题在面试的时候也很容易被问到。有两种方法，一种是继承Thread类，一种是实现Runnable接口。其实Thread类也是实现了Runnable接口的，看一眼源码就会知道了。

&nbsp;&nbsp;&nbsp;&nbsp;这里实现起来其实很简单，继承Thread类就去重写run方法，实现Runnable就去实现run方法，然后调用start方法执行线程。这里要说一点的是，多线程具有异步性，就是在代码中写的顺序，不一定代表执行结果的顺序。举个例子，多个线程打印数字。
```bash
public class PrintText extends Thread {
    private int num = 0;
    public PrintText(int num) {
        this.num = num;
    }
 
    @Override
    public void run() {
        System.out.println(num);
    }
}
```
下面是程序主入口
```bash
public class Test {
    public static void main(String[] args) {
        PrintText1 pt1 = new PrintText(1);
        PrintText1 pt2 = new PrintText(2);
        PrintText1 pt3 = new PrintText(3);
        PrintText1 pt4 = new PrintText(4);
        pt1.start();
        pt2.start();
        pt3.start();
        pt4.start();
    }
}
```
这里程序的执行顺序是1、2、3、4，但是显示的结果确实不确定的，因为他们谁先占用到了CPU的资源是不确定的，所以千万不用想当然的把代码的顺序作为了程序打印结果的依据。

---
# 并发访问数据
&nbsp;&nbsp;&nbsp;&nbsp;讲到多线程，肯定就要说说并发访问数据的问题了。多个线程同时去修改一个数据的时候，很容易产生问题。其实这个东西我们接触过，就是在学习数据库的时候，有一个读脏数据、不可重复读那块。

&nbsp;&nbsp;&nbsp;&nbsp;我们想想一个场景，两个线程都要多一个数字做加一操作，比如这时候数字是5，那么两个线程同时读到目前是5（其实肯定是有先后顺序的，只不过这个时间差很短，短到两个线程都还没有完成加一并写入的操作执行完），那么每个线程各自加一得到6并写入变量，那么执行完的结果就是6，但是实际上两个线程都进行了加一操作，正确的结果应该是7才对。

&nbsp;&nbsp;&nbsp;&nbsp;如何解决这个问题呢？两种办法synchronized关键字和Lock，这个我们以后在说。

&nbsp;&nbsp;&nbsp;&nbsp;其实synchronized这个关键字就在我们身边，只不过我们从没注意过，我们总是在用，甚至一开始就学的System.out.println()这个打印语句，其中的println就是使用了这个关键字的一个线程安全的方法。

---
# 几个基础方法
&nbsp;&nbsp;&nbsp;&nbsp;下面我们来学几个多线程的基础方法，也就是Thread类给我们提供的一些方法。

## currentThread()方法
&nbsp;&nbsp;&nbsp;&nbsp;这个方法会返回当前正在执行的线程的一些信息。其实也都很容易理解。
```bash
Thread.currentThread().getName(); // 返回当前线程的名称
Thread.currentThread().getId(); // 返回当前线程的唯一标识
```
&nbsp;&nbsp;&nbsp;&nbsp;这里需要注意的是，currentThread返回的是当前正在执行的线程的信息，而main方法的入口本身也是一个主线程。所以假设我们在上面打印数字的例子中，实现Thread类的构造方法中获取当前的线程名字，会是main，而在run方法中获取线程名字才是当前这个类的线程，一般默认是Thread-0（这里如果不给线程设置名称的话，他默认按照线程创建顺序把线程命名为Thread-n这样的形式）。

## isAlive()方法
&nbsp;&nbsp;&nbsp;&nbsp;其实看英文也能差不多猜到，就是获取当前线程是否在活动状态。如果直接在run()方法中判断，一定会是true。说白了，就是run方法只要还在执行过程中，他就会是true。

## sleep()方法
&nbsp;&nbsp;&nbsp;&nbsp;这个大家应该很熟悉吧，就是让线程等待多久的方法，默认单位是毫秒。一般我们都是用的Thread.sleep(1000)这样，其实他指的是currentThread的线程。

## 停止线程
&nbsp;&nbsp;&nbsp;&nbsp;在Java中有3中方法停止正在运行的线程。
&nbsp;&nbsp;&nbsp;&nbsp;a、用退出标志位，使线程正常退出，也就是当run方法执行完后线程停止。
&nbsp;&nbsp;&nbsp;&nbsp;b、stop方法强制停止。（这个方法已经弃用了，虽然很简单，但是简单必有坑）
&nbsp;&nbsp;&nbsp;&nbsp;c、使用interrupt方法中断线程。

&nbsp;&nbsp;&nbsp;&nbsp;你当然可以用退出标志位来处理了，一般也就是各种if嵌套什么的，逻辑会比较混乱；stop就不用说了，官方都弃用了；最后也就是推荐的使用interrupt中断了。

&nbsp;&nbsp;&nbsp;&nbsp;使用interrupt中断，可没有for循环中的break那么简单，调用thread的interrupt方法，并不会立刻终端线程，而只是标记这个线程要中断，能不能中断还是要看线程当前的状态的。你觉得这样不好？stop就是因为可以立刻中断，让线程没有好好办法善后，所以才推荐用interrupt的呀~

&nbsp;&nbsp;&nbsp;&nbsp;配合interrupt()方法的还有几个方法，他们是interrupted，isInterrupted，看起来好像是一个意思是吧。其实不一样，前者意思是当前线程是否已经中断，后者是线程是否已经中断。看一下程序的声明，会发现，前者是**静态（static）**方法，那肯定就是Thread可以直接调用了，所以默认也就是当前线程了；后者是类的普通方法，那么肯定要在某个类的实例化之后才能调用，所以就是那个线程的判断了。

&nbsp;&nbsp;&nbsp;&nbsp;这里还要有一点需要注意，就是这个interrupted方法，如果在线程已经是中断的状态下的话连续调用两次，第一次会是true，第二次就回事false了。为什么呢？官方给出的解释是这个方法具有清楚状态的功能。所以第一次返回true，并且清除了状态，所以第二次就返回false了，这点要特别注意。

&nbsp;&nbsp;&nbsp;&nbsp;所以我们我们怎么通过interrupt去结束线程呢？当然可以在run方法里面通过interrupted方法的判断来做if，但是这样真的会有些混乱，所以推荐的是**异常法**。先举例一个if判断的坏处。
```bash
public class PrintText extends Thread {
 
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            if (this.interrupted()) {
                System.out.println("interrupted");
                break;
            }
            System.out.println(i);
        }
        System.out.println("finish");
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;上面的代码，如果程序出现中断了，那么会跳出for循环，但是如果想不打印最后的finish字符内容的话，怎么处理呢？在做一个判断吗？
&nbsp;&nbsp;&nbsp;&nbsp;如果这里我们**把break改为抛出一个异常**，然后在整个代码块上做个try...catch，这样，就可以不打印finish信息了。如果想打印的话，可以让try...catch不包含这个打印语句，这样就可以打印出来了，所以异常法更灵活一些吧。当然你这里也可以使用return不打印finish，但是也不够灵活。

## 睡眠中停止
&nbsp;&nbsp;&nbsp;&nbsp;如果线程正处于sleep状态的时候，调用线程的interrupt，sleep会自动抛出InterruptedException，这样同样可以达到我们的目的。但是这种情况需要注意的是，抛出异常的同时也会把中断状态清除，这点要格外留意。
&nbsp;&nbsp;&nbsp;&nbsp;另外这里我给大家总结一点，就是sleep和interrupt两个方法的执行前后所产生的不同结果。上面说的是在sleep过程中调用interrupt，会抛出异常中断；还有一种情况，就是先interrupt，然后线程仍会执行，这时候调用sleep，立即也会产生中断异常。

## 暂停与恢复
&nbsp;&nbsp;&nbsp;&nbsp;暂停与恢复应该很好理解，就是让线程的运行暂停或者恢复。对应的方法分别为suspend与resume，但是需要说的也是一样，这两个方法也已经被官方弃用了，弃用就说明一定有他不好的地方。使用起来很简单，这里就不做介绍了，大家可以自己试试。
&nbsp;&nbsp;&nbsp;&nbsp;这里说一下他们的缺点，比如有一个线程锁定了一个对象，然后暂停了，会造成对这个对象的长时间锁定，可能会堵死程序。另外也会造成数据的不一致，比如一个线程需要对这个数据做两次加一，结果只做了一次的时候暂停了，别的程序拿到的不是一个正确的结果，就出错了。

## yield方法
&nbsp;&nbsp;&nbsp;&nbsp;这个方法意思就是主动放弃当前CPU资源，让其他线程使用。但是这里的其他线程也包括他自己的，所以并不是一个精准的功能。但是会造成CPU资源切换所导致的时间花销。

## 线程的优先级
&nbsp;&nbsp;&nbsp;&nbsp;线程可以设置优先级，有个方法是setPriority(int newPriority)。

&nbsp;&nbsp;&nbsp;&nbsp;线程优先级只有10级就是1到10，如果超出范围会抛出异常的，这点大家可以看一下源码了解。默认的优先级都是5，但是这里需要说明一点的是，优先级并不会绝对代表程序的执行顺序，只是个建议，实际上可能会和我们设置的不一样。但是跨度很大的比如1和10的话，10比1先执行的可能性就大很多。

&nbsp;&nbsp;&nbsp;&nbsp;另外线程的优先级具有继承性，比如A线程优先级是10，那么由A线程启动的B线程，优先级也会是10。

## 守护线程
&nbsp;&nbsp;&nbsp;&nbsp;守护线程就是一种特殊的线程，他依托于某一线程A存在，当A线程结束的时候，守护线程也会自动结束。
&nbsp;&nbsp;&nbsp;&nbsp;具体用法就也很简单，就是通过调用thread.setDaemon(true)来设置的。到此，所有有关多线程的基础内容也就介绍完了，希望大家能够掌握。

---
转载自：[赵伊凡's Blog](http://irfen.me/java-multi-thread-1/)