## Spring

---

###  1. 列举一些重要的 Spring 模块

==参考答案==



<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/jvme0c60b4606711fc4a0b6faf03230247a.png" alt="Spring主要模块" style="zoom: 67%;" />

####  2.1 Spring Core

核心模块， Spring 其他所有的功能基本都需要依赖于该模块，主要提供 IoC 依赖注入功能的支持。

#### 2.2 Spring Aspects

该模块为与 AspectJ 的集成提供支持。

#### 2.3 Spring AOP

提供了面向切面的编程实现。

#### 2.4 Spring Data Access/Integration 

Spring Data Access/Integration 由 5 个模块组成：

- spring-jdbc : 提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响。
- spring-transactions: 提供对事务的支持。
- spring-orm : 提供对 Hibernate 等 ORM 框架的支持。
- spring-oxm ： 提供对 Castor 等 OXM 框架的支持。
- spring-jms : Java 消息服务。

#### 2.5 Spring Web

Spring Web 由 4 个模块组成：

- spring-web ：对 Web 功能的实现提供一些最基础的支持。
- spring-webmvc ： 提供对 Spring MVC 的实现。
- spring-websocket ： 提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
- spring-webflux ：提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步.

#### 2.6 Spring Test

Spring 团队提倡测试驱动开发（TDD）。有了控制反转 (IoC)的帮助，单元测试和集成测试变得更简单。

Spring 的测试模块对 JUnit（单元测试框架）、TestNG（类似 JUnit）、Mockito（主要用来 Mock 对象）、PowerMock（解决 Mockito 的问题比如无法模拟 final, static， private 方法）等等常用的测试框架支持的都比较好。



### 2. 请你说说Spring的核心是什么

==参考答案==

IoC和AOP是Spring框架的核心。

IoC（Inversion of Control）是控制反转，这是一种面向对象编程的设计思想*（类似于点外卖）*。创建被调用者的实例不是由调用者完成，而是由Spring 容器完成，并注入调用者。一个对象依赖的其它对象会自动传递进来，而不是这个对象自己创建或者查找依赖对象。即，不是调用者从容器中查找依赖对象，而是容器创建依赖对象后主动将其传递给它。

说到IoC就不得不说DI（Dependency Injection），DI是依赖注入，它是IoC实现的实现方式。调用者不需要从容器中查找依赖对象，而是容器创建依赖对象后主动将其传递给它。**依赖注入是目前最优秀的解耦方式。依赖注入让 Spring 的 Bean 之间以配置文件的方式组织在一起，而不是以硬编码的方式耦合在一起的。**



AOP（Aspect Oriented Programing）是面向切面编程思想，是对OOP的补充，它可以在OOP的基础上进一步提高编程的效率。能够将那些与业务无关，却被业务模块所共同调用的业务逻辑（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。



### 3. 说一说你对Spring容器的了解

==参考答案==

Spring主要提供了两种类型的容器：BeanFactory和ApplicationContext。

- BeanFactory：是基础类型的IoC容器，采用延迟加载策略，即在第一次调用**`getBean()`**时，才真正装配该对象。所以，相对来说，容器启动速度较快，所需要的资源较少。**适用于资源有限，并且功能要求不是很严格的场景。**
- ApplicationContext：它是在BeanFactory的基础上构建的，是相对比较高级的容器实现，除了拥有BeanFactory的所有支持，还提供了其他高级特性。ApplicationContext 容器，会在容器对象初始化时，将其中的所有对象一次性全部装配好，以后若要使用到这些对象，只需从内存中直接获取即可，执行效率较高，但需要占用较多的内存。**适用于在系统资源充足，并且要求更多功能的场景中。**

### 4. DI注入的两种方式

==参考答案==

- **构造方法注入**

就是被注入对象可以在它的构造方法中声明依赖对象的参数列表，让外部知道它需要哪些依赖对象。在构造调用者实例的同时，完成被调用者的实例化。即，使用构造器设置依赖关系。

