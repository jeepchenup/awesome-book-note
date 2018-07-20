# 理解<? extends T>和<? super T>

> 引自知乎答案：[胖君](https://www.zhihu.com/people/pang-pang-37-37)

`<? extends T>` 和 `<? super T>` 是Java泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念。

-   `<? extends T>`：是指 “上界通配符（Upper Bounds Wildcards）”。
-   `<? super T>`：是指 “下界通配符（Lower Bounds Wildcards）”。

> 为什么要用通配符和边界？

使用泛型的过程中，经常出现一种很别扭的情况。比如，我们有Fruit类，和它的派生类Apple类。

```java
class Fruit {}
class Apple extends Fruit {}
```

然后有一个最简单的容器：Plate类。盘子里可以放一个泛型的“东西”。我们可以对这个东西做最简单的“放”和“取”的动作：set( )和get( )方法。

```java
class Plate<T>{
    private T item;
    public Plate(T t){item=t;}
    public void set(T t){item=t;}
    public T get(){return item;}
}
```

现在我定义一个“水果盘子”，逻辑上水果盘子当然可以装苹果。

```java
Plate<Fruit> p = new Plate<Apple>(new Apple());
```

但实际上Java编译器不允许这个操作。会报错，“装苹果的盘子”无法转换成“装水果的盘子”。

> error: incompatible types: Plate<Apple> cannot be converted to Plate<Fruit>

所以我的尴尬症就犯了。实际上，编译器脑袋里认定的逻辑是这样的：

-   苹果 IS-A 水果
-   装苹果的盘子 NOT-IS-A 装水果的盘子

所以，就算容器里装的东西之间有继承关系，但容器之间是没有继承关系的。所以我们不可以把 `Plate<Apple>` 的引用传递给 `Plate<Fruit>`。

为了让泛型用起来更舒服，Sun的大脑袋们就想出了 `<? extends T>` 和 `<? super T>` 的办法，来让”水果盘子“和”苹果盘子“之间发生关系。

> 什么是上界？

下面代码就是“上界通配符（Upper Bounds Wildcards）”：

```java
Plate<？ extends Fruit>
```

翻译成人话就是：**一个能放水果以及一切是水果派生类的盘子**。再直白点就是： **啥水果都能放的盘子**。这和我们人类的逻辑就比较接近了。`Plate<？ extends Fruit>` 和 `Plate<Apple>` 最大的区别就是：`Plate<？ extends Fruit>` 是 `Plate<Fruit>` 以及 `Plate<Apple>` 的基类。直接的好处就是，我们可以用“苹果盘子”给“水果盘子”赋值了。

```java
Plate<? extends Fruit> p=new Plate<Apple>(new Apple());
```

如果把Fruit和Apple的例子再扩展一下，食物分成水果和肉类，水果有苹果和香蕉，肉类有猪肉和牛肉，苹果还有两种青苹果和红苹果。

```java
//Lev 1
class Food{}

//Lev 2
class Fruit extends Food{}
class Meat extends Food{}

//Lev 3
class Apple extends Fruit{}
class Banana extends Fruit{}
class Pork extends Meat{}
class Beef extends Meat{}

//Lev 4
class RedApple extends Apple{}
class GreenApple extends Apple{}
```

在这个体系中，上界通配符 “Plate<？ extends Fruit>” 覆盖下图中蓝色的区域。

![](/imgs/java-base/jb-1-1.png)

> 什么是下界？

相对应的，“下界通配符（Lower Bounds Wildcards）”：

```java
Plate<？ super Fruit>
```

表达的就是相反的概念：**一个能放水果以及一切是水果基类的盘子** 。`Plate<？ super Fruit>` 是 `Plate<Fruit>` 的基类，但不是 `Plate<Apple>` 的基类。对应刚才那个例子，`Plate<？ super Fruit>` 覆盖下图中红色的区域。

![](/imgs/java-base/jb-1-2.png)

> 上下界通配符的副作用

边界让Java不同泛型之间的转换更容易了。但不要忘记，这样的转换也有一定的副作用。那就是容器的部分功能可能失效。

还是以刚才的Plate为例。我们可以对盘子做两件事，往盘子里set( )新东西，以及从盘子里get( )东西。

```java
class Plate<T>{
    private T item;
    public Plate(T t){item=t;}
    public void set(T t){item=t;}
    public T get(){return item;}
}
```

> 上界<? extends T>不能往里存，只能往外取

