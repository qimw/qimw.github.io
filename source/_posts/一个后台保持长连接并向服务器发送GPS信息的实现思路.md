---
title: 一个后台保持长连接并向服务器发送GPS信息的实现思路
date: 2020-06-20 12:12:35
tags:
- Android
---

在开发中有时候会遇到这样一个需求，就是需要APP和后台保持一个连接，并且要向服务器定时发送gps信息，这里提供一个解决思路仅供参考。
<!--more-->

# 一个后台保持长连接并向服务器发送GPS信息的实现思路


## 需求

- APP保持一个与服务器的长连接，并且每隔一段时间向服务器发送gps信息
- 当手机熄屏时仍然能够在后台接着发送数据

## 解决思路

### 首先是TCP和HTTP、UDP的区别

#### HTTP

​使用HTTP通讯第一步是向服务器发送请求，这个请求包括请求头和请求内容，下面分开来讲。

​请求头（request header）：

- 请求的方法是POST/GET,请求的URL，http协议版本。

- 请求的数据，和编码方式。

- 是否有cookie，是否缓存等。

请求正文（request body）：客户端提交的信息。

POST和GET请求的区别：

- GET方法:GET方法是默认的HTTP请求方法，我们日常用GET方法来提交表单数据，然而用GET方法提交的表单数据只经过了简单的编码，同时它将作为URL的一部分向Web服务器发送，因此，如果使用GET方法来提交表单数据就存在着安全隐患上。例如Http://127.0.0.1/login.jsp?Name=zhangshi&Age=30&Submit=%cc%E+%BD%BB从上面的URL请求中，很容易就可以辩认出表单提交的内容。（？之后的内容）另外由于GET方法提交的数据是作为URL请求的一部分所以提交的数据量不能太大。

- 
POST方法:POST方法是GET方法的一个替代方法，它主要是向Web服务器提交表单数据，尤其是大批量的数据。POST方法克服了GET方法的一些缺点。通过POST方法提交表单数据时，数据不是作为URL请求的一部分而是作为标准数据传送给Web服务器，这就克服了GET方法中的信息无法保密和数据量太小的缺点。因此，出于安全的考虑以及对用户隐私的尊重，通常表单提交时采用POST方法。

​

实际上HTTP请求一共有八种，这里就不介绍了，有兴趣的话可以自己google下。

向服务器发送请求之后，接下来服务器就会给客户端返回HTTP相应。同样包括了header和body两部分，这两部分就比较简单了，response header包括了1.cookies或者sessions2.状态码3.内容大小等，response body就是服务器返回的信息。

#### TCP