```xml
   <bean id="bookDao" class="com.dao.impl.BookDaoImpl" init-method="init" destroy-method="destroy">
<!--        构造器注入（基本类型）-->
        <constructor-arg name="connectionNum" value="100" />
        <constructor-arg name="databaseName" value="mysql" />
   </bean>
   <bean id="userDao" class="com.dao.impl.UserDaoImpl" />
   <bean id="bookService" class="com.service.impl.BookServiceImpl">
<!--        构造器注入（引用类型）-->
        <constructor-arg name="bookDao" ref="bookDao" />
        <constructor-arg name="userDao" ref="userDao" />
   </bean>
```

```java
    private BookDao bookDao;
    private UserDao userDao;  
    private int connectionNum;
    private String databaseName;
   // 构造器注入
    public BookServiceImpl(BookDao bookDao, UserDao userDao) {
        this.bookDao = bookDao;
        this.userDao = userDao;
    }
 
    public BookDaoImpl(int connectionNum, String databaseName){
        this.databaseName = databaseName;
        this.connectionNum = connectionNum;
    }
```

`<constructor-arg />`标签中用于指定参数的属性有： 

name：指定参数名称。 

*index：指明该参数对应着构造器的第几个参数，从 0 开始。若参数类型相同，或之间有包含关系，则需要保证赋值顺序要与构造器中的参数顺序一致。*

*type 属性可用于指定其类型。基本类型直接写类型关键字即可，非基本类型需要写全限定性类名。*



- **setter方法注入**

通过setter方法，可以更改相应的对象属性。所以，当前对象只要为其依赖对象所对应的属性添加setter方法，就可以通过setter方法将相应的依赖对象设置到被注入对象中。

```java
    private BookDao bookDao;
    private UserDao userDao;
    private int connectionNum;
    private String databaseName;
    // setter注入
    public void setBookDao(BookDao bookDao){
        this.bookDao = bookDao;
    }

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void setConnectionNum(int connectionNum) {
        this.connectionNum = connectionNum;
    }

    public void setDatabaseName(String databaseName) {
        this.databaseName = databaseName;
    }
```

```xml
   <bean id="bookDao" class="com.dao.impl.BookDaoImpl" init-method="init" destroy-method="destroy">
<!--        1.setter注入（基本类型）-->
        <property name="databaseName" value="mysql" />
        <property name="connectionNum" value="100" />
   </bean>
   <bean id="userDao" class="com.dao.impl.UserDaoImpl" />
   <bean id="bookService" class="com.service.impl.BookServiceImpl">
<!--        1. setter注入（引用类型）-->
        <property name="bookDao" ref="bookDao" />
        <property name="userDao" ref="userDao" />
   </bean>
```

### 5. 什么是Spring Bean？

==参考答案==

简单来说，Bean 代指的就是那些被 IoC 容器所管理的对象。

我们需要告诉 IoC 容器帮助我们管理哪些对象，这个是通过配置元数据来定义的。配置元数据可以是 XML 文件、注解或者 Java 配置类。

![img](https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/062b422bd7ac4d53afd28fb74b2bc94d.png)



### 6. 将一个类声明为 Bean 的注解有哪些?

==参考答案==

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。

- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。

- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。

- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

  

### 7. @Component 和 @Bean 的区别是什么？

==参考答案==

- `@Component` 注解作用于类，而`@Bean`注解作用于方法。
- `@Component`通常是通过类路径扫描来自动检测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径，从中找出标识了需要装配的类，并自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是在标有该注解的方法中定义产生这个 bean（  设置该方法的返回值作为spring管理的bean）。
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

```java
@Configuration
public class JdbcConfig {
    @Bean
    public DataSource dataSource(BookDao bookDao){
        System.out.println(bookDao);
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driverClassName);
        ds.setUrl(url);
        ds.setUsername(username);
        ds.setPassword(password);
        return ds;
    }
}
```

下面这个例子是通过 `@Component` 无法实现的。

```java
@Bean
public OneService getService(status) {
    switch (status)  {
        case 1:
        	return new serviceImpl1();
        case 2:
            return new serviceImpl2();
        case 3:
            return new serviceImpl3();
    }
}
```

### 8. Spring是如何管理Bean的？

==参考答案==

Spring通过IoC容器来管理Bean，我们可以通过XML配置或者注解配置，来指导IoC容器对Bean的管理。因为注解配置比XML配置方便很多，所以现在大多时候会使用注解配置的方式。

