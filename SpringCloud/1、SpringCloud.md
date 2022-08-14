# 1、认识微服务

需要学习的技术栈：

![image-20220808203917063](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808203917063.png)



## 1.1、单体架构

将业务的所有功能集中在一个项目中开发，打成一个包部署。

![image-20220808205537391](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808205537391.png)

**优点：**

- 架构简单，一个项目就能完成
- 部署成本低，一个tomcat，把包放上去，就能访问了

**缺点：**

- 耦合度高 (维护困难、升级困难)



## 1.2、分布式架构

根据业务功能对系统做拆分，每个业务功能模块作为独立项目开发，称为一个服务。

![image-20220808205846372](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808205846372.png)



**优点：**

- 降低服务耦合
- 有利于服务升级拓展

**缺点：**

- 服务调用关系错综复杂
  - 比如：在单体服务中，如果要查询订单信息，直接调service就行了。在分布式中，不能直接调用代码，只能通过远程调用接口。



分布式架构虽然降低了服务耦合，但是服务拆分时也有很多问题需要思考：

- 服务拆分粒度如何？
- 服务集群地址如何维护？
- 服务之间如何实现远程调用？
- 服务健康状态如何感知？



## 1.3、微服务

微服务是一种经过良好架构设计的**分布式**架构方案，微服务架构特征：

- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

![image-20220808210723732](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808210723732.png)

微服务这种方案需要技术框架来落地，全球的互联网公司都在积极尝试自己的微服务落地技术。在国内最知名的就是SpringCloud和阿里巴巴的Dubbo。

![image-20220808211528462](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808211528462.png)

微服务技术对比：

![image-20220808211727062](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808211727062.png)

企业需求：

![image-20220808212244482](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808212244482.png)

## 1.4、SpringCloud

SpringCloud是目前国内使用最广泛的微服务框架。官网地址：https://spring.io/projects/spring-cloud。

SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验。

其中常见的组件包括：

![image-20220808212638035](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808212638035.png)

另外，SpringCloud底层是依赖于SpringBoot的，并且有版本的兼容关系，如下：

![image-20220808212703885](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808212703885.png)

我们学习的版本是 Hoxton.SR10，因此对应的SpringBoot版本是2.3.x版本。



## 1.5、总结

- 单体架构：简单方便，高度耦合，扩展性差，适合小型项目。例如：学生管理系统
- 分布式架构：松耦合，扩展性好，但架构复杂，难度大。适合大型互联网项目，例如：京东、淘宝
- 微服务：一种良好的分布式架构方案
  - 优点：拆分粒度更小、服务更独立、耦合度更低
  - 缺点：架构非常复杂，运维、监控、部署难度提高
- SpringCloud是微服务架构的一站式解决方案，集成了各种优秀微服务功能组件



# 2、服务拆分和远程调用

任何分布式架构都离不开服务的拆分，微服务也是一样。



## 2.1、服务拆分原则

**服务拆分注意事项：**

- 单一职责：不同微服务，不要重复开发相同业务
- 数据独立：不要访问其它微服务的数据库
- 面向服务：将自己的业务暴露为接口，供其它微服务调用

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808213114766.png" alt="image-20220808213114766" style="zoom: 67%;" />



## 2.2、服务拆分示例

**导入服务拆分Demo**

1. 导入课前资料提供的工程：cloud-demo
2. 项目结构：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808213322558.png" alt="image-20220808213322558" style="zoom:50%;" />

3. 将课前资料准备的sql导入数据库中：cloud-user.sql、cloud-order.sql



cloud-demo：父工程，管理依赖

- order-service：订单微服务，负责订单相关业务
- user-service：用户微服务，负责用户相关业务

要求：

- 订单微服务和用户微服务都必须有各自的数据库，相互独立
- 订单服务和用户服务都对外暴露Restful的接口
- 订单服务如果需要查询用户信息，只能调用用户服务的Restful接口，不能查询用户数据库



### 2.2.1、导入sql

首先，将课前资料提供的`cloud-order.sql`和`cloud-user.sql`导入到mysql中。

cloud-user表中初始数据如下：

![image-20220808213932599](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808213932599.png)

