# Spring IN ACTION

> 源码地址：[Spring-in-Action](https://github.com/jeepchenup/Spring-in-Action)

<details>
<summary>目录</summary>

-   [Chapter 1](#spring-1)
-   [Chapter 2](#spring-2)
-   [Chapter 3](#spring-3)

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

## <a id="spring-3">Chapter 3 - 高级装配</a>

-   本章的内容

    -   [Spring profile](#spring-3-1)
    -   [条件化的 bean 声明](#spring-3-2)
    -   [自动装配与歧义性](#spring-3-3)
    -   bean 的作用域
    -   Spring 表达式语言

### <a id="spring-3-1">3.1 Spring profile</a>

Spring profile 就是为了简化项目迁移流程和降低迁移项目带来的人工成本。

Spring 提供的 profile 将本地开发环境、测试环境以及生产环境都有效的划分开来。我们可以通过简单的配置就可以让项目在相应的环境中运行。

有两种配置 profile 的方法：

1.  使用 `@Profile` 注解，表明这个类是配置类，需要在指定的环境下才会激活。

    ![](/imgs/springinaction/cha3-1.png)

2.  在 XML 配置文件中通过设置 **&lt;beans&gt;** 中的 `profile` 属性来指定其运行的环境。

    ![](/imgs/springinaction/cha3-2.png)

激活某个profile 

文中提供了下面几种方式：

1.  作为 DispatcherServlet 的初始化参数
1.  作为 Web 应用的上下文参数
1.  作为 JNDI 条目
1.  作为环境变量
1.  作为 JVM 的系统属性
1.  在集成测试类上，使用 @ActiveProfiles 注解设置

这里图个方便，就选用最后一个方式来验证 Spring profile 的功能。

### <a id="spring-3-2">3.2 条件化的 bean 声明</a>

条件化的 bean 声明：顾名思义，只有在满足了一定的条件时，bean 对象才会被创建出来。

#### 如何来设置条件？

可以通过 `@Conditional` 注解来实现。

**com.sprintinaction.conditionbean.ConditionBeanConfig.Class**

```java
@Configuration
@PropertySource("/conditionbean.properties")
public class ConditionBeanConfig {

    @Bean
    @Conditional(MagicExistCondition.class)
    // Conditional 会去执行 MagicExistCondition 中的 match 方法
    // 根据其返回的 boolean 值来确定是否创建 bean 对象
    public MagicBean magicBean() {
        return new MagicBean();
    }
}
```

这里还需要注意一点，凡是在 `@Conditional` 中添加的类都需要继承 `org.springframework.context.annotation.Condition` 这个接口。

```java
public interface Condition {

	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

> 注意：书中只给了如何判断当前运行环境中是否存在 'magic' 属性，却没有告诉我们如何来模拟设置。

我的做法是在 JavaConfig 中使用 `@PropertySource("/conditionbean.properties")` 来引入一个配置文件，其内容是简单的 **magic=magic**。这样测试用例就能成功通过了。

### <a id="spring-3-3">3.3 自动装配与歧义性</a>

作者提供给我们一个例子：

```java
@Autowired
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}

@Component
public class Cake implements Dessert { ... }
@Component
public class Cookies implements Dessert { ... }
@Component
public class IceCream implements Dessert { ... }
```

像上面的情况下，Spring 是无法自动装载 bean 对象的。因为每个 bean 都是继承了 Dessert 这个接口，每个都符合，Spring 反而没有主意了。这就是自动装配可能存在的歧义性。

### 如何避免出现自动装载的歧义性？

书中提供了 2 种解决方案：

1.  `@Primary` 或者 `<bean primary="true">`，出现相同的 bean 时，优先注入 `@Primary` 修饰的 bean。
1.  `@Qualifier`，直接限定要注入的 bean。

简单一点理解就是，`@Primary` 是让 Spring 自己选；`@Qualifier` 是你替 Spring 做了决定。