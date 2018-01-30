---
title: Spring MVC的请求流程
date: 2017-11-29 19:25:08
tags:
- Spring MVC
- DispatcherServlet
comments: true
categories: 原创
---
# 概述
&nbsp;&nbsp;&nbsp;&nbsp;Spring MVC是建立在IoC容器的基础上的，Spring IoC是一个独立的模块，它并不是直接在Web容器中发挥作用，要想在Web容器中使用IoC容器，需要Spring**为IoC设计一个启动过程**，把IoC容器导入，并**在Web容器中建立**起来。

&nbsp;&nbsp;&nbsp;&nbsp;这个过程是和Web容器的启动过程**集成**在一起的。这个过程中，一方面处理Web容器的启动，另一方面通过设计特定的**Web容器拦截器**，**将IoC容器载入**到Web环境中来，并将其**初始化**。<!-- more -->
以Tomcat作为Web容器为例，其中，web.xml是应用的部署描述文件：

``` bash
<servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

&nbsp;&nbsp;&nbsp;&nbsp;上述代码中有一个MVC中非常重要的类：**DispatcherServlet**，它起着**分发请求**的作用；**context-param参数**的配置用来**指定**Spring IoC容器读取**Bean定义的XML文件的路径**；**ContextLoaderListener**是一个监听器，他的生命周期与Web服务器相关联，这个监听器**负责完成IoC容器在Web环境中的启动工作**。

---
<br>

# 上下文在Web容器中的启动
## IoC容器启动的基本过程
&nbsp;&nbsp;&nbsp;&nbsp;IoC容器的启动过程就是**建立上下文**的过程，由ContextLoaderListener启动的上下文为**根上下文**。在根上下文的基础上，还有一个与Web MVC相关的上下文来**保存控制器（DispatcherServlet）需要的MVC对象**，它是根上下文的**子上下文**。这两者构成一个层次化的上下文体系，这个体系是由**ContextLoader**来完成。

&nbsp;&nbsp;&nbsp;&nbsp;在ContextLoader中，完成了**两个IoC容器**建立的基本过程，一个是在Web容器中建立起双亲IoC容器（根上下文），另一个是生成相应的WebApplicationContext（子上下文）并将其初始化。
<br>

## ContextLoader的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;ContextLoaderListener通过使用**ContextLoader**来完成实际的WebApplicationContext，也就是**IoC容器的初始化工作**。它就像Spring应用程序在Web容器中的启动器。

&nbsp;&nbsp;&nbsp;&nbsp;ContextLoader在监听器的ServletContextListener接口实现类的初始化回调中创建，同时会利用创建出来的ContextLoader来完成IoC容器的初始化。**初始化过程**中，完成了**根上下文**在Web容器中的创建，创建成功后，根上下文会被存到Web容器的**ServletContext**中去，这样就建立了一个**全局的关于整个应用的上下文** 。

---
<br>

# Spring MVC的设计与实现
&nbsp;&nbsp;&nbsp;&nbsp;在上文的代码中，可以看到DispatcherServlet也进行了配置，它是一个**前端控制器**，所有的Web请求都要通过它来处理，进行转发、匹配、数据处理后，并转由页面进行展现。因此这**个DispatcherServlet**可以看成是Spring MVC实现中**最为核心的部分**。

&nbsp;&nbsp;&nbsp;&nbsp;除了这条主线，在Spring MVC中，对于不同的Web请求的**映射需求**，Spring MVC提供了不同的**HandlerMapping**的实现。另外，不同Controller的实现也对应了不同的控制器使用场景，这些Controller控制器需要实现**handleRequest接口**方法，并返回**ModelAndView**对象。
<br>

## Spring MVC设计概述
&nbsp;&nbsp;&nbsp;&nbsp;在完成ContextLoaderListener的初始化后，Web容器开始初始化DispatcherServlet。<br>
&nbsp;&nbsp;&nbsp;&nbsp;DispatcherServlet会**建立自己的上下文**来持有Spring MVC的Bean对象，在建立自己持有的IoC容器时，会从ServletContext中**得到根上下文作为**DispatcherServlet持有上下文的**双亲上下文**。有了这个根上下文，再对自己持有的上下文进行初始化，然后保存到ServletContext中，供以后检索和使用。DispatcherServlet的启动过程就是Spring MVC的启动过程。

&nbsp;&nbsp;&nbsp;&nbsp;DispatcherServlet的工作大致可以分为两部分：
1. **初始化**，由initServletBean()启动，通过initWebApplicationContext()方法最终调用DispatcherServlet的**initStrategies方法**，在这个方法里，DispatcherServlet对MVC其他部分进行了初始化，比如**handlerMapping和ViewResolver**等。
2. **对Http请求进行响应**，调用doService()方法，在这个方法调用里封装了doDispatch()方法，它是实现MVC模式的主要部分。
<br>

## MVC处理HTTP分发请求
&nbsp;&nbsp;&nbsp;&nbsp;在DispatcherServlet初始化完成时，在上下文环境中定义的所有HandlerMapping都已经被加载了。HandlerMapping接口中定义了一个**getHandler方法**，它可以**获得HTTP请求对应的HandlerExecutionChain**；这个Chain持有一个**Interceptor链**和一个**handler对象**，**handler对象其实就是HTTP请求对应的Controller**，而HTTP请求和handler的映射关系是通过一个**handlerMap（HashMap）来保存**的。

&nbsp;&nbsp;&nbsp;&nbsp;在接受到HTTP请求之后，DispatcherServlet调用**doService()方法对HTTP请求参数进行快照处理**，在doService方法中，封装了doDispatcher()方法，而**对请求的处理**过程实际上是**由doDispatcher()方法来完成**的：通过getHandler获得HandlerExecutionChain（HandlerAdapter对handler进行合法判断）对请求进行处理后，**处理结果会封装到ModelAndView对象**中，为视图提供展现数据。
<br>

## Spring MVC视图的呈现
&nbsp;&nbsp;&nbsp;&nbsp;将处理结果封装到ModelAndView对象中后，为了对视图进行呈现，会从ModelAndView对象中取得**视图对象**，然后调用视图对象的**render()方法**，由这个视图对象来完成特定的视图呈现工作。

&nbsp;&nbsp;&nbsp;&nbsp;在render()方法中，有一个**resolveViewName()方法**，这个方法会**寻找视图对象的逻辑名**，如果在ModelAndView中设置了视图对象的名称，就调**用ViewResolver方法进行解析**，具体实现为：直接到上下文中通过名称的对应关系取到作为View对象的Bean。<br>
&nbsp;&nbsp;&nbsp;&nbsp;如果ModelAndView中已经有了最终完成视图呈现的视图对象，就直接使用该对象。

---
参考：《Spring技术内幕》——计文柯
转载请注明出处：[小Hang同学的博客](http://www.yhang6.com/)