cloud-order表中初始数据如下：

![image-20220808214001576](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808214001576.png)

cloud-order表中持有cloud-user表中的id字段。



### 2.2.2、导入demo工程

项目结构如下：

![image-20220808214530130](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808214530130.png)

导入后，会在IDEA右下角出现弹窗：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808214606748.png" alt="image-20220808214606748"  />

点击弹窗，然后选择：Show run configurations in Services，就会看到这样的菜单：

![image-20220808214727125](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808214727125.png)



## 2.3、实现远程调用案例

在order-service服务中，有一个根据id查询订单的接口，查询任意订单，用户总为null：

```json
{"id":101,"price":699900,"name":"Apple 苹果 iPhone 12 ","num":1,"userId":1,"user":null}
```

在user-service中有一个根据id查询用户的接口。

**总结：**

- 微服务需要根据业务模块拆分，做到单一职责，不要重复开发相同业务
- 微服务可以将业务暴露为接口，供其它微服务使用
- 不同微服务都应该有自己独立的数据库



### 2.3.1、案例需求

需求：根据订单id查询订单的同时，把订单所属的用户信息一起返回

![image-20220808221725545](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808221725545.png)

查询出订单信息后，需要远程调用服务获取用户信息。

**远程调用方式分析：**

![image-20220808222033287](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220808222033287.png)

因此，我们需要在order-service中 向user-service发起一个http的请求，调用http://localhost:8081/user/{userId}这个接口。

**步骤：**

1. 注册RestTemplate

   在order-service的OrderApplication中注册RestTemplate

   ```java
   @MapperScan("cn.itcast.order.mapper")
   @SpringBootApplication
   public class OrderApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(OrderApplication.class, args);
       }
   
       @Bean
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

   

2. 服务远程调用RestTemplate

   修改order-service中的OrderService的queryOrderById方法

   ```java
   @Service
   public class OrderService {
       @Autowired
       private OrderMapper orderMapper;
       @Autowired
       private RestTemplate restTemplate;
       public Order queryOrderById(Long orderId) {
           // 1.查询订单
           Order order = orderMapper.findById(orderId);
           // 2. 利用RestTemplate发起请求
           // 2.1. url路径
           String url = "http://localhost:8081/user/" + order.getUserId();
           // 2.2. 发送http请求，实现远程调用
           User user = restTemplate.getForObject(url, User.class);
           // 3. 封装 user对象到 order
           order.setUser(user);
           // 4.返回
           return order;
       }
   }
   ```



### 2.3.2、总结

**微服务调用方式：**

- 基于RestTemplate发起的http请求实现远程调用
- http请求做远程调用是与语言无关的调用，只要知道对方的ip、端口、接口路径、请求参数即可。



## 2.4、提供者与消费者

在服务调用关系中，会有两个不同的角色：

- 服务提供者：一次业务中，被其它微服务调用的服务。（提供接口给其它微服务）
- 服务消费者：一次业务中，调用其它微服务的服务。（调用其它微服务提供的接口）

![image-20220809210937195](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809210937195.png)

但是，服务提供者与服务消费者的角色并不是绝对的，而是相对于业务而言。

如果服务A调用了服务B，而服务B又调用了服务C，服务B的角色是什么？

- 对于A调用B的业务而言：A是服务消费者，B是服务提供者
- 对于B调用C的业务而言：B是服务消费者，C是服务提供者

因此，服务B既可以是服务提供者，也可以是服务消费者。



# 3、Eureka注册中心

**服务调用出现的问题：**

- 服务消费者该如何获取服务提供者的地址信息
- 如果有多个服务提供者，消费者该如何选择
- 消费者如何得知服务提供者的健康状态

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809211555833.png" alt="image-20220809211555833" style="zoom:50%;" />

## 3.1、Eureka的结构和作用

这些问题都需要利用SpringCloud中的注册中心来解决，其中最广为人知的注册中心就是Eureka，其结构如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809212045335.png" alt="image-20220809212045335" style="zoom:67%;" />

**消费者该如何获取服务提供者具体信息**

- 服务提供者启动时向eureka注册自己的信息
- eureka保存这些信息
- 消费者根据服务名称向eureka拉取提供者信息

**如果有多个服务提供者，消费者该如何选择**

- 服务消费者利用负载均衡算法，从服务列表中挑选一个

**消费者如何感知服务提供者健康状态**

- 服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态
- eureka会更新记录服务列表信息，心跳不正常会被剔除
- 消费者就可以拉取到最新的信息



> 注意：一个微服务，既可以是服务提供者，又可以是服务消费者，因此eureka将服务注册、服务发现等功能统一封装到了eureka-client端



**总结：**

在Eureka架构中，微服务角色有两类：

- EurekaServer：服务端，注册中心
  - 记录服务信息
  - 心跳监控
- EurekaClient：客户端
  - Provider：服务提供者，例如案例中的 user-service
    - 注册自己的信息到EurekaServer
    - 每隔30秒向EurekaServer发送心跳
  - consumer：服务消费者，例如案例中的 order-service
    - 根据服务名称从EurekaServer拉取服务列表
    - 基于服务列表做负载均衡，选中一个微服务后发起远程调用



## 3.2、搭建eureka-server

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809212848630.png" alt="image-20220809212848630" style="zoom:67%;" />

### 3.2.1、创建eureka-server服务

在cloud-demo父工程下，创建一个子模块，选择maven模块，不选择骨架，命名为eureka-server。



### 3.2.2、引入eureka依赖

引入SpringCloud为eureka提供的starter依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```



