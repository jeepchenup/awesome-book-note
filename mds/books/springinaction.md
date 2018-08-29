# Spring IN ACTION

> 源码地址：[Spring-in-Action](https://github.com/jeepchenup/Spring-in-Action)

<details>
<summary>目录</summary>

-   [Chapter 1](#spring-1)
-   [Chapter 2](#spring-2)

</details>

## <a id="spring-1">Chapter 1 - 简化 Java 开发</a>

> 这章节主要介绍了 Spring 为 Java 开发带来了哪些便利之处。

-   依赖注入所带来的最大收益就是 —— 松耦合。
-   AOP 能够确保 POJO 的简单性。
-   Spring 通过模板来消除 **样板式** 代码。

### 总结

Spring 框架关注于通过 DI、AOP 和消除样板式代码来简化企业级 Java 开发。

## <a id="spring-2">Chapter 2 - 装配 Bean</a>

> 本章会涉及如何声明 Bean，如何扫描 Bean

### 3 种装配 Bean 的方式

1.  在 XML 中进行显示配置

1.  在 Java 中进行显示配置

1.  隐式的 bean 自动装配
    
    通过 @Component, @Autowired, @ComponentScan 来配合实现。

### Bean 的 2 种扫描方法

1.  组件扫描：Spring 会自动发现应用上下文中所创建的bean。

    ![](/imgs/springinaction/cha2-1.png)

1.  自动装配：Spring 自动满足bean之间的依赖关系。

    ![](/imgs/springinaction/cha2-2.png "默认会扫描与配置类相同的包")

    @ComponentSan 会去扫描带有 @Component 注解的类。