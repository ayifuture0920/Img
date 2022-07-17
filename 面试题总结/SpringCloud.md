## Spring Cloud

### 1. 什么是Spring Cloud？

==参考答案==

`Spring Cloud` 就是微服务系统架构的一站式解决方案，在平时我们构建微服务的过程中需要做如 **注册中心** 、**配置管理中心** 、**消息总线** 、**负载均衡** 、**断路器** 、**数据监控** 等操作，而 Spring Cloud 为我们提供了一套简易的编程模型，使我们能在 Spring Boot 的基础上轻松地实现微服务项目的构建。

### 2. SpringCloud常见组件有哪些？

> **问题说明**：这个题目主要考察对SpringCloud的组件基本了解
>
> **难易程度**：简单
>
> **参考话术**：

SpringCloud 包含的组件很多，有很多功能是重复的。其中最常用组件包括：

- 注册中心组件：Eureka、Nacos等
- 负载均衡组件：Ribbon
- 远程调用组件：OpenFeign
- 网关组件：Zuul、Gateway
- 服务保护组件：Hystrix、Sentinel
- 服务配置管理组件：SpringCloudConfig、Nacos

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220630174028347.png" alt="image-20220630174028347" style="zoom: 50%;" />



<img src="https://www.sisuoit.com/wp-content/uploads/2021/11/1638198280-9a605782d5836ba.jpg" alt="Spring Cloud / Alibaba 微服务架构实战|完结无密-思索IT" style="zoom: 80%;" />



### 2. Spring Cloud 的注册中心框架：Eureka

#### 2.1 Eureka是什么？

==参考答案==

Eureka 是 Spring Cloud 的服务发现框架（即注册中心）。

> Eureka 是基于 REST（代表性状态转移）的服务，主要在 AWS 云中用于定位服务，以实现负载均衡和中间层服务器的故障转移。我们称此服务为 Eureka 服务器。Eureka 还带有一个基于 Java 的客户端组件 Eureka Client，它使与服务的交互变得更加容易。客户端还具有一个内置的负载平衡器，可以执行基本的循环负载平衡。在 Netflix，更复杂的负载均衡器将Eureka包装起来，以基于流量，资源使用，错误条件等多种因素提供加权负载均衡，以提供出色的弹性。

单体 Eureka ：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220615230454201.png" alt="image-20220615230454201" style="zoom: 40%;" />

Eureka 集群：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/16e86737ea057126%7Etplv-t2oaga2asx-zoom-in-crop-mark%3A1304%3A0%3A0%3A0.awebp" alt="Eureka架构图" style="zoom: 80%;" />



#### 2.2 order-service **(Consumer)** 如何得知 user-service **(Provider)** 实例地址？

==参考答案==

获取地址信息的流程如下：

- <font color="red">**服务注册 Register **</font>：user-service 服务实例启动后，将自己的信息注册到 eureka-server（Eureka服务端）

  当 `Eureka` 客户端向 `Eureka Server` 注册时，它提供自身的**元数据**，比如IP地址、端口，运行状况指示符URL，主页等。

  > user-service 将作为 **Provider** 提供服务，order-service 将作为 **Consumer** 向 user-service 请求服务。
  >
  > **eureka 服务也会将自己的实例信息注册到 eureka-server，目的是为了方便做 Eureka 集群管理**

  ```yaml
  ########## user-service的配置文件
  spring:
    application:
      name: user-service #user-service的服务名称
  eureka:
    client:
      service-url: #eureka-server的地址信息
        defaultZone: http://localhost:10086/eureka
  ```

- <font color="red">**服务中介**</font>：eureka-server 保存服务名称到服务实例地址列表的映射关系

  其实就是服务提供者和服务消费者之间的“桥梁”，服务提供者可以把自己注册到服务中介那里，而服务消费者如需要消费一些服务(使用一些功能)就可以在服务中介中寻找注册在服务中介的服务提供者。

  ```yaml
  ########## eureka-server的配置文件--application.yaml
  server:
    port: 10086 #eureka服务端口
  spring:
    application:
      name: eureka-server #eureka的服务名称
  eureka:
    client:
      service-url: #eureka的地址信息
        defaultZone: http://localhost:10086/eureka
  ```

  ```java
  /*EurekaApplication，用于启动eureka-server服务*/
  
  @EnableEurekaServer
  @SpringBootApplication
  public class EurekaApplication {
      public static void main(String[] args) {
          SpringApplication.run(EurekaApplication.class, args);
      }
  }
  ```

- <font color="red">**获取注册列表信息 Fetch Registries **</font>：order-service 根据服务名称，拉取实例地址列表。这个叫服务发现或服务拉取

