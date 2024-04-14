# 一、初识Sentinel



## 1、雪崩问题及解决方案

微服务中，服务间调用关系错综复杂，一个微服务往往依赖于多个其他微服务。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414141520969.png" alt="image-20240414141520969" style="zoom: 50%;" />

如图，如果服务提供者D发生了故障，当前应用A的部分业务因为依赖于服务D，因此也会被阻塞。此时，其他不依赖于服务D的业务似乎不受影响。

但是，依赖服务D的业务请求被阻塞，用户不会得到响应，则Tomcat的这个线程不会释放，于是越来越多的用户请求到来，越来越多的线程会阻塞。

服务器支持的线程和并发数有限，请求一直阻塞，会导致服务器资源耗尽，从而导致所有其他服务都不可用，那么当前服务也就不可用了。

那么，依赖于当前服务的其他服务随着时间的推移，最终也都会变得不可用，形成级联失败，雪崩就发生了

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715172710340.png" alt="image-20210715172710340" style="zoom: 67%;" />

微服务调用链路中的某个服务故障，引起整个链路中的所有微服务都不可用，这就是雪崩。



### 1.1、四种解决方案

- **超时处理**：设定超时时间，请求超过一定时间没有响应就返回错误信息，不会无休止等待

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715172820438.png" alt="image-20210715172820438" style="zoom: 67%;" />

  超时处理仅仅是缓解了雪崩问题，释放时间是一秒，假如请求进来的频率是一秒两个，进来的速度没有释放的快，资源也会被耗尽

- **仓壁模式**：限定每个业务能使用的线程数，避免耗尽整个tomcat的资源，因此也叫线程隔离

  ![image-20210715172946352](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715172946352.png)

  船舱都会被隔板分离为多个独立空间，当船体破损时，只会导致部分空间进水，将故障控制在一定范围内，避免整个船体都被淹没

  与此类似，我们可以限定每个业务能使用的线程数，避免耗尽整个tomcat的资源，，因此也叫线程隔离

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715173215243.png" alt="image-20210715173215243" style="zoom:67%;" />

  这种模式也会造成资源浪费，比如服务C已经宕机了，每次请求还会占用线程资源

- **熔断降级**：由**断路器**统计业务执行的异常比例，如果超出阈值则会**熔断**该业务，拦截访问该业务的一切请求

  <img src="assets/image-20210715173428073.png" alt="image-20210715173428073" style="zoom:67%;" />

  断路器会统计访问某个服务的请求数量，异常比例，当发现访问服务D的请求异常比例过高时，认为服务D有导致雪崩的风险，会拦截访问服务D的一切请求，形成熔断。

- 流量控制：限制业务访问的QPS(每秒钟处理的请求数量)，避免服务因流量的突增而故障

  <img src="assets/image-20210715173555158.png" alt="image-20210715173555158" style="zoom: 67%;" />



### 1.2、总结

- 什么是雪崩问题
  - 微服务之间相互调用，因为调用链中的一个服务故障，引起整个链路都无法访问的情况
- 如何避免因瞬间高并发流量而导致服务故障
  - 流量控制
- 如何避免因服务故障引起的雪崩问题
  - 超时处理
  - 线程隔离
  - 熔断降级，也叫断路器模式



## 2、服务保护技术对比

在SpringCloud中支持多种服务保护技术：

- Netfix Hystrix
- Sentinel
- Resilience4J

早期比较流行的是Hystrix框架，但目前国内使用最广泛的还是阿里的Sentinel框架，下面是对比：

|                  | Sentinel                                       | Hystrix                      |
| ---------------- | ---------------------------------------------- | ---------------------------- |
| **隔离策略**     | 信号量隔离                                     | 线程池隔离/信号量隔离        |
| **熔断降级策略** | 基于慢调用比例或异常比例                       | 基于异常比例                 |
| 实时指标实现     | 滑动窗口                                       | 滑动窗口（基于RxJava）       |
| 规则配置         | 支持多种数据源                                 | 支持多种数据源               |
| 扩展性           | 多个扩展点                                     | 插件的形式                   |
| 基于注解的支持   | 支持                                           | 支持                         |
| **限流**         | 基于QPS，支持基于调用关系的限流                | 有限的支持，基于线程池隔离   |
| **流量整形**     | 支持慢启动、匀速排队模式                       | 不支持                       |
| 系统自适应保护   | 支持                                           | 不支持                       |
| **控制台**       | 开箱即用，可配置规则、查看秒级监控、机器发现等 | 不完善，只支持服务状态查看   |
| 常见的框架适配   | Servlet、SpringCloud、Dubbo、gRPC等            | Servlet、SpringCloud Netflix |

- 隔离策略区别
  - 线程池隔离：请求进入tomcat后，会给每一个被隔离的业务创建一个独立的线程池，比tomcat直接处理的方式多出成倍的线程，这种方式隔离性较好，但是随着线程数的增长，会给CPU造成负担，损失服务性能
  - 信号量隔离：请求进入tomcat后，不会创建独立线程池，而是统计当前业务使用了几个线程，做出限制，也就是限制每个业务能使用的线程数量，隔离性差一些
- 流量整形：让突发的流量变成匀速的流量



## 3、Sentinel介绍和安装



### 3.1、初识Sentinel

Sentinel是阿里巴巴开源的一款微服务流量控制组件。官网地址：https://sentinelguard.io/zh-cn/index.html

Sentinel具有以下特征（了解即可）：

