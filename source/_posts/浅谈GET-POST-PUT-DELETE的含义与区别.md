---
title: '浅谈GET,POST,PUT,DELETE含义及区别'
date: 2017-08-18 10:10:03
tags:
- GET
- POST
- PUT
- DELETE
comments: true
categories: 日志
---

&nbsp;&nbsp;&nbsp;&nbsp; 近期在学习SpringMVC的有关知识，里面涉及到HTTP协议里的GET,POST,PUT和DELETE这四种与服务器交互的基本方法，感觉懵懵懂懂，不是很理解，于是上网查询相关资料。来做一下简单的总结，方便以后学习，有什么不对的地方欢迎指正。

---
## 含义
**1.GET**<br> GET请求用于向服务器发出索取请求，从而获取相关信息，它的操作是幂等的、安全的（不管进行多少次操作，该资源的状态都不会改变）。其类似于我们请求数据库的select(R)操作，只用作查询数据，没有副作用。

<!-- more -->
**2.POST**<br> POST请求用于向服务器发送数据，从而改变信息，该操作既不安全也不幂等，比较常见的问题：当我们多次发送POST请求时，会创建多个资源。其类似于请求数据库的insert(C)操作，创建新的内容。目前几乎所有的提交操作都使用POST请求。

**3.PUT**<br> PUT操作和POST请求类似，但是PUT操作是幂等的（不管进行多少次操作，结果是一样。如：修改了一篇文章，用PUT请求多次，该文章还是一样）。PUT操作类似于数据库的update(U)操作，修改相关类容。

**4.DELETE**<br> 顾名思义，DELETE请求用于删除某个数据，和数据库的delete(D)操作一样。

----
## 区别
### PUT和POST的区别
&nbsp;&nbsp;&nbsp;&nbsp; 如上文所说，PUT和POST都是向服务器发送数据，如果服务端允许，那么创建操作既可以用POST，也可以使用PUT。两者的区别在于：PUT方法是幂等的，而POST不是；在请求时，PUT方法需要作用在一个具体的资源上（如：“http://hi.baidu.com/mianshiti?key1=value1” ），它会将提交的数据编码在url中，而url的长度又会因浏览器的长度限制而存在数据传输有限问题；POST方法会将提交的数据放在请求的body中，不会显示在url上（如：“http://hi.baidu.com/mianshiti” ），因此没有数据大小限制的问题。<br>
&nbsp;&nbsp;&nbsp;&nbsp; 因为PUT方法把数据编码在url中，所以这些变量会显示在浏览器的地址栏上，也会被记录在服务器端的日志中，所以相比较之下，POST方法是更加安全的,此处的安全和GET提到的“安全”是不同概念，此处是真正意义上的Security。

----
### GET和POST的区别
&nbsp;&nbsp;&nbsp;&nbsp;首先说原理区别：
&nbsp;&nbsp;&nbsp;&nbsp;一般在浏览器输入网站访问资源都是通过GET方式，在form提交中，可以通过method改变提交方式；正如上文所说，GET方式是幂等的、安全的(只限于非修改信息)，而POST方法是既不幂等、也不安全的。GET方法一般用于**查询/获取**信息，而POST方法一般用于**更新/创建**资源信息。<br>
&nbsp;&nbsp;&nbsp;&nbsp; 上面说了原理性问题，但在实际操作的时候却没有多少人按照HTTP规范做，这也是导致GET和POST容易混淆的原因，导致这个问题的原因很多，比如说：
&nbsp;&nbsp;&nbsp;&nbsp;(1)很多人贪方便，更新资源时用了GET，POST方法要用form表单，麻烦一点。
&nbsp;&nbsp;&nbsp;&nbsp;(2)增删改查操作其实都可以用GET/POST方法完成。<br><br>

&nbsp;&nbsp;&nbsp;&nbsp;表现形式区别：
&nbsp;&nbsp;&nbsp;&nbsp;跟PUT和POST一样，GET方法也需要将提交的数据在地址栏上显示出来，所以POST是更加安全的；传输数据大小也和PUT与POST一样，GET方法会有数据大小的限制，这里需要强调一点：HTTP协议没有对传输数据大小进行限制，对url的长度也没有限制，但特定的浏览器和服务器对url长度是有限制的。

----
参考文章：[http://blog.csdn.net/gideal_wang/article/details/4316691](http://blog.csdn.net/gideal_wang/article/details/4316691)