`Eureka` 客户端从服务器获取注册表信息，并将其**缓存在本地**。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与 `Eureka` 客户端的缓存信息不同, `Eureka` 客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，`Eureka` 客户端则会重新获取整个注册表信息。 `Eureka` 服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同。`Eureka` 客户端和 `Eureka` 服务器可以使用JSON / XML格式进行通讯。在默认的情况下 `Eureka` 客户端使用压缩 `JSON` 格式来获取注册列表的信息。






#### 2.3 Consumer 如何从多个 Provider 实例中选择具体的实例？

==参考答案==

- Consumer 从实例列表中利用负载均衡算法选中一个实例地址

- 向该实例地址发起远程调用

  

#### 2.4 Consumer 如何得知某个 Provider 实例是否依然健康，是不是已经宕机？

==参考答案==

-  Provider 会每隔一段时间（默认30秒）向 eureka-server 发起请求，报告自己状态，称为心跳
- 当超过一定时间没有发送心跳时，eureka-server 会认为微服务实例故障，将该实例从服务列表中剔除
- Consumer 拉取服务时，就能将故障实例排除了

<font color="red">**服务续约 Renew**</font>：**`Eureka` 客户会每隔30秒（默认情况下）发送一次心跳来续约**。 通过续约来告知 `Eureka Server` 该 `Eureka` 客户仍然存在，没有出现问题。

<font color="red">**服务剔除 Eviction**</font>：在默认的情况下，**当 Eureka 客户端连续90秒（3个续约周期）没有向 Eureka 服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除**。

<font color="red">**服务下线 Cancel**</font>：Eureka 客户端在程序关闭时向 Eureka 服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent();`


### 3. 负载均衡 Ribbon

#### 3.1 什么是 RestTemplate?

==参考答案==

**`RestTemplate`是`Spring`提供的一个访问Http服务的客户端类**，就是微服务之间是使用的 `RestTemplate` 来实现远程调用的。如user-service 服务和 order-service 服务实都将将自己的信息注册到 eureka-server，当 order-service 服务实例需要调用 user-service 服务实例时（此时 order-service 是 **Consumer**，user-service 是 **Provider**），就需要在order-service中使用 `RestTemplate`来实现远程调用。

```java
@Autowired
private RestTemplate restTemplate;

@GetMapping("{orderId}")
public Order queryOrderByUserId(@PathVariable("orderId") Long orderId) {
    // 1.利用RestTemplate发起http请求，查询用户
    // 1.1 url路径
    // String url = "http://localhost:8081/user/" + order.getUserId();
    String url = "http://user-service/user/" + order.getUserId();
    // 1.2 发送http请求，实现远程调用
    User user = restTemplate.getForObject(url, User.class);
    order.setUser(user);
    // 2.返回
    return order;
}
```

#### 3.2 什么是 Ribbon ？

==参考答案==

`Ribbon` 是 `Netflix` 公司的一个开源的负载均衡 项目，是一个客户端/进程内负载均衡器，**运行在消费者端**。工作原理就是 `Consumer` 端获取到了所有的服务列表之后，在其**内部**使用**负载均衡算法**，进行对多个系统的调用。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220616002308499.png" alt="image-20220616002308499" style="zoom:50%;" />



#### 3.3 Ribbon的基本工作流程是什么？

==参考答案==

基本流程如下：

- LoadBalancerIntercepor 拦截我们的RestTemplate请求 http://user-service/user/1
- RibbonLoadBalancerClient 会从请求url中获取服务名称，也就是user-service
- DynamicServerListLoadBalancer 根据 user-service 到 eureka 拉取服务列表
- eureka 返回列表，localhost:8081、localhost:8082
- IRule利用内置负载均衡规则，从列表中选择一个，例如localhost:8081
- RibbonLoadBalancerClient修改请求地址，用localhost:8081替代userservice，得到http://localhost:8081/user/1，发起真实请求

#### 3.4 Nginx 和 Ribbon 的区别和相同点有哪些？

==参考答案==

提到 **负载均衡** 就不得不提到大名鼎鼎的 `Nignx` 了，而和 `Ribbon` 不同的是，它是一种**集中式**的负载均衡器。何为集中式呢？简单理解就是 **将所有请求都集中起来，然后再进行负载均衡**

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/16ebc233e2b80367%7Etplv-t2oaga2asx-zoom-in-crop-mark%3A1304%3A0%3A0%3A0.awebp" alt="img" style="zoom: 67%;" />

可以看到 `Nginx` 是接收了所有的请求进行负载均衡的，而对于 `Ribbon` 来说它是在消费者端进行的负载均衡。如下图。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/16ebc2364acb65aa%7Etplv-t2oaga2asx-zoom-in-crop-mark%3A1304%3A0%3A0%3A0.awebp" alt="img" style="zoom: 67%;" />

> 请注意 `Request` 的位置，在 `Nginx` 中请求是先进入负载均衡器，而在 `Ribbon` 中是先在客户端进行负载均衡才进行请求的。

#### 3.5 Ribbon 的几种负载均衡算法

==参考答案==

在 `Ribbon` 中有很多的负载均衡调度算法，其默认是使用的 `RoundRobinRule` 轮询策略。

- **RoundRobinRule**：轮询策略。`Ribbon` 默认采用的策略。若经过一轮轮询没有找到可用的 `provider`，其最多轮询 10 轮。若最终还没有找到，则返回 null。
- **RandomRule**: 随机策略，从所有可用的 provider 中随机选择一个。
- **RetryRule**: 重试策略。先按照 RoundRobinRule 策略获取 provider，若获取失败，则在指定的时限内重试。默认的时限为 500 毫秒。
- **ZoneAvoidanceRule** ：以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。
- **WeightedResponseTimeRule**：为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。  

最需要知道的是默认轮询算法，并且可以更换默认的负载均衡算法，只需要在配置文件中做出修改就行。

```yaml
${providerName}: # 给某个微服务配置负载均衡规则，比如userservice服务
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

