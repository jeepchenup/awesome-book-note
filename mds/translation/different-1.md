#   JAX-RS vs Spring for REST Endpoints

>   2018-10-8   
>   [原文地址](https://stormpath.com/blog/jax-rs-vs-spring-rest-endpoints)

REST 端通常是被用在将 web 服务端和客户端解耦的地方。许多开发者通过 Spring 或者 JAX-RS 来达到这个目的。有些人只使用其中的一种方式来实现 REST 而不是另一种。在这篇文章中，我将使用基本相同的代码来辨别两者之间的区别。

**JAX-RS** 全称 Java API for RESTful Web Services，是一个 Java 编程语言API 规范，它支持根据具象状态传输(Representational State Transfer, REST)体系结构模式创建 Web 服务。这是 Java EE 6 的一部分，但是可以很容易地在一个简单的 servlet 容器中使用。它是专门为简化 REST 资源编写而创建的。

**Spring Framework** 以及其支持库的范围显然要比创建一些 REST 端要大的多，但是那些情况是针对于不同的 post 情况。今天我们来构建一些简单的 CRUD 应用，通过 [Jackson库](http://wiki.fasterxml.com/JacksonHome) 的 JSON 来管理 Stormtroopers。

## Lay Down the Foundation - Model and DAO

为了集中注意力，我将把 Maven 的依赖关系排除在本文之外。你可以在 [Github](http://github.com/stormpath/jaxrs-spring-blog-example) 上浏览完整的源代码。

首先，我们需要去掉一些常见的部分。所有的例子都将使用一个简单的 model 和DAO 来注册和管理 `Stormtrooper` 对象。

```java
public class Stormtrooper {
 
    private String id;
    private String planetOfOrigin;
    private String species;
    private String type;
 
    public Stormtrooper() {
        // empty to allow for bean access
    }
 
    public Stormtrooper(String id, String planetOfOrigin, String species, String type) {
        this.id = id;
        this.planetOfOrigin = planetOfOrigin;
        this.species = species;
        this.type = type;
    }
 
    ...
    // bean accessor methods
```

Stormtrooper 有四个成员变量：id, planetOfOrigin, species 和 type。

DAO 层接口也很简单，提供了基础地 CRUD 方法和一个额外的 list 方法：

```java
public interface StormtrooperDao {
 
    Stormtrooper getStormtrooper(String id);
 
    Stormtrooper addStormtrooper(Stormtrooper stormtrooper);
 
    Stormtrooper updateStormtrooper(String id, Stormtrooper stormtrooper);
 
    boolean deleteStormtrooper(String id);
 
    Collection<Stormtrooper> listStormtroopers();
}
```

对于这些例子，StormtrooperDao 的实际实现并不是那么重要。如果你敢兴趣，你可以看一下这个类 - DefaultStormtrooperDao，它随机生成 50 个 Stormtroopers。

## Spring

现在，我们已经解决了常见的问题，现在我们可以进入 Spring 示例的核心部分了。 没有一个能比 Spring boot 更加简单的配置一个 Spring 应用了。

```java
@SpringBootApplication
public class SpringBootApp {
 
    @Bean
    protected StormtrooperDao stormtrooperDao() {
        return new DefaultStormtrooperDao();
    }
 
    public static void main(String[] args) {
        SpringApplication.run(SpringBootApp.class, args);
    }
}
```

有几点需要指出：

1.  @SpringBootApplication 注释设置了 Spring 组件的自动配置和类路径扫描。

1.  @Bean 将 DefaultStormtrooperDao 对象作为 StormtrooperDao 的实例注册进去。

1.  `main` 静态方法用来启动 Spring 应用。

## Spring Controller

接下来，我们来实现 REST 端或者是一个 Controller。

```java
@RestController
@RequestMapping("/troopers")
public class StormtroooperController {
 
    private final StormtrooperDao trooperDao;
 
    @Autowired
    public StormtrooperController(StormtrooperDao trooperDao) {
        this.trooperDao = trooperDao;
    }
 
    @RequestMapping(path = "/{id}", method = RequestMethod.GET)
    public Stormtrooper getTrooper(@PathVariable("id") String id) throws NotFoundException {
 
        Stormtrooper stormtrooper = trooperDao.getStormtrooper(id);
        if (stormtrooper == null) {
            throw new NotFoundException();
        }
        return stormtrooper;
    }
 
    @RequestMapping(method = RequestMethod.POST)
    public Stormtrooper createTrooper(@RequestBody Stormtrooper trooper) {
        return trooperDao.addStormtrooper(trooper);
    }
 
    @RequestMapping(path = "/{id}", method = RequestMethod.POST)
    public Stormtrooper updateTrooper(@PathVariable("id") String id, @RequestBody Stormtrooper updatedTrooper) throws NotFoundException {
        return trooperDao.updateStormtrooper(id, updatedTrooper);
    }
 
 
    @RequestMapping(path = "/{id}", method = RequestMethod.DELETE)
    @ResponseStatus(value = HttpStatus.NO_CONTENT)
    public void deleteTrooper(@PathVariable("id") String id) {
        trooperDao.deleteStormtrooper(id);
    }
 
    @RequestMapping(method = RequestMethod.GET)
    public Collection<Stormtrooper> listTroopers() {
        return trooperDao.listStormtroopers();
    }
}
```

下面我们逐一分析上述代码：

```java
@RestController
@RequestMapping("/troopers")
public class StormtroooperController {
```

对于 `@Controller` 和 `@ResponseBody` 来说，`@RestController` 是一个方便的注解。被 `@RequestMapping` 修饰的类在 classpath 扫描期间会被标记为 web 组件。类级别的 `@RequestMapping` 用来定义该类任意 RequestMapping 的基础映射。在本例中，本类所有的端点开始的 URL 地址是 `/troopers`。

```java
@RequestMapping(path = "/{id}", method = RequestMethod.POST)
public Stormtrooper updateTrooper(@PathVariable("id") String id, @RequestBody Stormtrooper updatedTrooper) throws NotFoundException {
    return trooperDao.updateStormtrooper(id, updatedTrooper);
}
```

`@RequestMapping` 注解有很多选择，这里只是使用其中的一部分功能：

-   `@PathVariable("id")` 对应着 `path = "/{id}"`。将给定的方法参数传递到 URL 路径的 `{id}` 部分 - 示例 URL: `/troopers/FN-2187`。
-   `method=RequestMethod.GET` 指定请求方式为 get
-   `value = HttpStatus.NO_CONTENT` 设置了预期 HTTP 响应状态码为 204。

HTTP 请求传递过来的参数将被反序列化后，再传递到用 `@RequestBody` 注解的方法参数。返回值 `return trooperDao.updateStormtrooper(id, updatedTrooper);` 是直接通过 `@RquestBody` 注解（或简单实用 `@RestController`）序列化结果，接着响应到 HTTP 上面。`@RquestBody` 和 `@RestController` 将绕过任何 MVC 模板。

```java
@RequestMapping(path = "/{id}", method = RequestMethod.POST)
public Stormtrooper updateTrooper(@PathVariable("id") String id, @RequestBody Stormtrooper updatedTrooper) throws NotFoundException {
    return trooperDao.updateStormtrooper(id, updatedTrooper);
}
```

在代码块中，`updateTrooper()` 这个方法支持来自 `/trooper/{id}` 发出的 POST HTTP 请求，并包括一个序列化的 `Stormtrooper(JSON)`。如果请求路径是 `/troopers/FN-2187`，那么 POST 请求路径中 id 部分的值就是 `FN-2187` 并且传递给方法中的 id 参数。最后将一个更新后的 `Stormtrooper` 对象返回，并且将其序列化到 HTTP 相应中。

## Run the Spring Example

要运行这个例子，请先获取[资源](https://github.com/stormpath/jaxrs-spring-blog-example)，并且改变 Spring boot 的目录，运行 `mvn spring-boot:run` 指定启动应用，最后向服务器发送请求。

>   拿到所有 Stormtrooper 的集合

```txt
$ curl http://localhost:8080/troopers
 
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Tue, 08 Nov 2016 20:33:36 GMT
Transfer-Encoding: chunked
X-Application-Context: application
 
[
    {
        "id": "FN-2187",
        "planetOfOrigin": "Unknown",
        "species": "Human",
        "type": "Basic"
    },
    {
        "id": "FN-0984",
        "planetOfOrigin": "Coruscant",
        "species": "Human",
        "type": "Aquatic"
    },
    {
        "id": "FN-1253",
        "planetOfOrigin": "Tatooine",
        "species": "Unidentified",
        "type": "Sand"
    },
    ...
] 
```

根据 id 获取对应的 Stormtrooper 的对象。

```txt
$ curl http://localhost:8080/troopers/FN-2187
 
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Date: Tue, 08 Nov 2016 20:38:53 GMT
Transfer-Encoding: chunked
X-Application-Context: application
 
{
    "id": "FN-2187",
    "planetOfOrigin": "Unknown",
    "species": "Human",
    "type": "Basic"
}
```