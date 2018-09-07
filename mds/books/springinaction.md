# Spring IN ACTION

> 源码地址：[Spring-in-Action](https://github.com/jeepchenup/Spring-in-Action)

<details>
<summary>目录</summary>

-   [Chapter 1](#spring-1)
-   [Chapter 2](#spring-2)
-   [Chapter 3](#spring-3)
-   [Chapter 4](#spring-4)

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
    -   [bean 的作用域](#spring-3-4)
    -   [Spring 表达式语言](#spring-3-5)

### <a id="spring-3-1">3.1 Spring profile</a>

> 本章所有的代码都在 [Chapter 3](https://github.com/jeepchenup/Spring-in-Action/tree/master/Chapter3)下。

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

### <a id="spring-3-4">3.4 bean 的作用域</a>

Spring 创建的 bean 对象默认情况下都是 **单例** 的。

什么是单例？就是每次注入或者说创建出来的对象都是同一个。严格意义上面来讲，只有第一次算是创建，后面再用到这个对象的时候只是引用同一个对象而已(有兴趣的可以了解一下[单例模式](/mds/design-model/ds-structure-0.md))。

Spring 定义了多种 bean 的作用域：

1.  Singleton，在整个应用中，只创建 bean 的一个实例。

1.  Prototype，每次注入或者通过 Spring 应用上下文获取的时候，都会创建一个新的 bean 实例。
1.  Session，在 Web 应用中，为每个会话创建一个 bean 实例。
1.  Request，在 Web 应用中，为每个请求创建一个 bean 实例。

#### 如何定义 bean 的 scope？

1.  通过 `@Scope` 注解定义。
    ```java
    @Component
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    // @Scope("singleton") 作用一样
    public class SingletonBean {
        @Override
        public boolean equals(Object obj) {
            System.out.println(this);
            System.out.println(obj);
            return this == obj;
        }
    }
    ```

1.  通过在 XML 文件中定义 bean 的 `scope` 属性值来指定。
    ```java
    <bean id="prototypeBean" scope="prototype" class="com.springinaction.scope.PrototypeBean"/>
    ```

书中介绍了 Singleton, Prototype, Session 这三种使用方式。为了便于测试，这里我就使用了 Singleton 与 Prototype 这两种范围。

### <a id="spring-3-5">3.5 Spring 表达式语言</a>

> 这节会介绍 `org.springframework.core.env.Environment` 这个类的使用场景。

了解 EL 表达式或许会更好理解。它其实就是一个占位符，在程序运行的时候将根据占位符中的 key 来查找相应的 value 当做程序需要的参数注入进去。

无论是 EL 表达式还是 SpringEL 都是需要读取外部文件。

#### 外部注入值

由于，直接赋值的方法不太好，这样做代码灵活性不够高。避免这样的做法，最简单就是通过外部注入的属性值方式。

`@PropertySource`可以将指定的参数设置到 Spring 的 Environment 里面。我们可以通过 `org.springframework.core.env.Environment` 这个对象来获取其中的属性值。

```java
@Configuration
@PropertySource("/runtimeInject.properties")
public class RuntimeInjectConfig {

    @Autowired
    Environment environment;

    @Bean
    public Student student() {
        return new Student(environment.getProperty("name"), environment.getProperty("phone"), environment.getProperty("number"));
    }
}
```

#### @Value 注入

```java
public Teacher(@Value("${name}") String name, @Value("${sex}") String sex) {
    this.name = name;
    this.sex = sex;
}
```

#### 在 XML 文件中设置

```xml
<context:property-placeholder location="runtimeInject.properties"/>

<bean id="student" class="com.springinaction.runtimeInject.Student">
    <constructor-arg index="0" value="${name}"/>
    <constructor-arg index="1" value="${phone}"/>
    <constructor-arg index="2" value="${number}"/>
</bean>
```

#### 使用 Spring EL 进行装配

`#{ <expression string> }` 其实就是通过这个表达式来获取参数。

其特性如下：

1.  使用 bean 的 ID 来引用 bean。
1.  调用方法和访问对象的属性(其实就是第一条特性的衍生)。
1.  对值进行算数、关系和逻辑运算。
1.  正则表达式匹配。
1.  集合操作。

## <a id="spring-4">Chapter4 - 面向切面的 Spring</a>

-   本章内容

    -   [Spring AOP 术语](#spring-4-1)
    -   通过 POJO 创建切面
    -   使用 @AspectJ 注解
    -   为 AspectJ 切面注入依赖

### <a id="spring-4-1">4.1 Spring AOP 术语</a>

<img align="center" src="/imgs/springinaction/cha4-1.png"/>

-   通知(Advice)
    
    切面的工作被称为通知。通知定义了切面是什么以及何时使用。

-   连接点(Join point)

    连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至是修改一个字段时。

-   切点(Poincut)

    如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或者多个连接点。

-   切面(Aspect)

    切面是通知和切点的结合。

-   引入(Introduction)

    引入允许我们向现有的类（在不修改该类的前提下）添加新的方法或者属性。

-   织入(Weaving)

    织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。

-   Advisor

    ![](/imgs/springinaction/cha4-2.png)