### 3.2.3、编写启动类

给eureka-server服务编写一个启动类，一定要添加一个@EnableEurekaServer注解，开启eureka的注册中心功能：

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```



### 3.2.4、编写配置文件

编写一个application.yml文件，内容如下：

```yaml
server:
  port: 10086  # 服务端口
spring:
  application:
    name: eurekaserver # eureka的服务名称，注册的时候需要带上服务名字
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka # eureka的地址信息，多个逗号隔开
```

eureka自己也是一个微服务，启动时会把自己也注册到eureka服务上，所以也需要配置服务名和地址信息，作为注册信息使用。



### 3.2.5、启动eureka服务

启动微服务，然后在浏览器访问：http://127.0.0.1:10086

看到下面结果就是成功了：

![image-20220809214742165](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809214742165.png)

`Instances currently registered with Eureka`是最关键的！意思是：注册到Eureka的实例 (一个服务，每部署一份，就是一个实例)

Eureka会记录一个服务的所有实例，图中注册的就是eureka自己。



### 3.2.6、总结

- 搭建EurekaServer
  - 引入eureka-server依赖
  - 添加@EnableEurekaServer注解
  - 在application.yml中配置eureka地址



## 3.3、服务注册

下面，我们将user-service注册到eureka-server中去。



### 3.3.1、引入依赖

在user-service的pom文件中，引入下面的eureka-client依赖：

```yaml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



### 3.3.2、编写配置文件

在user-service中，修改application.yml文件，添加服务名称、eureka地址：

