---
title: JVM运行时数据区
date: 2017-07-07 00:00:00
tags:
- JVM
categories: 
- JAVA
---

这是第一篇介绍JVM的文章，所以是一些概念性的东西，先了解一下即可。主要介绍了JVM中运行时数据区的分类（程序计数器、Java虚拟机栈、本地方法栈、Java堆、方法区等）以及各部分的作用。
<!--more-->


# JVM-运行时数据区

​    这是关于JVM的第一篇文章，主要介绍JVM中运行时数据区的分类以及各个部分的作用。

​    Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域，这些区域都有各自的用途、以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。Java虚拟机所管理的内存包括一下几个运行时的数据区域：

![运行时数据区](20180224095734717.png)

 下面我们分别进行介绍

## 程序计数器

​    程序计数器（Program Counter Register）是一块比较小的内存空间，它可以看作是当先线程所执行的字节码的行号指示器，类似于操作系统中的CS:IP。但是为了保证切换线程之后能够正确执行字节码，每一个线程都有自己的一个程序计数器。我们将这种内存区域称为“线程私有”内存。

​    如果当前线程执行的是一个Java方法，那么这个计数器记录的是正在执行的虚拟机字节码指令的地址。但是如果正在执行的是Native方法，这个计数器的值为空。这块内存区域也是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。

## Java虚拟机栈

![虚拟机栈结构](20180223223921237.png)

​    Java虚拟机栈（Java Virtual Machine Stacks）也是内存私有的。虚拟机栈描述的是Java方法执行的内存模型，每个方法执行是都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等**有关这个方法的**信息。每一个方法从调用直至执行完成的过程，就对应着一个栈桢在虚拟机栈中入栈到出栈的过程。

​    在Java虚拟机规范中，对这个区域规定了两种一场状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常；如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），如果扩展时无法申请到足够的内存就会抛出OutOfMemeryError异常。

​    虚拟机栈的结构如图所示，分为局部变量表、操作数栈、动态链接以及返回地址。下面我们对这四个部分分别进行介绍。

### 局部变量表

​    首先我们介绍一下变量槽 （Variable Slot）的概念。变量槽是局部变量表的最小部分单位，一般为32位，但是没有强制规定。局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，但是它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄（Java句柄的概念在Java堆中会提到）或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。

​    其中64位长度的long和double类型的数据会占用2个变量槽大小，其余的数据了类型只占用一个。局部变量表所需的内存空间在编译期间分配完成，所以当进入一个方法时，这个方法需要在帧中分配多大的变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

### 操作数栈

​    操作数栈（Operand Stack）也常称为操作栈，是一个后入先出栈。在Class 文件的Code 属性的 max_stacks 指定了执行过程中最大的栈深度。Java 虚拟机的解释执行引擎称为”**基于栈的执行引擎**“，这里的栈就是指操作数栈。

​    方法执行中进行算术运算或者是调用其他的方法进行参数传递的时候是通过操作数栈进行的。

​    在概念模型中，两个栈帧是相互独立的。但是大多数虚拟机的实现都会进行优化，令两个栈帧出现一**部分重叠**。令下面的部分操作数栈与上面的局部变量表重叠在一块，这样在方法调用的时候可以共用一部分数据，无需进行额外的参数复制传递。

### 动态链接

​    每个栈帧都包含一个执行运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的**动态连接**（Dynamic Linking）。

​    Class 文件中存放了大量的符号引用，字节码中的方法调用指令就是以常量池中指向方法的符号引用作为参数。这些符号引用一部分会在类加载阶段或第一次使用时转化为直接引用，这种转化称为**静态解析**。另一部分将在每一次运行期间转化为直接引用，这部分称为**动态连接**。

### 方法返回地址

当一个方法开始执行以后，只有两种方法可以退出当前方法：

- 当执行遇到返回指令，会将返回值传递给上层的方法调用者，这种退出的方式称为正常完成出口（Normal Method Invocation Completion），一般来说，调用者的PC计数器可以作为返回地址。
- 当执行遇到异常，并且当前方法体内没有得到处理，就会导致方法退出，此时是没有返回值的，称为异常完成出口（Abrupt Method Invocation Completion），返回地址要通过异常处理器表来确定。

当方法返回时，可能进行3个操作：

- 恢复上层方法的局部变量表和操作数栈
- 把返回值压入调用者栈帧的操作数栈
- 调整 PC 计数器的值以指向方法调用指令后面的一条指令

## 本地方法栈

​    本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行Java方法服务，而本地方法栈i则为虚拟机使用到的Native方法服务。在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机直接就吧本地方法栈和虚拟机栈合二为一。本地方法栈与虚拟机栈会产生同样的Error。

## Java堆

​    Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例都要在这里分配内存。但是随着JIT编译器的发展与逃逸分析技术的逐渐成熟，栈上分配、标量替换优化技术将带来一些改变，所有的对象分配在堆上也渐渐变得不是那么绝对了。

​    Java堆是垃圾收集器管理的主要区域（另一部分是方法区），因此很多时候也被称为”GC堆“，从内存回收的角度来看，由于现在的收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代，在细致一点的有Eden空间、From Survivor空间、To Survivor空间等。从内存分配的角度来看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer TLAB），用于提前分配给线程，进行对象的创建。

​    根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可。在实现时，可以是固定大小，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的（通过-Xmx和-Xms控制）。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

## 方法区

​    方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，用于存储已经被虚拟机加载的类的信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范吧方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap，目的应该是与Java堆区分开。

​    方法区除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。但是在HotSpot虚拟机中是将这个区域作为永久代来实现，这样就可以使用GC进行管理。

## 运行时常量池是

​    运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池（Constant Pool Table）用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池存放。

​    既然运行时常量池是方法区的一部分，自然受到方法区的内存限制，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

## 直接内存

​    直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError，所以我们放在这里一起讲解。

​    在Jdk1.4中新加入了NIO类，引入了一种基于通道与缓冲区的I/O方式，它可以使用Native函数库直接分配堆外内存，然后用一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中复制数据。

​    这样导致的一个问题是，各个内存区域总和大于物理内存限制（包括物理的和操作系统级的限制），从而导致动态扩展时出现OutOfMemoryError异常。 