### 4. SpringCloudAlibaba 的注册中心：Nacos

[Nacos](https://nacos.io/) 是阿里巴巴的产品，现在是 [SpringCloud](https://spring.io/projects/spring-cloud) 中的一个组件。相比 [Eureka](https://github.com/Netflix/eureka) 功能更加丰富，在国内受欢迎程度较高。

Nacos 是 SpringCloudAlibaba 的组件，而 SpringCloudAlibaba 也遵循 SpringCloud 中定义的服务注册、服务发现规范。因此使用 Nacos 和使用 Eureka 对于微服务来说，并没有太大区别。

#### 4.1 Nacos 与 Eureka 的共同点和区别

==参考答案==

Nacos的服务实例分为两种类型：

- **临时实例**：如果实例宕机超过一定时间，会从服务列表剔除。默认的类型

- **非临时实例**：如果实例宕机，不会从服务列表剔除，也可以叫永久实例。

Nacos 和 Eureka整体结构类似，**服务注册**、**服务拉取**、**心跳等待**，但是也存在一些差异：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210714001728017.png" alt="image-20210714001728017" style="zoom: 50%;" />

1. **Nacos与eureka的共同点**
   - 都支持服务注册和服务拉取	

   - 都支持服务提供者心跳方式做健康检测

2. **Nacos**与 **Eureka** 的区别有哪些？

> **问题说明**：考察对Nacos、Eureka的底层实现的掌握情况
>
> **难易程度**：难
>
> **参考话术**：

Nacos 与 Eureka 的不同之处，可以从以下几点来描述：

- **接口方式**：Nacos 与 Eureka 都对外暴露了 Rest 风格的 API 接口，用来实现服务注册、发现等功能
- **实例类型**：Nacos 的实例有永久和临时实例之分；而 Eureka 只支持临时实例
- **健康检测**：Nacos 对临时实例采用心跳模式检测，对永久实例采用主动请求来检测；Eureka 只支持心跳模式
- **服务发现**：Nacos 支持定时拉取和订阅推送两种模式；Eureka 只支持定时拉取模式

#### 4.2 Nacos 配置共享的优先级是什么？

==参考答案==

当nacos、服务本地同时出现相同属性时，优先级有高低之分：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210714174623557.png" alt="image-20210714174623557" style="zoom:67%;" /> 

#### 4.3 Nacos的服务注册表结构是怎样的？

> **问题说明**：考察对Nacos数据分级结构的了解，以及Nacos源码的掌握情况
>
> **难易程度**：一般
>
> **参考话术**：

Nacos 采用了数据的**分级存储模型**，最外层是 **Namespace** ，用来隔离环境。然后是 **Group**，用来对服务分组。接下来就是**服务（Service）**了，一个服务包含多个**实例**，但是可能处于不同机房，因此 Service 下有多个**集群（Cluster）**，Cluster 下是不同的**实例（Instance）**。

对应到Java代码中，Nacos 采用了一个多层的 Map 来表示。结构为 **Map<String, Map<String, Service>>**，其中最外层 Map 的 key 就是 namespaceId，值是一个 Map 。内层 Map 的 key 是 group 拼接 serviceName，值是 Service 对象。Service 对象内部又是一个Map，key 是集群名称，值是Cluster对象。而Cluster对象内部维护了Instance的集合。



如图：

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20210925215305446.png" alt="image-20210925215305446" style="zoom: 50%;" />

#### 4.4 Nacos如何支撑阿里内部数十万服务注册压力？

> **问题说明**：考察对Nacos源码的掌握情况
>
> **难易程度**：难
>
> **参考话术**：

Nacos 内部接收到注册的请求时，不会立即写数据，而是将服务注册的任务放入一个阻塞队列就立即响应给客户端。然后利用线程池读取阻塞队列中的任务，异步来完成实例更新，从而提高并发写能力。



### 5. 服务网关

#### 5.1 为什么需要服务网关？

网关是所有微服务的统一入口，网关有三个核心功能：

**权限控制**：网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**路由和负载均衡**：一切请求都必须先经过 gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**限流**：当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大。

在SpringCloud中网关的实现包括两种：

- gateway
- zuul

Zuul 是基于 Servlet 的实现，属于阻塞式编程。而SpringCloudGateway 则是基于 Spring5 中提供的 WebFlux，属于响应式编程的实现，具备更好的性能。

### 6. 跨域问题

#### 6.1 了解跨域问题吗？

> **跨域**：<u>浏览器</u>对于 <u>javascript</u> 的 <u>同源策略</u> 的限制 。

> **同源策略**：是指<u>协议</u>，<u>域名</u>，<u>端口</u>都要相同，其中有一个不同都会产生跨域；  

以下情况都属于跨域：

| 跨域原因说明       | 示例                                      |
| ------------------ | ----------------------------------------- |
| 域名不同           | `www.jd.com` 与 `www.taobao.com`          |
| 域名相同，端口不同 | `www.jd.com:8080` 与 `www.jd.com:8081`    |
| 二级域名不同       | `item.jd.com` 与 `miaosha.jd.com`         |
| 协议不同           | `http://www.a.com` 与 `https://www.a.com` |

#### 6.2 什么是 CORS ？

##### 6.2.1 **CORS** (Cross Origin Resource Sharing) ：跨源资源共享

> 跨源资源共享 ( CORS )（或通俗地译为跨域资源共享）是一种基于 HTTP 头的机制，该机制通过允许服务器标示除了它自己以外的其它 origin（域，协议和端口），使得浏览器允许这些 origin 访问加载自己的资源。跨源资源共享还通过一种机制来检查服务器是否会允许要发送的真实请求，该机制通过浏览器发起一个到服务器托管的跨源资源的"预检"请求。在预检中，浏览器发送的头中标示有 HTTP 方法和真实请求中会用到的头。

它允许浏览器向跨源服务器，发出[`XMLHttpRequest`](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)请求，从而克服了AJAX只能[同源](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)使用的限制。CORS 请求失败会产生错误，但是为了安全，在 JavaScript 代码层面是无法获知到底具体是哪里出了问题。你只能查看浏览器的控制台以得知具体是哪里出现了错误。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220704160057435.png" alt="image-20220704160057435" style="zoom: 67%;" />

##### 6.2.2 简单请求

某些请求不会触发 CORS 预检请求。这样的请求为“简单请求”。若请求 满足所有下述条件，则该请求可视为“简单请求”：

- 使用下列方法之一：
  - [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)
  - [`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD)
  - [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST)
- 除了被用户代理自动设置的首部字段（例如`Connection`，`User-Agent`）和在 Fetch 规范中定义为禁用首部名称的其他首部，允许人为设置的字段为 Fetch 规范定义的对 CORS 安全的首部字段集合。该集合为：
  - [`Accept`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept)
  - [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)
  - [`Content-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Language)
  - [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type)（需要注意额外的限制）
- `Content-Type`的值仅限于下列三者之一：
  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`
- 请求中的任意 [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 对象均没有注册任何事件监听器；[`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 对象可以使用 [`XMLHttpRequest.upload`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/upload) 属性访问。
- 请求中没有使用 [`ReadableStream`](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream) 对象。

##### 6.2.3 预检请求

与前述简单请求不同，“需预检的请求”要求必须首先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响。

<img src="https://raw.githubusercontent.com/ayifuture0920/java-study/main/pictures/image-20220704140424816.png" alt="image-20220704140424816" style="zoom: 50%;" />



#### 6.3 如何解决跨域问题？

目前比较常用的跨域解决方案有3种：

- **Jsonp**

最早的解决方案，利用 script 标签可以跨域的原理实现。限制：需要服务的支持。只能发起 GET 请求

- **nginx 反向代理**


思路是：利用nginx把跨域反向代理为不跨域，支持各种请求方式

缺点：需要在nginx进行额外配置，语义不清晰 

- **CORS**

规范化的跨域请求解决方案，安全可靠。

优势：

- 在服务端进行控制是否允许跨域，可自定义规则
- 支持各种请求方式

缺点：

- 会产生额外的请求

我们这里会采用 cors 的跨域方案。