```yaml
spring:
  application:
    name: userservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

order-service也是同样的步骤。



### 3.3.3、启动多个user-service实例

我们可以将user-service多次启动， 模拟多实例部署，但为了避免端口冲突，需要修改端口设置：

![image-20220809220541176](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809220541176.png)

填写Name为UserApplication2；

填写VM options为：-Dserver.port=8082

现在，SpringBoot窗口会出现两个user-service启动配置，不过，第一个是8081端口，第二个是8082端口。

启动两个user-service实例。

查看eureka-server管理页面：

![image-20220809221014304](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809221014304.png)



### 3.3.4、总结

- 服务注册
  - 引入eureka-client依赖
  - 在application.yml中配置eureka地址
- 无论是消费者还是提供者，引入eureka-client依赖、知道eureka地址后，都可以完成服务注册



## 3.4、服务发现

下面，我们将order-service的逻辑修改：向eureka-server拉取user-service的信息，实现服务发现。



### 3.4.1、引入依赖

之前说过，服务发现、服务注册统一都封装在eureka-client依赖，因此这一步与服务注册时一致。

在order-service的pom文件中，引入下面的eureka-client依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



### 3.4.2、配置文件

服务发现也需要知道eureka地址，因此第二步与服务注册一致，都是配置eureka信息；

在order-service中，修改application.yml文件，添加服务名称、eureka地址：

```yaml
spring:
  application:
    name: orderservice
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```



### 3.4.3、服务拉取和负载均衡

服务拉取是基于服务名称获取服务列表，然后再对服务列表做负载均衡；

不过这些动作不用我们去做，只需要添加一些注解即可。

1. 修改OrderService的代码，修改访问的url路径，用服务名代替ip、端口：

   ```java
   String url = "http://userservice/user/" + order.getUserId();
   ```

2. 在order-service项目的启动类OrderApplication中的RestTemplate添加**负载均衡**注解：

   ```java
   @Bean
   @LoadBalanced
   public RestTemplate restTemplate(){
       return new RestTemplate();
   }
   ```

spring会自动帮助我们从eureka-server端，根据userservice这个服务名称，获取实例列表，而后完成负载均衡。



### 3.4.4、总结

- 服务发现
  - 引入eureka-client依赖
  - 在application.yml中配置eureka地址
  - 给RestTemplate添加@LoadBalanced注解
  - 用服务提供者的服务名称远程调用



# 4、Ribbon负载均衡

上一节中，我们添加了@LoadBalanced注解，即可实现负载均衡功能，这是什么原理呢？



## 4.1、负载均衡原理

SpringCloud底层其实是利用了一个名为Ribbon的组件，来实现负载均衡功能的。

![image-20220809223720739](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220809223720739.png)

那么我们发出的请求明明是http://userservice/user/1，怎么变成了http://localhost:8081的呢？



## 4.2、源码跟踪

之前写在RestTemplate注册bean上的`@LoadBalanced`：表示RestTemplate发起的请求要被Ribbon拦截和处理了。

为什么我们只输入了service名称就可以访问了呢？之前还要获取ip和端口。

显然有人帮我们根据service名称，获取到了服务实例的ip和端口。它就是`LoadBalancerInterceptor`，这个类会在对RestTemplate的请求进行拦截，然后从Eureka根据服务id获取服务列表，随后利用负载均衡算法得到真实的服务地址信息，替换服务id。



### 4.2.1、LoadBalancerInterceptor

![image-20220811211852202](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811211852202.png)

`LoadBalancerInterceptor`实现了`ClientHttpRequestInterceptor`接口，这个接口里的方法，代表拦截由客户端发起的请求。

调用order-service的方法后，可以看到这里的intercept方法，拦截了用户的HttpRequest请求，然后做了几件事：

- `request.getURI()`：获取请求uri，本例中就是 http://user-service/user/1
- `originalUri.getHost()`：获取uri路径的主机名，其实就是服务id，`user-service`
- `this.loadBalancer.execute()`：处理服务id，和用户请求。

这里的`this.loadBalancer`是`LoadBalancerClient`类型，我们继续跟入。



### 4.2.2、LoadBalancerClient

继续跟入`LoadBalancerClient`的execute方法，可以进到`RibbonLoadBalancerClient`类中：

![image-20220811213149105](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811213149105.png)

代码是这样的：

- getLoadBalancer(serviceId)：根据服务id获取ILoadBalancer，而ILoadBalancer会拿着服务id去eureka中获取服务列表并保存起来。
- getServer(loadBalancer)：利用内置的负载均衡算法，从服务列表中选择一个。本例中，可以看到获取了8082端口的服务

放行后，再次访问并跟踪，发现`getServer`方法获取的是127.0.0.1:8081。

果然实现了负载均衡。



### 4.2.3、负载均衡策略IRule

在刚才的代码中，可以看到获取服务使通过一个`getServer`方法来做负载均衡。

我们继续跟入：

![image-20220811214040795](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811214040795.png)



继续跟踪源码chooseServer方法，发现这么一段代码：

![image-20220811214254554](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811214254554.png)



我们看看这个rule是谁：

![image-20220811214356749](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811214356749.png)



这里的rule默认值是一个`RoundRobinRule`，看类的介绍：

![image-20220811214410231](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811214410231.png)

这不就是轮询的意思吗。

到这里，整个负载均衡的流程我们就清楚了。



### 4.2.4、总结

SpringCloudRibbon的底层采用了一个拦截器，拦截了RestTemplate发出的请求，对地址做了修改。用一幅图来总结一下：

![image-20220811214641951](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811214641951.png)

基本流程如下：

- 拦截我们的RestTemplate请求http://userservice/user/1
- RibbonLoadBalancerClient会从请求url中获取服务名称，也就是user-service
- DynamicServerListLoadBalancer根据user-service到eureka拉取服务列表
- eureka返回列表，localhost:8081、localhost:8082
- IRule利用内置负载均衡规则，从列表中选择一个，例如localhost:8081
- RibbonLoadBalancerClient修改请求地址，用localhost:8081替代userservice，得到http://localhost:8081/user/1，发起真实请求



## 4.3、负载均衡策略

### 4.3.1、负载均衡策略

Ribbon的负载均衡规则是一个叫做IRule的接口来定义的，每一个子接口都是一种规则：

![image-20220811215228832](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220811215228832.png)

不同规则的含义如下：

| **内置负载均衡规则类**    | **规则描述**                                                 |
| ------------------------- | ------------------------------------------------------------ |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。它是Ribbon默认的负载均衡规则。 |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略：   （1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。  （2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit属性进行配置。 |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| **ZoneAvoidanceRule**     | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。 |
| BestAvailableRule         | 忽略那些短路的服务器，并选择并发数较低的服务器。             |
| RandomRule                | 随机选择一个可用的服务器。                                   |
| RetryRule                 | 重试机制的选择逻辑                                           |

默认的实现就是ZoneAvoidanceRule，是一种轮询方案。



### 4.3.2、自定义负载均衡策略

通过定义IRule实现可以修改负载均衡规则，有两种方式：

1. 代码方式：在order-service中的OrderApplication类中，定义一个新的IRule：

   ```java
   @Bean
   public IRule randomRule(){
       return new RandomRule(); // 根据返回值类型进行定义
   }
   ```

2. 配置文件方式：在order-service的application.yml文件中，添加新的配置也可以修改规则：

   ```yaml
   userservice: # 给某个微服务配置负载均衡规则，这里是userservice服务名
     ribbon:
       NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则 
   ```

注意：

- 代码方式，是全局的配置。只要是在order-service服务中，不管调用户服务还是商品服务，都是随机策略。
- 配置文件方式，是局部的配置。案例中指定的是userservice，表示在调用用户服务时才是随机策略。

- 一般用默认的负载均衡规则，不做修改。



## 4.4、饥饿加载

Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长。

而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载：

```yaml
ribbon:
  eager-load:
    enabled: true # 开启饥饿加载
    clients: # 指定对userservice这个服务饥饿加载，多个服务直接在下面加
      - userservice
