# 抽象工厂模式

### 意图

涉及到 **产品族** 的时候，就需要引入抽象工厂模式了。

### 抽象工厂模式如何实现？

因为电脑是由许多的构件组成的，我们将 CPU 和主板进行抽象，然后 CPU 由 CPUFactory 生产，主板由 MainBoardFactory 生产，然后，我们再将 CPU 和主板搭配起来组合在一起，如下图：

![](/imgs/summary/ds-c-2-1.png)

客户端是这样调用的：

```java
// 得到 Intel 的 CPU
CPUFactory cpuFactory = new IntelCPUFactory();
CPU cpu = intelCPUFactory.makeCPU();

// 得到 AMD 的主板
MainBoardFactory mainBoardFactory = new AmdMainBoardFactory();
MainBoard mainBoard = mainBoardFactory.make();

// 组装 CPU 和主板
Computer computer = new Computer(cpu, mainBoard);
```

单独看 **CPU 工厂** 和 **主板工厂** ，它们分别是前面我们说的工厂模式。这种方式也容易扩展，因为要给电脑加硬盘的话，只需要加一个 HardDiskFactory 和相应的实现即可，不需要修改现有的工厂。

但是，这种方式有一个问题，那就是如果 **Intel 家产的 CPU 和 AMD 产的主板不能兼容使用** ，那么这代码就容易出错，因为客户端并不知道它们不兼容，也就会错误地出现随意组合。

### 产品族

下面就是我们要说的产品族的概念，它代表了组成某个产品的一系列附件的集合：

![](/imgs/summary/ds-c-2-2.png)

当涉及到这种产品族的问题的时候，就需要抽象工厂模式来支持了。我们 **不再定义 CPU 工厂、主板工厂、硬盘工厂、显示屏工厂** 等等，我们直接定义电脑工厂，每个电脑工厂负责生产所有的设备，这样能保证肯定不存在兼容问题。

![](/imgs/summary/ds-c-2-3.png)

这个时候，对于客户端来说，不再需要单独挑选 CPU厂商、主板厂商、硬盘厂商等，直接选择一家品牌工厂，品牌工厂会负责生产所有的东西，而且能保证肯定是兼容可用的。

```java
public static void main(String[] args) {
    // 第一步就要选定一个“大厂”
    ComputerFactory cf = new AmdFactory();
    // 从这个大厂造 CPU
    CPU cpu = cf.makeCPU();
    // 从这个大厂造主板
    MainBoard board = cf.makeMainBoard();
      // 从这个大厂造硬盘
      HardDisk hardDisk = cf.makeHardDisk();

    // 将同一个厂子出来的 CPU、主板、硬盘组装在一起
    Computer result = new Computer(cpu, board, hardDisk);
}
```

### 总结

抽象工厂模式是根据产品族将工厂抽象化，一个具体的工厂就能生产出一个完整的产品。但是抽象工厂模式的问题也是显而易见的，比如我们要加个显示器，就需要修改所有的工厂，给所有的工厂都加上制造显示器的方法。这有点违反了 **对修改关闭，对扩展开放** 这个设计原则。

## [Back](../../summary.md)