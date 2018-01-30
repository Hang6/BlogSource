---
title: Spring事务处理
date: 2017-12-08 16:33:36
tags:
- Spring
- 事务
comments: true
categories: 原创
---
# Spring声明式事务处理
&nbsp;&nbsp;&nbsp;&nbsp;在使用声明式事务处理的时候，通常是结合IoC容器和Spring已有的TransactionProxyFactoryBean对事务管理进行配置，比如：可以在TransactionProxyFactoryBean中为事务方法配置传播行为、并发事务隔离级别等事务处理属性，从而对声明式事务处理提供指导。

&nbsp;&nbsp;&nbsp;&nbsp;声明式事务处理大致可以分为以下几个部分：<!-- more -->
- **读取和处理在IoC容器中配置的事务处理属性，并转化为Spring事务处理需要的内部数据结构**。这里涉及的类是TransactionAttributeSourceAdvisor，它是一个AOP通知器，通过它来完成对事务处理属性值的处理。
- **Spring事务处理模块实现统一的事务处理过程**。这个过程包含处理事务配置属性，以及与线程绑定完成事务处理的过程，Spring通过TransactionInfo和TransactionStatus这两个数据对象，在事务处理过程中记录和传递相关执行场景
- **底层的事务处理实现**。底层的事务操作，Spring委托给具体的事务处理器完成。这些处理器就是在IoC容器中配置声明式事务处理时，配置的PlatformTransactionManager的具体实现，比如DataSourceTransactionManager等。
<br>

## 事务处理拦截器的配置
&nbsp;&nbsp;&nbsp;&nbsp;在使用声明式事务处理的时候，需要在IoC容器中配置TransactionProxyFactoryBean。在IoC容器进行注入的时候，会**创建TransactionInterceptor**对象，这个对象会创建一个TransactionAttributePointcut，为**读取TransactionAttribute**做准备。

&nbsp;&nbsp;&nbsp;&nbsp;在容器初始化的过程中，实现了**afterPropertiesSet()方法**，这个方法**完成了Spring事务处理的AOP配置**。在这里，会为ProxyFactory设计通知、目标对象，并最终返回Proxy代理对象。在代理对象建立起来之后，当调用代理方法的时候，会调用相应的TransactionInterceptor拦截器，根据TransactionAttribute配置的事务属性进行配置，从而为事务处理做好准备。

&nbsp;&nbsp;&nbsp;&nbsp;Spring为声明式事务处理的实现做了些准备工作：包括**为AOP配置基础设施**，这些基础设施包括设置**拦截器TransactionInterceptor、通知器TransactionAttributeSourceAdviso**r。同时，在TransactionProxyFactoryBean的实现中，还可以看到**注入**进来的**PlatformTransactionManager**的**事务处理属性TransactionAttribute**等。
<br>

## 事务处理配置的读入
&nbsp;&nbsp;&nbsp;&nbsp;在AOP配置完成的基础上，以TransactionAttributeSourceAdvisor的实现为入口，来了解具体的事务属性配置式如何读入的。

&nbsp;&nbsp;&nbsp;&nbsp;在声明式事务处理中，通过对目标对象的方法调用进行拦截实现，这个拦截通过AOP发挥作用。对于拦截的启动，首先需要对方法调用**是否拦截进行判断**，判断的**依据**就是那些在TransactionProxyFactoryBean中**为目标对象设置的事务属性**。在对事务属性TransactionAttributes进行设置时，会从事务属性配置中读取事务**方法名和配置属性**，然后将其作为一个**键值对加入到一个nameMap中**。

&nbsp;&nbsp;&nbsp;&nbsp;具体的**实现过程**如下：首先，以调用方法名为索引在nameMap中查找相应的事务处理属性，如果能够找到，那么说明该方法和事务方法是直接对应的；如果找不到，那么就会遍历整个nameMap进行命名模式（不需要完整的方法名，可以使用通配符*）上的匹配。匹配上之后，从nameMap中取出事务处理属性，触发事务处理拦截器的拦截。

&nbsp;&nbsp;&nbsp;&nbsp;通过以上过程便得到了与目标对象调用方法相关的**TransactionAttribute对象**，这个对象**完成了对事物处理配置的封装**。它包含了对事务方法的配置信息，为TransactionInterceptor做好了对调用的目标方法添加事务处理的准备。
<br>

## 事务处理拦截器的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;在完成了以上的准备工作之后，经过TransactionProxyFactoryBean的AOP包装，此时如果对目标对象进行方法调用，起作用的实际上是一个Proxy代理对象。

&nbsp;&nbsp;&nbsp;&nbsp;当拦截器TransactionInterceptor拦截到Proxy代理对象对目标方法的调用的时候，会调用拦截器中的**invoke()方法进行回调**，去**获取**前面准备好的**事务处理配置**，之后会取得配置的**PlatformTransactionManager**，由这个事务处理器来实现事务的创建、提交、回滚操作。

---
<br>

# Spring事务处理的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;在声明式事务中，**Spring框架**完成了**对事务处理的统一管理**，以及**对并发事务和事务属性的处理**。采用的是一个比较复杂的过程，在这个过程中封装了在事务处理中对事务的**创建、提交和回滚**等核心操作。
<br>

## 事务的创建
&nbsp;&nbsp;&nbsp;&nbsp;作为声明式事务处理的起始点，需要注意拦截器的invoke()回调中使用的createTransactionIfNecessary方法。

&nbsp;&nbsp;&nbsp;&nbsp;在这个方法的调用中，会向AbstractTransactionManager**执行getTransaction()方法**，这个获取Transaction事务对象的过程，会根据事务的情况做出不同处理，然后**创建一个TransactionStatus，并把这个TransactionStatus设置到对应的TransactionInfo中去，同时将TransactionInfo和当前的线程绑定**，从而完成事务创建过程。创建的具体过程由PlatformTransactionManager完成。


&nbsp;&nbsp;&nbsp;&nbsp;事务创建的结果是**生成一个TransactionStatus对象**，通过这个对象来**保存事务处理需要的基本信息**，这个对象与前面提到过的TransactionInfo对象联系在一起。**TransactionStatus是TransactionInfo的一个属性，然后会把TransactionInfo保存在ThreadLocal对象里**，这样当前线程可以**通过ThreadLocal对象取得TransactionInfo，以及这个事务对应的TransactionStatus对象**，从而把事务的处理信息与调用事务方法的当前线程绑定起来。
<br>

## 事务的提交和回滚
&nbsp;&nbsp;&nbsp;&nbsp;事务提交的入口同样是在TransactionInterceptor中的invoke()中实现，它通过直接调用事务处理器来完成事务提交。如果事务处理过程中发生了异常，那么会调用回滚方法。

---
<br>

# Spring事务处理器的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;Spring事务处理的主要过程是分两个部分完成的，通用的事务处理框架是在AbstractPlatformManager中完成，而Spring的事务接口与数据源实现的接口，多半是由具体的事务管理器来完成，它们都是作为AbstractPlatformManager的子类来使用的。
<br>

## DataSourceTransactionManager的实现
&nbsp;&nbsp;&nbsp;&nbsp;在DataSourceTransactionManager中，在事务开始的时候，对调用doBegin()方法，首先会得到相对应的Connection，然后会根据事务设置的需要，对Connection的相关属性进行配置，比如将Connection的autoCommit功能关闭，并对像TimeoutInSeconds这样的事务处理参数进行配置，最后通过TransactionSynchronizationManager来对资源进行绑定。

---
参考：《Spring技术内幕》——计文柯
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)