```



# 5、Nacos注册中心

国内公司一般都推崇阿里巴巴的技术，比如注册中心，SpringCloudAlibaba也推出了一个名为Nacos的注册中心。



## 5.1、认识和安装Nacos

Nacos是阿里巴巴的产品，现在是SpringCloud中的一个组件。相比Eureka功能更加丰富，在国内受欢迎程度较高。



### 5.1.1、Windows安装

开发阶段采用单机安装即可。

1. 下载安装包

   在Nacos的GitHub页面，提供有下载链接，可以下载编译好的Nacos服务端或者源代码：

   GitHub主页：https://github.com/alibaba/nacos

   GitHub的Release下载页：https://github.com/alibaba/nacos/releases

   本课程采用1.4.1.版本的Nacos，windows版本使用`nacos-server-1.4.1.zip`包即可。

2. 解压

   将这个包解压到任意非中文目录下。

   目录说明：

   - bin：启动脚本
   - conf：配置文件

3. 端口配置

   Nacos的默认端口是8848，如果你电脑上的其它进程占用了8848端口，请先尝试关闭该进程。

   如果无法关闭占用8848端口的进程，也可以进入nacos的conf目录，修改配置文件`application.properties`中的端口：

   ```properties
   #*************** Spring Boot Related Configurations ***************#
   ### Default web context path:
   server.servlet.contextPath=/nacos
   ### Default web server port:
   server.port=8848
   ```

4. 启动

   启动非常简单，进入bin目录。然后执行命令即可：

   ```
   startup.cmd -m standalone  #standalone表示单机启动，还有集群启动。
   ```

5. 访问

   在浏览器输入cmd启动时的地址即可访问。



### 5.1.2、Linux安装

Linux或者Mac安装方式与Windows类似。

1. 安装JDK

   Nacos依赖于JDK运行，索引Linux上也需要安装JDK才行。

   上传到某个目录，例如：`/usr/local/`

   然后解压缩：

   ```sh
   tar -xvf jdk-8u144-linux-x64.tar.gz
   ```

   然后重命名为java

   配置环境变量：

   ```sh
   export JAVA_HOME=/usr/local/java
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   设置环境变量：

   ```sh
   source /etc/profile
   ```

