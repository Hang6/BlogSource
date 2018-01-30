---
title: Spring AOP的实现
date: 2018-01-22 14:56:01
tags:
- Spring
- AOP
comments: true
categories: 原创
---
# AOP概述
&nbsp;&nbsp;&nbsp;&nbsp;Spring AOP最重要的设计模式——**代理模式**

## Advice通知
&nbsp;&nbsp;&nbsp;&nbsp;Advice（通知）定义在连接点做什么，为切面增强提供织入接口。Advice是AOP联盟定义的一个接口，在Spring AOP的实现中，使用了这个统一接<!-- more -->口，为AOP切面增强的织入功能做了更多的细化和扩展，比如提供了更具体的通知类型，如BeforeAdvice、AfterAdvice、ThrowsAdvice等。

## Pointcut切点
&nbsp;&nbsp;&nbsp;&nbsp;Pointcut决定Advice通知应该作用于哪个连接点，也就是说通过**Pointcut**来**定义**需要增强的方法的**集合**，这些集合的选取可以按照一定的规则来完成。

&nbsp;&nbsp;&nbsp;&nbsp;Pointcut需要返回一个**MethodMatcher**，由这个MethodMatcher来**判断是否需要对当前方法调用进行增强**，或者是否需要对当前调用方法应用配置好的Advice通知。

## Advisor通知器
&nbsp;&nbsp;&nbsp;&nbsp;完成对目标方法的切面增强设计和关注点的设计以后，需要一个Advisor对象把它们结合起来。通过Advisor，可以定义应该使用哪个通知并在哪个关注点使用它。

---
# Spring AOP的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;Spring AOP最核心的思想便是使用**代理对象**来完成对目标方法的调用，使其能按照我们配置的方式去实现切面增强。在Spring中，是通过配置和调用ProxyFactoryBean来完成建立代理对象这个任务。配置如下：

```
<bean id="testAdvisor" class="com.abc.TestAdvisor"/>
<bean id="testAOP" class="org.springframework.aop.ProxyFactoryBean">
<property name="proxyInterfaces">
    <value>com.test.AbcInterface</value>
</property>
<property name="target">
    <bean class="com.abc.TestTarget"/>
</property>
<property name="interceptorNames">
    <list><value> testAdvisor</value></list>
</property>
</bean>
```
## AopProxy代理对象
&nbsp;&nbsp;&nbsp;&nbsp;代理对象可以通过两种方式生成，一种是**JDK动态代理**，另一种是**CGLIB**来生成。如果目标对象是接口类，适合使用JDK来生成代理对象，否则Spring会使用CGLIB来生成目标对象的代理对象。

&nbsp;&nbsp;&nbsp;&nbsp;对于JDK动态代理，通过调用Proxy的newProxyInstance方法生成代理对象。在生成对象时，需要指明三个参数，一个是类装载器，一个是代理接口，另一个是Proxy回调方法所在的对象，这个对象需要实现**InvocationHandler接口**，这个接口定义了**invoke()方法**，提供代理对象的回调入口，这个方法完成了AOP编织实现的封装。<br>
&nbsp;&nbsp;&nbsp;&nbsp;对于CGLIB，是通过**Enhancer对象**生成代理对象。在Enhancer中，通过设置callback回调，封装了AOP的实现。

&nbsp;&nbsp;&nbsp;&nbsp;在JDk动态代理中，代理对象会通过**invoke()方法**来完成对拦截器链和目标方法的调用；而CGLIB会通过**intercept()方法**来调用拦截器链和目标方法。如果没有设置拦截器，那么会对目标方法直接进行调用。

## AOP拦截器链的调用
&nbsp;&nbsp;&nbsp;&nbsp;AOP对目标对象增强的实现封装在AOP的拦截器链中，由一个个具体的拦截器来完成。虽然有两种不同的代理方式，但是它们对AOP拦截的处理都殊途同归：它们**对拦截器链的调用都是在ReflectiveMethodInvocation中通过proceed方法实现的**。

&nbsp;&nbsp;&nbsp;&nbsp;在运行拦截器的拦截方法之前，需要通过**Pointcut切点中的matches对代理方法完成匹配判断**，通过这个判断来决定拦截器是否满足切面增强的要求。满足后，会递归调用proceed方法，逐个运行拦截器的拦截方法。以此来完成了目标方法的切面增强。

---
参考：《Spring技术内幕》——计文柯
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)