以下是管理Bean时常用的一些注解：

1. @ComponentScan用于声明扫描策略，通过它的声明，容器就知道要扫描哪些包下带有声明的类，也可以知道哪些特定的类是被排除在外的。

2. @Component、@Repository、@Service、@Controller用于声明Bean，它们的作用一样，但是语义不同。@Component用于声明通用的Bean，@Repository用于声明DAO层的Bean，@Service用于声明业务层的Bean，@Controller用于声明视图层的控制器Bean，被这些注解声明的类就可以被容器扫描并创建。

3. @Autowired、@Qualifier用于注入Bean，即告诉容器应该为当前属性注入哪个Bean。其中，@Autowired是按照Bean的类型进行匹配的，如果这个属性的类型具有多个Bean，就可以通过@Qualifier指定Bean的名称，以消除歧义。

4. @Scope用于声明Bean的作用域，默认情况下Bean是单例的，即在整个容器中这个类型只有一个实例。可以通过@Scope注解指定prototype值将其声明为多例的，也可以将Bean声明为session级作用域、request级作用域等等，但最常用的还是默认的单例模式。

5. @PostConstruct、@PreDestroy用于声明Bean的生命周期。其中，被@PostConstruct修饰的方法将在Bean实例化后被调用，@PreDestroy修饰的方法将在容器销毁前被调用。

   

### 9. 介绍Bean的作用域

==参考答案==

默认情况下，Bean在Spring容器中是单例的，我们可以通过@Scope注解修改Bean的作用域。该注解有如下5个取值，它们代表了Bean的5种不同类型的作用域：

| 类型          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| singleton     | 在Spring容器中仅存在一个实例，即Bean以单例的形式存在。       |
| prototype     | 每次调用getBean()时，都会执行new操作，返回一个新的实例。     |
| request       | 每次HTTP请求都会创建一个新的Bean。                           |
| session       | 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean。 |
| globalSession | 同一个全局的Session共享一个Bean，一般用于Portlet环境。       |

### 10. Bean 的生命周期了解么?

==参考答案==