2. 上传安装包

   使用tar.gz结尾的压缩包。

   上传到Linux服务器的某个目录，例如`/usr/local/src`目录下。

3. 解压

   命令解压缩安装包：

   ```sh
   tar -xvf nacos-server-1.4.1.tar.gz
   ```

   然后删除安装包：

   ```sh
   rm -rf nacos-server-1.4.1.tar.gz
   ```

4. 端口配置

   与windows中类似

5. 启动

   在nacos/bin目录中，输入命令启动Nacos：

   ```sh
   sh startup.sh -m standalone
   ```



## 5.2、服务注册到Nacos

Nacos是SpringCloudAlibaba的组件，而SpringCloudAlibaba也遵循SpringCloud中定义的服务注册、服务发现规范。因此使用Nacos和使用Eureka对于微服务来说，并没有太大区别。

主要差异在于：

- 依赖不同
- 服务地址不同



### 5.2.1、服务注册

1. 在cloud-demo父工程中添加spring-cloud-alilbaba的管理依赖：

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-alibaba-dependencies</artifactId>
       <version>2.2.5.RELEASE</version>
       <type>pom</type>
       <scope>import</scope>
   </dependency>
   ```

2. 注释掉order-service和user-service中原有的eureka依赖。

3. 在user-service和order-service中的pom文件中引入nacos-discovery依赖：

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

4. 修改user-service和order-service中的application.yml文件，注释eureka地址，添加nacos地址：

   ```yaml
   spring:
     cloud:
       nacos:
         server-addr: localhost:8848 # Nacos服务地址
   ```

5. 启动并测试：

   ![image-20220813095012479](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813095012479.png)



### 5.2.2、总结

- Nacos服务搭建
  - 下载安装包
  - 解压
  - 在bin目录下运行指令：startup.cmd -m standalone
- Nacos服务注册或发现
  - 引入nacos.discovery依赖
  - 配置nacos地址spring.cloud.nacos.server-addr



## 5.3、Nacos服务分级存储模型

一个**服务**可以有多个**实例**，例如我们的user-service，可以有:

- 127.0.0.1:8081
- 127.0.0.1:8082
- 127.0.0.1:8083

假如这些实例分布于全国各地的不同机房，例如：

- 127.0.0.1:8081，在上海机房
- 127.0.0.1:8082，在上海机房
- 127.0.0.1:8083，在杭州机房

Nacos就将同一机房内的实例 划分为一个**集群**。这样也可以做到容灾。

也就是说，user-service是服务，一个服务可以包含多个集群，如杭州、上海，每个集群下可以有多个实例，形成分级模型，如图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813095924395.png" alt="image-20220813095924395" style="zoom: 67%;" />

服务调用尽可能选择本地集群的服务，跨集群调用延迟较高

本地集群不可访问时，再去访问其它集群

![image-20220813100056972](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813100056972.png)

杭州机房内的order-service应该优先访问同机房的user-service。



### 5.3.1、给user-service配置集群

修改user-service的application.yml文件，添加集群配置：

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称，这里代指杭州
```

重启两个user-service实例后，我们可以在nacos控制台看到下面结果：

![image-20220813100856580](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813100856580.png)

我们再次复制一个user-service启动配置，添加属性：

```sh
-Dserver.port=8083 -Dspring.cloud.nacos.discovery.cluster-name=SH
```

启动UserApplication3后再次查看nacos控制台：