![三次握手](https://img-blog.csdn.net/20180406192625392?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nfa2l0ZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

​     手机能够使用联网功能是因为手机底层实现了TCP/IP协议，可以使手机终端通过无线网络建立TCP连接。TCP协议可以对上层网络提供接口，使上层网络数据的传输建立在“无差别”的网络之上。

​    建立起一个TCP连接需要经过“三次握手”：

第一次握手：客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；

第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；

第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。

​     握手过程中传送的包里不包含数据，三次握手完毕后，客户端与服务器才正式开始传送数据。理想状态下，TCP连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP 连接都将被一直保持下去。断开连接时服务器和客户端均可以主动发起断开TCP连接的请求，断开过程需要经过“四次握手”。HTTP也是基于TCP实现的。

​    在这里我们用到的是Socket实现和服务器的连接。socket就是对TCP/IP协议的封装和应用。使用socket建立连接之后，每隔一段时间即使没有进行通信也要发送心跳包。其中的原因简单来说（详细原因可以看下面的参考资料），如果一个连接一段时间没有进行通信的话，那么网络提供商就会自动断开连接，这样就会对我们的程序造成影响。这时候你可能就要问了，如果每个程序的客户端都要不断发送这种无意义的心跳包的话，会不会对网络造成很大的负担。确实是这样的，这种情况我们称为“信令风暴”，可以参考一下最后的资料，这里就不介绍了。

#### UDP

​    UDP（User Data Protocol，用户数据报协议）是与TCP相对应的协议。它是面向非连接的协议，它不与对方建立连接，而是直接把数据包发送过去！
​    UDP适用于一次只传送少量数据、对可靠性要求不高的应用环境。比如，我们经常使用“ping”命令来测试两台主机之间TCP/IP通信是否正常，其实“ping”命令的原理就是向对方主机发送UDP数据包，然后对方主机确认收到数据包，如果数据包是否到达的消息及时反馈回来，那么网络就是通的。UDP协议是面向非连接的协议，没有建立连接的过程。正因为UDP协议没有连接的过程，所以它的通信效果好；但也正因为如此，它的可靠性不如TCP协议高。

### 常驻后台的方法

​    我们立刻应该想到的是自定义一个Service，在Service中创建一个线程来与服务器进行Socket连接。但是如果不采取任何措施的话，在系统内存不足时就会被系统杀掉，即使内存充足，手机休眠之后网络连接也还是会被断开，这样显然是不符合我们的需求的，那么如何才能解决这些问题呢。

​    首先我们看如何避免内存不足时被系统杀掉。网上有三类Service的保活方法，分别是创建前台Service、相互唤醒和利用系统的漏洞来启动一个前台的Service进程（还有一种方法就是联系手机厂商，把你的APP加入白名单，不过这种方法成本太高）。这里因为我们做的APP不是流氓软件，所以说就采用了启动前台Service的方法。如果想了解其他两种方法可以看最后面的参考资料。

​    当采用上面这种思路实现之后，运行一段时间我们就会发现Socket连接还是被中断了，这是因为在手机休眠时，系统会停止耗电的操作，网络连接显然是其中的一种。这时候就要隆重介绍WakeLock了。

​    官方文档是这样介绍WakeLock的：

> A wake lock is a mechanism to indicate that your application needs to have the device stay on.
> 
> Any application using a WakeLock must request the `android.permission.WAKE_LOCK` permission in an `<uses-permission>` element of the application’s manifest. Obtain a wake lock by calling `newWakeLock(int, String)`.
> 
> Call `acquire()` to acquire the wake lock and force the device to stay on at the level that was requested when the wake lock was created.
> 
> Call `release()` when you are done and don’t need the lock anymore. It is very important to do this as soon as possible to avoid running down the device’s battery excessively.

 重点就在第一行，wakelock机制可以让你的手机保持唤醒状态。下面就是如何使用它了。我们接着看文档。

> 任何使用WakeLock的应用必须申请`android.permission.WAKE_LOCK`权限，并且通过调用`newWakeLock(int, String)`方法来获得wake lock。获得wakelock之后就可以调用`acquire()`来获得锁，调用`release()`来释放锁。注意如果不释放这个锁的话，就会一直响应耗电的操作，手机就不能正常休眠，造成的后果就是电量的大量消耗。

### 实现方式

- 使用Service保持连接并处理心跳包和gps信息的发送
- 在产生搜索到GPS信号前发送空包
- 因为GPS会间隔一段时间产生一次GPS信息，所以我们可以使用一个线程安全的队列，将gps信息enqueue，再开一个新的线程每隔一段时间dequeue一下，不为空就发送。
- 采用前台服务：内存不足时Service保活
- wakelock：手机休眠时保持gps和tcp连接
- 测试情况（这个没啥卵用，懒得删了，为啥没用看下面那几个坑）

情况(测试机：红米4)GPSTCPAPP未使用前台服务和wakelock约1min后gg约3min后gg内存充足，继续运行只使用前台服务约1min后gg约4min后gg内存充足，继续运行只使用wakelockgggg内存充足，继续运行两种方式都使用ggggalive
### 最后还有几个坑

- MIUI神隐模式手机休眠后即使使用wakelock也不能保持cpu唤醒状态，要把你的APP手动加入白名单才可以。这一点设计得很智障。
- Socket 的 isClosed() 方法不能判断是否保持了连接，因为只有手动调用close()方法时才会返回true，没有调用close()方法时，即使断开了连接，返回的也是false。
- 同样 isConnected()也不能判断，这个方法返回的是是否**曾经**建立过连接，比方说，我现在建立了连接了，然后网断了，现在没网了，这个方法还是返回true。
- 所以说还是根据你发送包的时候服务器是不是返回了正确的字段来判断是否保持了连接（目前为止我只想到这一种方法）。
- socket要做断线重连机制。
- socket断掉期间产生的gps信息在恢复连接之后要及时发送。（我是将他们存在数据库了）。

参考资料：

> Service保活手段 [http://www.jianshu.com/p/63aafe3c12af](http://www.jianshu.com/p/63aafe3c12af)
> 
> 为什么要发送心跳包 [http://blog.csdn.net/long704480904/article/details/47029417](http://blog.csdn.net/long704480904/article/details/47029417)
> 
> 信令风暴[https://www.zhihu.com/question/20849677](https://www.zhihu.com/question/20849677)