- **丰富的应用场景**：Sentinel承接了阿里巴巴近10年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可应用等
- **完备的实时监控**：Sentinel提供实时的监控功能，可以在控制台中看到接入应用的单台机器秒级数据，甚至500台以下规模的集群的汇总运行情况
- **广泛的开源生态**：Sentinel提供开箱即用的与其他开源框架/库的整合模块，例如与SpringCloud、Dubbo、gRPC的整合。只需要引入相应的依赖并进行简单的配置即可快速地接入Sentinel
- **完善的SPI扩展点**：Sentinel提供简单易用、完善的SPI扩展接口。可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。



### 3.2、安装Sentinel

1. 下载

   Sentinel官方提供了UI控制台，方便我们对系统做限流设置。可以在github下载

2. 运行

   将jar包放到任意非中文目录，执行命令

   ```shell
   java -jar sentinel-dashboard-1.8.1.jar
   ```

   如果要修改Sentinel的默认端口、账户、密码，可以通过下列配置：

   | 配置项                           | 默认值   | 说明       |
   | -------------------------------- | -------- | ---------- |
   | server.port                      | 8080     | 服务端口   |
   | sentinel.dashboard.auth.username | sentinel | 默认用户名 |
   | sentinel.dashboard.auth.password | sentinel | 默认密码   |

   例如修改端口：

   ```sh
   java -Dserver.port=8090 -jar sentinel-dashboard-1.8.1.jar
   ```

3. 访问

   访问http://localhost:8080页面，就可以看到sentinel的控制台了：

   <img src="C:\Users\15840\AppData\Roaming\Typora\typora-user-images\image-20240414152152671.png" alt="image-20240414152152671" style="zoom: 67%;" />

   需要输入账号和密码，默认都是：sentinel

   登录后，发现一片空白，什么都没有，这是因为我们还没有与微服务整合：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414152211702.png" alt="image-20240414152211702" style="zoom:67%;" />

   

## 4、微服务整合Sentinel

我们在order-service中整合sentinel，并连接sentinel的控制台，步骤如下：

1. 引入Sentinel依赖

   ```xml
   <!--sentinel-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId> 
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. 配置控制台

   修改application.yaml文件，添加下面内容：

   ```yaml
   server:
     port: 8088
   spring:
     cloud: 
       sentinel:
         transport:
           dashboard: localhost:8080
   ```

3. 访问order-service的任意端点

   打开浏览器，访问http://localhost:8088/order/101，这样才能触发sentinel的监控。

   然后再访问sentinel的控制台，查看效果：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414174516864.png" alt="image-20240414174516864" style="zoom:80%;" />



# 二、流量控制

雪崩问题虽然有四种方案，但是限流是避免服务因突发的流量而发生故障，是对微服务雪崩问题的预防。



## 2.1、簇点链路

当请求进入微服务时，首先会访问DispatcherServlet，然后进入Controller、Service、Mapper，这样的一个调用链路就叫做**簇点链路**。簇点链路中被监控的每一个接口就是一个**资源**。

就是项目内的调用链路，链路中被监控的每个接口就是一个资源。默认情况下sentinel会监控SpringMVC的每一个端点（Endpoint，也就是Controller中的方法），因此SpringMVC的每一个端点（Endpoint）就是调用链路中的每一个资源。

例如，我们刚才访问的order-service中的OrderController中的端点：/order/{orderId}：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414204451249.png" alt="image-20240414204451249" style="zoom:80%;" />



流控、熔断等都是针对簇点链路中的资源来设置的，因此我们可以点击对应资源后面的按钮来设置规则：

- 流控：流量控制
- 降级：熔断降级
- 热点：热点参数限流，是限流的一种
- 授权：请求的权限控制



## 2.2、快速入门

点击资源/order/{orderId}后面的流控按钮，就可以弹出表单。

表单中可以填写限流规则，如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414205123698.png" alt="image-20240414205123698" style="zoom: 67%;" />

单机阈值的含义是限制/order/{orderId}这个资源的单机QPS，如果填1就是每秒只允许1次请求，超出的请求会被拦截并报错。



**练习**：

需求：给/order/{orderId}这个资源设置流控规则，QPS不能超过5，然后测试。

1. 首先在sentinel控制台添加限流规则

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715192455429.png" alt="image-20210715192455429" style="zoom:80%;" />

2. 利用jmeter测试

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715200635414.png" alt="image-20210715200635414" style="zoom:80%;" />

   规则含义是20个用户，2秒内运行完，QPS是10，超过了5。

   接下来开始运行，选中`流控入门，QPS<5`右键运行：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715200804594.png" alt="image-20210715200804594" style="zoom:80%;" />

   > 注意，不要点击菜单中的执行按钮来运行。

   结果：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210715200853671.png" alt="image-20210715200853671" style="zoom:80%;" />

   可以看到，成功的请求每次只有5个



## 2.3、流控模式

在添加限流规则时，点击高级选项，可以选择三种**流控模式**：

- 直接：统计当前资源的请求，触发阈值时对当前资源直接限流，也是默认的模式
- 关联：统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流
- 链路：统计从指定链路访问到本资源的请求，触发阈值时，对指定链路限流

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414210720860.png" alt="image-20240414210720860" style="zoom:67%;" />

快速入门测试的就是直接模式。



### 2.3.1、关联模式

**关联模式**：统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流

**配置规则**：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240414210926879.png" alt="image-20240414210926879" style="zoom: 80%;" />

**语法说明**：当/write资源访问量触发阈值时，就会对/read资源限流，避免影响/write资源

**使用场景**：比如用户支付时需要修改订单状态，同时用户要查询订单。查询和修改操作会争抢数据库锁，产生竞争。业务需求是优先支付和更新订单的业务，因此当修改订单业务触发阈值时，需要对查询订单业务限流



需求说明：

- 在OrderController新建两个端点：/order/query和/order/update，无需实现业务
- 配置流控规则，当/order/update资源被访问的QPS超过5时，对/order/query请求限流