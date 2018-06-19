# Book
![](/imgs/jvm/jvm-book.jpg "《深入理解Java虚拟机 第2版》")

## 目录

- [Java内存区域与内存溢出异常](#user-content-OOM)
    - [运行时数据区域](#user-content-RUNTIME-AREA)
    - [HotSpot虚拟机对象](#user-content-HOTSPOT-OBJ)
    - [总结](#user-content-OOM-SUMMARY)
- [垃圾收集器与内存分配策略](#user-content-GC-MECHINE-AND-MEMORY-ALLOCATION-STRATEGY)
    - [概述](#user-content-GC-OVERVIEW)
    - [引用](#user-content-REFERENCE)
    - [垃圾收集算法](#user-content-GC-ARITHMETHIC)
    - [HotSpot的算法实现](#HOTSPOT)
    - [内存分配与回收策略](#MEMORY-ALLOCATION-STRATEGY)
- [类文件结构](#CLASS-FILE-STRUCTURE)
- [虚拟机类加载机制](#CLASSLOADER)
    - [类加载的时机](#CLASS-LOAD-TIME)
    - [类加载的过程](#CLASS-LOAD-PROCESS)
    - [验证](#FILE-VALIDATION)
    - [准备](#FILE-PREPARE)
    - [解析](#FILE-ANALYSIS)
    - [初始化](#FILE-INIT)
    - [类加载器](#CLASSLOADER-MECHINE)
- [Java内存模型与线程](#MEMORY-MODEL-AND_THREAD)
    - [物理机的内存模型](#PHYSIC-MEMORY-MODEL)
    - [Java内存模型](#JAVA-MEMORY-MODEL)
    - [Java与线程](#JAVA-AND-THREAD)
- [线程安全和锁优化](#THREAD-SECURITY-AND-LOCK-OPTIMIZE)

### <a id="OOM">Java内存区域与内存溢出异常</a>
![运行时数据区域](/imgs/jvm/jvm-1.png)

1. PC Register(Program Counter Register，线程私有)，是当前线程所执行的字节码的行号指示器。
    - 此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。（这是一个固定的整数的储存空间，所以没有规定OOM）字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。
    - Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的。为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的PC Register，各条线程之间计数器互不影响，独立储存。
    - PC Register值：
        1. 线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址。
        2. 线程正在执行的是Native方法，这个计数器值为空（undefined）。

2. VM Stack(Java 虚拟机栈，线程私有)，生命周期和线程相同。
    - 虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建出一个栈帧(Stack Frame)，用于储存局部变量表、操作数栈、动态链接、方法出栈等信息。
    - 局部变量表，存放了编译期可知的各种基本数据类型、对象引用。注意：其中64位长度的long和double类型的数据会占用2个局部变量空间(Slot)，其余的基本数据类型都只占1个。