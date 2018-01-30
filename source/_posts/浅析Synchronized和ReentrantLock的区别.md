---
title: 浅析Synchronized和ReentrantLock的区别
date: 2017-12-08 10:30:37
tags:
- Synchronized
- ReentrantLock
comments: true
categories: 日志
---
# Synchronized
&nbsp;&nbsp;&nbsp;&nbsp;当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码，它是在 软件层面依赖JVM实现同步。synchronized方法或语句的使用提供了对与每个对象相关的隐式监视器锁的访问，但却强制所有锁获取和释放均要出现在一个块结构<!-- more -->中：**当获取了多个锁时，它们必须以相反的顺序释放，且必须在与所有锁被获取时相同的词法范围内释放所有锁**。<br>

---

# ReentrantLock
&nbsp;&nbsp;&nbsp;&nbsp;Lock接口实现提供了比使用synchronized方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的Condition对象。在硬件层面依赖特殊的CPU指令实现同步更加灵活。<br>
&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock是一个可重入的互斥锁Lock，它具有与使用synchronized方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。 

&nbsp;&nbsp;&nbsp;&nbsp;ReentrantLock相对于synchronized对了三个高级功能：<br>
1. 等待可中断
2. 公平锁
3. 绑定多个Condition

---

# Synchronized与ReentrantLock的用法和区别
1. synchronized是托管给**JVM**执行的，而Lock是Java写的控制锁的代码。 
2. synchronized原始采用的是CPU**悲观锁机制**，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。
3. ReentrantLock用的是**乐观锁**方式。每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。
4. ReentrantLock必须在finally中释放锁，否则后果很严重，编码角度来说使用synchronized更加简单，不容易遗漏或者出错。 
5. ReentrantLock提供了可轮询的锁请求，他可以尝试的去取得锁，如果取得成功则继续处理，取得不成功，可以等下次运行的时候处理，所以不容易产生死锁，而synchronized则一旦进入锁请求要么成功，要么一直阻塞，所以更容易产生死锁。 
6. synchronized的话，锁的范围是整个方法或synchronized块部分；而Lock因为是方法调用，可以跨方法，灵活性更大


&nbsp;&nbsp;&nbsp;&nbsp;**注意**:Synchronized是Java的关键字，因此是Java的内置特性，是基于**JVM层面**实现的；而Lock是一个Java的接口，所以ReentrantLock是基于**JDK层面**实现的。


&nbsp;&nbsp;&nbsp;&nbsp;一般情况下都是用synchronized原语实现同步，除非下列情况使用ReentrantLock:
- 某个线程在等待一个锁的控制权的这段时间需要中断 
- 需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程 
- 具有公平锁功能，每个到来的线程都将排队等候

<br>
<br>