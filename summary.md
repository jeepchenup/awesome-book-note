### Spring
- [BeanFactory与FactoryBean之间的区别？](#user-content-sp-1)
- [Spring IoC的理解，其初始化过程？](#user-content-sp-2)

#### <a id="sp-1">BeanFactory与FactoryBean之间的区别？</a>

同：
- 都是接口。
- 都在同一个包下`org.springframework.beans.factory`。

异：
- **FactoryBean**是让IoC容器中的Bean能够更加简单的生成出来。FactoryBean是一个工厂用来创建和返回指定Bean对象。
    
    FactoryBean code
    ```java
    org.springframework.beans.factory.FactoryBean<T>
    org.springframework.beans.factory.FactoryBean.getObject()
    org.springframework.beans.factory.FactoryBean.getObjectType()
    org.springframework.beans.factory.FactoryBean.isSingleton()
    ```

    - FactoryBean Usage Example
    ```java
    package com.wfms.spring.selftest;

    import org.springframework.beans.factory.FactoryBean;

    public class ToolFactory implements FactoryBean<Tool> {
        
        private int factoryId;
        private int toolId;
        
        @Override
        public Tool getObject() throws Exception {
            return new Tool(toolId);
        }

        @Override
        public Class<?> getObjectType() {
            return Tool.class;
        }

        @Override
        public boolean isSingleton() {
            return false;
        }

        public int getFactoryId() {
            return factoryId;
        }

        public int getToolId() {
            return toolId;
        }

        public void setFactoryId(int factoryId) {
            this.factoryId = factoryId;
        }

        public void setToolId(int toolId) {
            this.toolId = toolId;
        }

    }

    package com.wfms.spring.selftest;

    public class Tool {

        private int id;

        public Tool(int id) {
            this.id = id;
        }

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }
        
    }

    //Test
    package com.wfms.spring.selftest;

    import static org.junit.Assert.assertEquals;
    import static org.junit.Assert.assertNotNull;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations="classpath:spring/selftest/factorybean-bean.xml")
    public class FactoryBeanTest {

        @Autowired
        private ToolFactory toolFactory;
        
        @Test
        public void testToolFactoryBean() {
            assertNotNull(toolFactory);
            assertEquals(1, toolFactory.getToolId());
            assertEquals(9090, toolFactory.getFactoryId());
        }
        
        @Test
        public void testToolFactoryBeanCreateByClassPathXmlApplicationContext() {
            String locationPath = "classpath:/spring/selftest/factorybean-bean.xml";
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(locationPath);
            /**
             * 这里通过ClassPathXmlApplicationContext来获取Bean对象
             * 获取的对象并不是ToolFactory对象，而是Tool对象。
             */
            Tool tool = context.getBean("tool", Tool.class);
            
            assertNotNull(tool);
            assertEquals(1, tool.getId());
        }
    }

    ```

    `factorybean-bean.xml`
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


        <bean id="tool" class="com.wfms.spring.selftest.ToolFactory">
            <property name="factoryId" value="9090"/>
            <property name="toolId" value="1"/>
        </bean>
    </beans>
    ```

- **BeanFactory**是IoC最基本的容器，可以通过IoC容器来生产和管理Bean。为其他的IoC容器提供了最基本的规范。

#### 总结
在大多数情况下，我们不会直接使用或实现BeanFactory接口，除非你正在扩展框架的核心功能。当你有需要Spring管理的工厂创建的对象时，你就需要实现FactoryBean。更简洁地说，BeanFactory表示Spring容器，FactoryBean表示工厂类，其创建的对象被获取并作为容器中的bean注册。

参考： 
[geekAbyte](http://www.geekabyte.io/2014/11/difference-between-beanfactory-and.html)

#### <a id="sp-2">Spring IoC的理解，其初始化过程？</a>

- 理解Spring IoC可以从以下三个方面来看：
    1.  IoC是什么？ 
        IoC全称Inversion of Control，即“控制反转”，不是什么技术，而是一种**设计思想**。在开发中，IoC意味着你将设计好的对象交给容器来创建，而不是通过传统的方式来创建。
        - 谁控制谁？控制什么？
        传统创建对象的方式是通过调用`new`这个关键词来进行的，而在IoC中是有个专门的容器来创建这些对象。
        所以，谁控制谁？IoC容器控制了对象。控制什么？控制了外部资源的获取`不只是对象包括比如文件等`。
        - 为什么是反转？哪些方面反转了？
        有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象。  
        为什么是反转？因为由容器帮我们查找以及注入依赖对象，对象只是**被动的接受**依赖对象。  
        哪些方面反转了？依赖对象的获取被反转了。
    
    2.  IoC能做什么？    
        IoC实现了对象与对象之间的**松散耦合**，原来需要程序主动控制去获取依赖对象，转变到让IoC容器来控制。方便测试，利于功能复用，使得程序更加灵活。
        ![IoC实现的转变](/imgs/summary/sp-2-1.png)

    3. IoC和DI是什么关系？
        -   DI是什么？  
            DI，依赖注入。组件之间依赖关系由容器在运行期决定，即由容器动态的将某个**组件所依赖的对象**注入到组件中去，这个对象可能是另外一个组件也可能是POJO。
        -   理解DI的关键是：“谁依赖谁，为什么需要依赖？谁注入谁，注入了什么？”
            -   谁依赖谁：应用程序依赖IoC容器
            -   为什么需要依赖：应用程序需要IoC容器来提供对象所需的外部资源
            -   谁注入谁：IoC容器将应用程序所需的外部资源注入进去
            -   注入了什么：应用程序对象所需的外部资源

- Spring IoC的初始化过程？
    1.  Resource定位（Bean的定义文件定位）
    
    2.  将Resource定位好的资源载入到BeanDefinition
    3.  将BeanDefiniton注册到容器中

参考：
-   [jinnianshilongnian](http://jinnianshilongnian.iteye.com/blog/1413846)
-   [有爱jj](https://www.cnblogs.com/chenjunjie12321/p/6124649.html)
-   [Spring IoC容器初始化过程](https://www.jianshu.com/p/d5f1670c3c0f)