> 下面的内容整理自：[https://yemengying.com/2016/07/14/spring-bean-life-cycle/open in new window](https://yemengying.com/2016/07/14/spring-bean-life-cycle/) ，除了这篇文章，再推荐一篇很不错的文章 ：[https://www.cnblogs.com/zrtqsk/p/3735273.htmlopen in new window](https://www.cnblogs.com/zrtqsk/p/3735273.html) 。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/181453414212066.png" alt="img"  />

![img](https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/181454040628981.png)

- Bean 容器找到配置文件中 Spring Bean 的定义。
- Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值 利用 `set()`方法设置一些属性值。
- 如果 Bean 实现了 `BeanNameAware` 接口，调用 `setBeanName()`方法，传入 Bean 的名字。
- 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 `ClassLoader`对象的实例。
- 如果 Bean 实现了 `BeanFactoryAware` 接口，调用 `setBeanFactory()`方法，传入 `BeanFactory`对象的实例。
- 与上面的类似，如果实现了其他 `*.Aware`接口，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization()` 方法
- 如果 Bean 实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessAfterInitialization()` 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![img](https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/7EF8F66C3DFA7434E4CA11B47CF8F1F7)



### 11. 注入 Bean 的注解有哪些？

==参考答案==

Spring 内置的 `@Autowired` 以及 JDK 内置的 `@Resource` 和 `@Inject` 都可以用于注入 Bean。

| Annotaion    | Package                            | Source       |
| ------------ | ---------------------------------- | ------------ |
| `@Autowired` | `org.springframework.bean.factory` | Spring 2.5+  |
| `@Resource`  | `javax.annotation`                 | Java JSR-250 |
| `@Inject`    | `javax.inject`                     | Java JSR-330 |

`@Autowired` 和`@Resource`使用的比较多一些。



### 12. `@Autowired` 和 `@Resource` 的区别是什么？

==参考答案==

`Autowired` 属于 Spring 内置的注解，默认的注入方式为`byType`（根据类型进行匹配），也就是说会优先根据接口类型去匹配并注入 Bean （接口的实现类）。

**这会有什么问题呢？** 当一个接口存在多个实现类的话，`byType`这种方式就无法正确注入对象了，因为这个时候 Spring 会同时找到多个满足条件的选择，默认情况下它自己不知道选择哪一个。

这种情况下，注入方式会变为 `byName`（根据名称进行匹配），这个名称通常就是类名（首字母小写）。就比如说下面代码中的 `smsService` 就是我这里所说的名称，这样应该比较好理解了吧。

```java
// smsService 就是我们上面所说的名称
@Autowired
private SmsService smsService;
```

举个例子，`SmsService` 接口有两个实现类: `SmsServiceImpl1`和 `SmsServiceImpl2`，且它们都已经被 Spring 容器所管理。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Autowired
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Autowired
private SmsService smsServiceImpl1;
// 正确注入  SmsServiceImpl1 对象对应的 bean
// smsServiceImpl1 就是我们上面所说的名称
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;
```

我们还是建议通过 `@Qualifier` 注解来显示指定名称而不是依赖变量的名称。

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

`@Resource` 有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。

```java
public @interface Resource {
    String name() default "";
    Class<?> type() default Object.class;
}
```

如果仅指定 `name` 属性则注入方式为`byName`，如果仅指定`type`属性则注入方式为`byType`，如果同时指定`name` 和`type`属性（不建议这么做）则注入方式为`byType`+`byName`。

```java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;
```

简单总结一下：

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。
- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。
- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显示指定名称，`@Resource`可以通过 `name` 属性来显示指定名称。



### 13. 单例 Bean 的线程安全问题了解吗？

==参考答案==

单例 Bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

常见的有两种解决办法：

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，大部分 Bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。



### 14. Spring中默认提供的单例是线程安全的吗？

==参考答案==

不是。

Spring容器本身并没有提供Bean的线程安全策略。如果单例的Bean是一个无状态的Bean，即线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例的Bean是线程安全的。比如，Controller、Service、Dao这样的组件，通常都是单例且线程安全的。如果单例的Bean是一个有状态的Bean，则可以采用ThreadLocal对状态数据做线程隔离，来保证线程安全。



### 15.  说一说你对Spring AOP的理解

==参考答案==

AOP（Aspect Oriented Programming）是面向切面编程，它是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。所谓切面，相当于应用对象间的横切点，我们可以将其单独抽象为单独的模块。

1. #### **AOP的术语：**

- 切面（Aspect）：切面泛指交叉业务逻辑。事务处理、日志处理就可以理解为切面。常用的切面
- 织入（Weaving）：它是一个通过动态代理技术，为原有服务对象生成代理对象，然后将与切点定义匹配的连接点拦截，并按约定将各类通知织入约定流程的过程。

- 连接点（JoinPoint）：对应的是具体被拦截的对象，因为Spring只能支持方法，所以被拦截的对象往往就是指特定的方法，AOP将通过动态代理技术把它织入对应的流程中。
- 切点（Pointcut）：有时候，我们的切面不单单应用于单个方法，也可能是多个类的不同方法，这时，可以通过正则式和指示器的规则去定义，从而适配连接点。切点就是提供这样一个功能的概念。
- 通知（advice）：就是按照约定的流程下的方法，分为前置通知、后置通知、环绕通知、事后返回通知和异常通知，它会根据约定织入流程中。
- 目标对象（target）：即被代理对象。
- 引入（introduction）：是指引入新的类和其方法，增强现有Bean的功能。

2. #### **Spring AOP：**

AOP可以有多种实现方式，而Spring AOP基于动态代理来实现，默认如果使用接口的，用JDK提供的动态代理实现，如果是方法则使用CGLIB实现。

- JDK动态代理：这是Java提供的动态代理技术，可以在运行时创建接口的代理实例。Spring AOP默认采用这种方式，在接口的代理实例中织入代码。
- CGLib动态代理：采用底层的字节码技术，在运行时创建子类代理的实例。当目标对象不存在接口时，Spring AOP就会采用这种方式，在子类实例中织入代码。

3. #### **Spring AOP 和 AspectJ AOP 有什么区别？**

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** 

Spring AOP 基于动态代理(Proxying)来实现，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，如果切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比 Spring AOP 快很多。

>*面试官：什么是AOP？Spring AOP和AspectJ的区别是什么？https://segmentfault.com/a/1190000022019122*



### 16. 既然有没有接口都可以用CGLIB，为什么Spring还要使用JDK动态代理？

==参考答案==

在性能方面，CGLib创建的代理对象比JDK动态代理创建的代理对象高很多。但是，CGLib在创建代理对象时所花费的时间比JDK动态代理多很多。所以，对于单例的对象因为无需频繁创建代理对象，采用CGLib动态代理比较合适。反之，对于多例的对象因为需要频繁的创建代理对象，则JDK动态代理更合适。



### 17. 请你说说AOP的应用场景

==参考答案==

Spring AOP为IoC的使用提供了更多的便利，

一方面，应用可以直接使用AOP的功能，设计应用的切入点，把交叉业务逻辑多个模块的功能抽象出来，并通过简单的AOP的使用，灵活地编制到模块中，比如可以通过AOP实现应用程序中的日志功能。

另一方面，在Spring内部，一些支持模块也是通过Spring AOP来实现的，比如事务处理。从这两个角度就已经可以看到Spring AOP的核心地位了。



### 18. Spring AOP不能对哪些类进行增强？

==参考答案==

1. Spring AOP只能对IoC容器中的Bean进行增强，对于不受容器管理的对象不能增强。

2. 由于CGLib采用动态创建子类的方式生成代理对象，所以不能对final修饰的类进行代理。

   

### 19. Spring如何管理事务？

==参考答案==

Spring为事务管理提供了一致的编程模板，在高层次上建立了统一的事务抽象。也就是说，不管是选择MyBatis、Hibernate、JPA还是Spring JDBC，Spring都可以让用户以统一的编程模型进行事务管理。

Spring支持两种事务编程模型：

1. 编程式事务

   Spring提供了TransactionTemplate模板，利用该模板我们可以通过编程的方式实现事务管理，而无需关注资源获取、复用、释放、事务同步及异常处理等操作。相对于声明式事务来说，这种方式相对麻烦一些，但是好在更为灵活，我们可以将事务管理的范围控制的更为精确。

2. 声明式事务

   Spring事务管理的亮点在于声明式事务管理，它允许我们通过声明的方式，在IoC配置中指定事务的边界和事务属性，Spring会自动在指定的事务边界上应用事务属性。相对于编程式事务来说，这种方式十分的方便，只需要在需要做事务管理的方法上，增加@Transactional注解，以声明事务特征即可。



### 20. Spring的事务传播方式有哪些？

==参考答案==

当我们调用一个业务方法时，它的内部可能会调用其他的业务方法，以完成一个完整的业务操作。这种业务方法嵌套调用的时候，如果这两个方法都是要保证事务的，那么就要通过Spring的事务传播机制控制当前事务如何传播到被嵌套调用的业务方法中。

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时如何进行传播，如下表：

| 事务传播类型              | 说明                                                         |
| :------------------------ | :----------------------------------------------------------- |
| PROPAGATION_REQUIRED      | 如果当前没有事务，则新建一个事务；如果已存在一个事务，则加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，则以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，则抛出异常。               |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，则把当前事务挂起。               |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，则把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式执行操作，如果当前存在事务，则抛出异常。         |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

(T：主业务事务 ，如转账

 T2：交叉逻辑事务，如日志)

![image-20220514195538298](https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/image-20220514195538298.png)



### 21. Spring的事务如何配置，常用注解有哪些？

==参考答案==

![image-20220514200102417](https://raw.githubusercontent.com/ayifuture0920/java-study/master/pictures/image-20220514200102417.png)

事务的打开、回滚和提交是由事务管理器来完成的，我们使用不同的数据库访问框架，就要使用与之对应的事务管理器。在Spring Boot中，当你添加了数据库访问框架的起步依赖时，它就会进行自动配置，即自动实例化正确的事务管理器。

对于声明式事务，是使用@Transactional进行标注的。这个注解可以标注在类或者方法上。

- 当它标注在类上时，代表这个类所有公共（public）非静态的方法都将启用事务功能。
- 当它标注在方法上时，代表这个方法将启用事务功能。

另外，在@Transactional注解上，我们可以使用isolation属性声明事务的隔离级别，使用propagation属性声明事务的传播机制。

