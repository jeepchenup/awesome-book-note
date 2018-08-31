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

> 本章会涉及如何声明 Bean，如何扫描 Bean，如何装配 Bean。默认情况下，Spring 中的 bean 都是单例。

### 1. 声明 Bean

> 作者推荐优先使用 **隐式声明**，再者 **JavaConfig**，最后才是 **XML** 的方式。

-   隐式的声明，通过 `@Component` 注解让 Spring 自动发现。这个简单的注解表明该类会作为组件类，并告知 Spring 要为这个类创建 bean。

    ![](/imgs/springinaction/cha2-2.png '只能用来修饰class')
    > 由于组件扫描默认是不启动的，需要手动启动。

    1.  可以通过 @ComponentScan 注解来开启扫描。

        ![](/imgs/springinaction/cha2-3.png)

    1.  可以通过 XML 配置文件来开启扫描。

        ![](/imgs/springinaction/cha2-4.png)

    这两种方式也是 Spring 中最常用的方式。

-   在 JavaConfig 配置中，通过 `@Bean` 注解来声明 bean。

    @Bean 注解会告诉 Spring 注释的 **方法** 将会返回一个对象，该对象要注册为 Spring 应用上下文中的 bean。

    ![](/imgs/springinaction/cha2-5.png)

-   在 xml 文件中，通过 `<bean>` 标签来声明 bean。

    ![](/imgs/springinaction/cha2-6.png)

### 2. Bean 装配方式
    
通过 @Autowired 注释来实现。`@Autowired` 可以将声明的 bean 注入，无论是上述那种方式声明出来 bean 都可以注入。

![](/imgs/springinaction/cha2-1.png)

> 隐式装配 bean是最为推荐的方式。但是有时候，比不可免的要使用 xml 或 JavaConfig 的方式来进行配置 bean。因为比如，你想要将 **第三方库** 中的组件装配到你的应用中，这个时候隐式的装配就不行了。

## 总结

bean 的三种配置：

1. 隐式配置：@Component 配合 @ComponentSan 进行扫描。

1. JavaConfig 显示配置，@Bean 在 @Configuration 修饰的配置类中进行声明 bean 的声明。

1. XML 显示配置，bean 的声明就是在 XML 文件中通过 `<bean>` 标签来声明。