---
title: Spring 框架的核心：IoC容器的实现
date: 2017-11-16 21:21:37
tags:
- Spring
- IoC容器
- 依赖注入
comments: true
categories: 原创
---
# Spring IoC容器的设计
## 概述
&nbsp;&nbsp;&nbsp;&nbsp;在面向对象系统中，对象封装了数据和对数据的处理，对象的依赖关系常常体现在对数据和方法的依赖上。在实际编程中，我们应该尽量实现高内聚低耦<!-- more -->合，将依赖关系交给框架和IoC容器来实现就是一种不错的实现方法。

&nbsp;&nbsp;&nbsp;&nbsp;依赖控制反转的实现有很多种方式。在Spring中，IoC容器是实现这个模式的载体，它可以在对象生成或初始化时直接将数据注入到对象中，也可以将对象引用注入到对象数据域中的方式来注入对方法调用的依赖。

## BeanFactory
&nbsp;&nbsp;&nbsp;&nbsp;BeanFactory提供的是最基本的Ioc容器的功能。**BeanFactory只是一个接口类**，直接的BeanFactory实现是Ioc容器的**基本形式**，它提供了使用IoC容器的规范，而DefaultListableBeanFactory、XmlBeanFactory、ApplicationContext等都可以是容器附加了某种功能的具体实现。

&nbsp;&nbsp;&nbsp;&nbsp;**BeanFactory和FactoryBean**：在Spring中，所有的Bean都是由**BeanFactory**（也就是Ioc容器）来进行**管理**的。而FactoryBean，他是一个能产生或者修饰对象生成的**工厂Bean**，他的实现与设计模式中的工厂模式和修饰器模式类似。

## ApplicationContext
&nbsp;&nbsp;&nbsp;&nbsp;ApplicationContext是一个高级形态的Ioc容器，他在BeanFactory的基础上添加了附加功能，这些功能为ApplicationContext提供了以下BeanFactory所不具备的新特性：
- 支持不同的信息源。
- 访问资源。
- 支持应用事件。
- 对它的使用是一种面向框架的使用风格。


## IoC容器中Bean的生命周期
1. Bean实例的创建
2. 为Bean实例设置属性
3. 调用Bean的初始化方法
4. 应用可以通过IoC容器使用Bean
5. 当容器关闭时，调用Bean的销毁方法


---
# IoC容器的初始化过程
&nbsp;&nbsp;&nbsp;&nbsp;IoC容器的初始化是由**refresh()方法**来启动的，这个方法标志着IoC容器的正式启动。这个启动包括BeanDefinition的**Resource定位、载入和注册**三个基本过程。Spring是把这三个过程分开，并使用不同的模块来完成的。

&nbsp;&nbsp;&nbsp;&nbsp;IoC容器的初始化过程完成的主要工作就是在IoC容器中**建立BeanDefinition**的**数据映射**。Spring通对定义BeanDefinition来管理基于Spring的应用中的各种对象以及它们之间的相互依赖关系。对IoC容器来说，BeanDefinition就是对依赖反转模式中管理对象依赖关系的**数据抽象**，也是容器实现依赖控制反转功能的核心数据结构。

## BeanDefinition的Resource定位
&nbsp;&nbsp;&nbsp;&nbsp;这个Resource定位指的是BeanDefinition的**资源定位**，它是由ResourceLoader的**通过统一的Resource接口**来完成的。在定位过程完成后，为BeanDefinition的载入创造了I/O操作的条件，但是具体的数据还没有开始读入。

## BeanDefinition的载入和解析
&nbsp;&nbsp;&nbsp;&nbsp;这个**载入过程**相当于把定义好的**Bean在IoC容器中转化成一个Spring内部表示的数据结构**的过程，而这个容器内部的数据结构就是**BeanDefinition**。

&nbsp;&nbsp;&nbsp;&nbsp;IoC容器对Bean的**管理**和**依赖注入**功能的实现，是通过对其持有的BeanDefinition进行各种相关操作操作来完成的。这些BeanDefinition数据在IoC容器中通过一个HashMap来保持和维护。

&nbsp;&nbsp;&nbsp;&nbsp;BeanDefinition的载入分为两部分，首先通过调用XML的解析器得到document对象，但这些document对象并没有按照Spring的Bean规则进行解析；在完成通用的XML解析后，才是按照Spring的Bean规则进行解析，这个过程是在documentReader中实现的。

## BeanDefinition在IoC容器中的注册
&nbsp;&nbsp;&nbsp;&nbsp;在BeanDefinition完成载入和解析之后，用户定义的BeanDefinition信息已经在IoC容器内建立起了自己的数据结构以及相应的数据表示，但是这些数据还**不能直接提供给IoC容器**使用，需要在IoC容器中对这些BeanDefinition数据进行注册。

&nbsp;&nbsp;&nbsp;&nbsp;注册过程是通过调用BeanDefinitionRegistry接口来实现完成的。这个注册就是在IoC容器内部将BeanDefinition注入到一个HashMap中去，**IoC容器**就是**通过这个HashMap**来**持有这些BeanDefinition数据**的。HashMap的key是beanName，value是BeanDefinition。

---
# IoC容器的依赖注入
&nbsp;&nbsp;&nbsp;&nbsp;依赖注入的过程是在用户**第一次**向IoC容器索要Bean时触发的；但是我们可以在BeanDefinition信息中通过**lazy-init属**性来让容器完成对BeanDefinition的**预实例化**，让依赖注入在初始化的过程中完成。getBean()就是依赖注入发生的入口，之后会调用createBean。

&nbsp;&nbsp;&nbsp;&nbsp;与依赖注入关系特别密切的方法有**createBeanInstance**和**populateBean**。createBeanInstance中生成了Bean所包含的Java对象，这个对象可以通过工厂方法生成，也可以通过容器的autowired特性生成。populateBean对依赖关系进行了处理（对Bean对象的属性的处理）。

---

&nbsp;&nbsp;&nbsp;&nbsp;在Bean的创建和对象依赖注入的过程中，需要依据BeanDefinition中的信息来**递归**的完成依赖注入。这些递归都是以getBean()为入口的。

&nbsp;&nbsp;&nbsp;&nbsp;一个递归是在上下文体系中查找需要的Bean和创建Bean的递归调用。另一个递归是在依赖注入时，通过递归调用容器的getBean()方法，得到当前Bean的依赖Bean，同时也触发对依赖Bean的创建和注入。

&nbsp;&nbsp;&nbsp;&nbsp;在对Bean的属性进行依赖注入时，解析的过程也是一个递归过程。这样，根据依赖关系，一层一层的完成Bean的创建和注入，直到最后完成当前Bean的创建。

---
参考：《Spring技术内幕》——计文柯
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)