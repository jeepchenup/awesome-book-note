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
    - [HotSpot的算法实现](#user-content-HOTSPOT)
    - [内存分配与回收策略](#user-content-MEMORY-ALLOCATION-STRATEGY)
- [类文件结构](#user-content-CLASS-FILE-STRUCTURE)
- [虚拟机类加载机制](#user-content-CLASSLOADER)
    - [类加载的时机](#user-content-CLASS-LOAD-TIME)
    - [类加载的过程](#user-content-CLASS-LOAD-PROCESS)
    - [验证](#user-content-FILE-VALIDATION)
    - [准备](#user-content-FILE-PREPARE)
    - [解析](#user-content-FILE-ANALYSIS)
    - [初始化](#user-content-FILE-INIT)
    - [类加载器](#user-content-CLASSLOADER-MECHINE)
- [Java内存模型与线程](#user-content-MEMORY-MODEL-AND_THREAD)
    - [物理机的内存模型](#user-content-PHYSIC-MEMORY-MODEL)
    - [Java内存模型](#user-content-JAVA-MEMORY-MODEL)
    - [Java与线程](#user-content-JAVA-AND-THREAD)
- [线程安全和锁优化](#user-content-THREAD-SECURITY-AND-LOCK-OPTIMIZE)

### <a id="OOM">Java内存区域与内存溢出异常</a>

#### <a id="RUNTIME-AREA">运行时数据区域</a>
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
    - StackOverflowError（内存溢出）与OutOfMemoryError（内存泄漏）的区别:
        1. 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。
        2. 如果虚拟机栈可以动态扩展且扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

3. Native Method Stack(本地方法栈，线程私有)
    - 与虚拟机栈之间的区别就是虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则为虚拟机使用到的Native方法服务。
    - 本地方法栈区也会抛出StackOverflowError和OutOfMemoryError异常。

4. Heap(Java堆、GC堆，线程共享)
    - Java堆是Java虚拟机所管理的内存中最大的一块，是垃圾收集器管理的主要区域。
    - Java堆是被所有线程共享的一块内存区域。
    - Java堆存放对象实例以及数组。
    - 按分代收集算法来分：新生代和老年代。
    - 划分的目的是为了更好地回收内存，或者更快地分配内存。

5. Method Area(方法区，线程共享)
    - 方法区，用于储存已被虚拟机加载的类信息、常量、静态变量、及时编译器编译后的代码等数据。
    - Runtime Constant Pool(运行时常量池)，是方法区的一部分，所以运行时常量池自然受到方法区内存的限制，当常量池无法再申请到内存的时候，将会抛出OutOfMemoryError异常。
    - Constant Pool Table(常量池)，用于存放编译期生成的各种字面量和符号引用。

6. 直接内存
    - 直接内存既不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。
    - 由于频繁使用，也可能导致OutOfMemoryError异常的出现。

### <a id="HOTSPOT-OBJ">HotSpot虚拟机对象</a>

1. 创建对象（new指令）
    - 检查这个指令的参数是否能在常量池中定位到一个类的符号引用。
    - 检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，必须先执行相应的类加载过程。
    - 类加载检查通过之后（其实对象所需的内存大小已经完全确定），JVM将为新生对象分配内存，此时一块确定大小的内存从Java堆中划分出来。
    - Java堆的划分方式：选择哪种分配方式由**Java堆是否规整决定**
        - 指针碰撞(Bump the Pointer),假设Java堆中内存是**绝对规整**的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器。
        - 空闲列表(Free List),假设Java堆中的内存并**不是规整**的，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录。

> 对象在内存中储存的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）、对齐填充（Padding）。
2. 对象的内存布局
    - HotSpot虚拟机的**对象头**包括2部分：
        - 第一部分，用于储存对象自身的运行时数据，哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID等。
        - 第二部分是类型指针，即对象指向它的类元数据的指针。虚拟机通过这个指针来确定这个对象是哪个类的实例。
        - 如果对象是一个Java数组，对象头中还有一块用于记录数组的长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中却无法确定数组的大小。
    - **实例数据**，是对象真正储存的有效信息，也是在程序代码中所定义的各种类型的字段内容。
    - **对齐填充**，并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。对于HotSpot VM要求对象的大小必须是8字节的整数倍。当对象实例数据部分没有对齐时，就需要通过对齐填充来不全。

3. 对象的访问定位
    - 主流的访问方式：句柄、直接指针。
        - **句柄**，Java堆中将会划分出一块内存作为句柄池，reference中储存的就是对象的句柄地址，而句柄中包含了类信息。
        ![句柄模型](/imgs/jvm/jvm-2.png)
        - **直接指针**
        ![直接指针模型](/imgs/jvm/jvm-3.png)