![image-20220813100952747](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813100952747.png)



### 5.3.2、同集群优先的负载均衡

默认的`ZoneAvoidanceRule`并不能实现根据同集群优先来实现负载均衡。

因此Nacos中提供了一个`NacosRule`的实现，可以优先从同集群中挑选实例。

1. 给order-service配置集群信息

   修改order-service的application.yml文件，添加集群配置：

   ```yaml
   spring:
     cloud:
       nacos:
         server-addr: localhost:8848
         discovery:
           cluster-name: HZ # 集群名称
   ```

2. 修改负载均衡规则

   修改order-service的application.yml文件，修改负载均衡规则：

   ```yaml
   userservice:
     ribbon:
       NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
   ```

在本地集群没有userserver实例时，会访问其他集群的userserver服务。



### 5.3.3、总结

> Nacos服务分级模型

- Nacos服务分级存储模型
  - 一级是服务，例如userservice
  - 二级是集群，例如杭州或上海
  - 三级是实例，例如杭州机房的某台部署了userservice的服务器
- 如何设置实例的集群属性
  - 修改application.yml文件，添加spring.cloud.nacos.discovery.cluster-name属性即可



> 集群负载均衡策略

- NacosRule负载均衡策略
  - 优先选择同集群服务实例列表
  - 本地集群找不到提供者，才去其它集群寻找，并且会报警告
  - 确定了可用实例列表后，再采用随机负载均衡挑选实例



## 5.4、权重负载均衡

实际部署中会出现这样的场景：

服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。

但默认情况下NacosRule是同集群内随机挑选，不会考虑机器的性能问题。

因此，Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高。

在nacos控制台，找到user-service的实例列表，点击编辑，在弹出的编辑窗口，即可修改权重：

![image-20220813103737921](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813103737921.png)



> **注意**：
>
> Nacos控制台可以设置实例的权重值，0~1之间
>
> 同集群内的多个实例，权重越高被访问的频率越高
>
> 如果权重修改为0，则该实例永远不会被访问



## 5.5、环境隔离

Nacos中服务存储和数据存储的最外层都是一个名为namespace的东西，用来做最外层隔离。

Nacos提供了namespace来实现环境隔离功能。

- nacos中可以有多个namespace
- namespace下可以有group、service等
- 不同namespace之间相互隔离，例如不同namespace的服务互相不可见

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813104342626.png" alt="image-20220813104342626" style="zoom:67%;" />

### 5.5.1、创建namespace

默认情况下，所有service、data、group都在同一个namespace，名为public

我们可以点击页面新增按钮，添加一个namespace：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813104752979.png" alt="image-20220813104752979" style="zoom:80%;" />

然后填写表单：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813105020640.png" alt="image-20220813105020640" style="zoom: 80%;" />

就能在页面看到一个新的namespace。



### 5.5.2、给微服务配置namespace

给微服务配置namespace只能通过修改配置来实现。

例如，修改order-service的application.yml文件：

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ
        namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间，填ID
```

重启order-service后，访问控制台，可以看到order-service已经跑到dev中了：

![image-20220813105422695](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813105422695.png)

此时访问order-service，因为namespace不同，会导致找不到userservice，控制台会报错：

![image-20220813105532321](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813105532321.png)



### 5.5.3、总结

- Nacos环境隔离
  - 每个namespace都有唯一id
  - 服务设置namespace时要写id而不是名称
  - 不同namespace下的服务互相不可见



# 6、Nacos与Eureka的区别

Nacos的服务实例分为两种类型：

- 临时实例：如果实例宕机超过一定时间，会从服务列表剔除，默认的类型。
- 非临时实例：如果实例宕机，不会从服务列表剔除，也可以叫永久实例。

配置一个服务实例为永久实例：

```yaml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false # 设置为非临时实例
```



![image-20220813110133511](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220813110133511.png)



Nacos和Eureka整体结构类似，服务注册、服务拉取、心跳等待，但是也存在一些差异：

- Nacos与eureka的共同点
  - 都支持服务注册和服务拉取
  - 都支持服务提供者心跳方式做健康检测
- Nacos与Eureka的区别
  - Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
  - 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
  - Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
  - Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式