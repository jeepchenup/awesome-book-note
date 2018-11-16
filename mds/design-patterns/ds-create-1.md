# <a id="ds-1">工厂方法</a>
### 意图

> 如果简单工厂能够满足需求就不会有工厂方法了。之所以引入工厂方法模式，是因为我们往往需要使用两个或者两个以上的工厂。

定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法把实例化推迟到子类。

### 类图
**在简单工厂中，创建对象的是另一个类，而在工厂方法中，是由子类来创建对象。**

下图中，Factory 有一个 doSomethind() 方法，这个方法需要用到一个产品对象，这个产品对象由 factoryMethod() 方法创建。该方法是抽象的，需要由子类去实现。

![](../../imgs/summary/ds-1.png)  

### 工厂方法的实现

```java
public abstract class Factory {
    abstract public Product factoryMethod();
    public void doSomething() {
        Product product = factoryMethod();
        // do something with the product
    }
}

public class ConcreteFactory extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct();
    }
}

public class ConcreteFactory1 extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct1();
    }
}

public class ConcreteFactory2 extends Factory {
    public Product factoryMethod() {
        return new ConcreteProduct2();
    }
}

public class Client {
    public static void main(String[] args) {
        // 先决定用哪个工厂来创建产品
        ConcreteFactory1 factory = new ConcreteFactory1();
        // 再通过指定的产品做一些事情
        factory.doSomething();
    }
}
```

### 总结

相比较简单工厂，工厂模式的核心就是需要在第一步选好我们需要的工厂。第二步跟简单工厂一样，通过参数来决定生产什么产品。

##  [BACK](../../mds/summary.md)