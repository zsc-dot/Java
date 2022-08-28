# 1、初识MQ



## 1.1、同步和异步通讯

微服务间通讯有同步和异步两种方式：

同步通讯：就像打电话，需要实时响应。

异步通讯：就像发邮件，不需要马上回复。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822200600765.png" alt="image-20220822200600765" style="zoom: 67%;" />

两种方式各有优劣，打电话可以立即得到响应，但是你却不能跟多个人同时通话。发送邮件可以同时与多个人收发邮件，但是往往响应会有延迟。



### 1.1.1、同步通讯

我们之前学习的Feign调用就属于同步方式，虽然调用可以实时得到结果，但存在下面的问题：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822201122317.png" alt="image-20220822201122317" style="zoom: 50%;" />



### 1.1.2、异步通讯

异步调用则可以避免上述问题：

我们以购买商品为例，用户支付后需要调用订单服务完成订单状态修改，调用物流服务，从仓库分配响应的库存并准备发货。

在事件模式中，支付服务是事件发布者（publisher），在支付完成后只需要发布一个支付成功的事件（event），事件中带上订单id。

订单服务和物流服务是事件订阅者（Consumer），订阅支付成功的事件，监听到事件后完成自己业务即可。



为了解除事件发布者与订阅者之间的耦合，两者并不是直接通信，而是有一个中间人（Broker）。发布者发布事件到Broker，不关心谁来订阅事件。订阅者从Broker订阅事件，不关心谁发来的消息。



<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822202313649.png" alt="image-20220822202313649" style="zoom: 67%;" />



Broker 是一个像数据总线一样的东西，所有的服务要接收数据和发送数据都发到这个总线上，这个总线就像协议一样，让服务间的通讯变得标准和可控。



优势：

- 吞吐量提升：无需等待订阅者处理完成，响应更快速
- 故障隔离：服务没有直接调用，不存在级联失败问题
- 调用间没有阻塞，不会造成无效的资源占用
- 耦合度极低，每个服务都可以灵活插拔，可替换
- 流量削峰：不管发布事件的流量波动多大，都由Broker接收，订阅者可以按照自己的速度去处理事件



缺点：

- 架构复杂了，业务没有明显的流程线，不好管理
- 需要依赖于Broker的可靠、安全、性能



好在现在开源软件或云平台上 Broker 的软件是非常成熟的，比较常见的一种就是我们今天要学习的MQ技术。



### 1.1.3、总结

同步调用的优点：

- 时效性较强，可以立即得到结果

同步调用的问题：

- 耦合度高
- 性能和吞吐能力下降
- 有额外的资源消耗
- 有级联失败问题



异步通信的优点：

- 耦合度低
- 吞吐量提升
- 故障隔离
- 流量削峰

异步通信的缺点：

- 依赖于Broker的可靠性、安全性、吞吐能力
- 架构复杂了，业务没有明显的流程线，不好追踪管理



## 1.2、技术对比

MQ，中文是消息队列（MessageQueue），字面来看就是存放消息的队列。也就是事件驱动架构中的Broker。

比较常见的MQ实现：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka



几种常见MQ的对比：

|            | **RabbitMQ**            | **ActiveMQ**                   | **RocketMQ** | **Kafka**  |
| ---------- | ----------------------- | ------------------------------ | ------------ | ---------- |
| 公司/社区  | Rabbit                  | Apache                         | 阿里         | Apache     |
| 开发语言   | Erlang                  | Java                           | Java         | Scala&Java |
| 协议支持   | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议   | 自定义协议 |
| 可用性     | 高                      | 一般                           | 高           | 高         |
| 单机吞吐量 | 一般                    | 差                             | 高           | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                         | 毫秒级       | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                           | 高           | 一般       |



追求可用性：Kafka、 RocketMQ 、RabbitMQ

追求可靠性：RabbitMQ、RocketMQ

追求吞吐能力：RocketMQ、Kafka

追求消息低延迟：RabbitMQ、Kafka



# 2、快速入门

RabbitMQ是基于Erlang语言开发的开源消息通信中间件，官网地址：https://www.rabbitmq.com/



## 2.1、单机部署RabbitMQ

我们在Centos7虚拟机中使用Docker来安装。



### 2.1.1、下载镜像

方式一：在线拉取

```sh
docker pull rabbitmq:3-management
```



方式二：从本地加载

将镜像包上传到虚拟机中，使用命令加载镜像即可：

```sh
docker load -i mq.tar
```



### 2.1.2、安装MQ

执行下面的命令来运行MQ容器：

```sh
docker run \
 -e RABBITMQ_DEFAULT_USER=itcast \ # rabbitmq管理平台账户
 -e RABBITMQ_DEFAULT_PASS=123321 \ # rabbitmq管理平台密码
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \ # rabbitmq管理平台端口
 -p 5672:5672 \ # 消息通信端口
 -d \
 rabbitmq:3-management
```



### 2.1.3、MQ基本结构

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822211210890.png" alt="image-20220822211210890" style="zoom: 67%;" />



RabbitMQ中的一些角色：

- publisher：生产者
- consumer：消费者
- exchange个：交换机，负责消息路由
- queue：队列，存储消息
- virtualHost：虚拟主机，隔离不同租户的exchange、queue、消息的隔离



### 2.1.4、总结

RabbitMQ中的几个概念：

- channel：操作MQ的工具
- exchange：路由消息到队列中
- queue：缓存消息
- virtual host：虚拟主机，是对queue、exchange等资源的逻辑分组



## 2.2、RabbitMQ消息模型

RabbitMQ官方提供了5个不同的Demo示例，对应了不同的消息模型：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822212025501.png" alt="image-20220822212025501" style="zoom:67%;" />



## 2.3、入门案例

简单队列模式的模型图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220822212203569.png" alt="image-20220822212203569" style="zoom:67%;" />



官方的HelloWorld是基于最基础的消息队列模型来实现的，只包括三个角色：

- publisher：消息发布者，将消息发送到队列queue
- queue：消息队列，负责接受并缓存消息
- consumer：订阅队列，处理队列中的消息



### 2.3.1、publisher实现

思路：

- 建立连接
- 创建Channel
- 声明队列
- 发送消息
- 关闭连接和channel



代码实现：

```java
public class PublisherTest {
    @Test
    public void testSendMessage() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("192.168.150.101");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("itcast");
        factory.setPassword("123321");
        // 1.2.建立连接
        Connection connection = factory.newConnection();

        // 2.创建通道Channel
        Channel channel = connection.createChannel();

        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);

        // 4.发送消息
        String message = "hello, rabbitmq!";
        channel.basicPublish("", queueName, null, message.getBytes());
        System.out.println("发送消息成功：【" + message + "】");

        // 5.关闭通道和连接
        channel.close();
        connection.close();

    }
}
```



### 2.3.2、consumer实现

代码思路：

- 建立连接
- 创建Channel
- 声明队列
- 订阅消息



代码实现：

```java
public class ConsumerTest {

    public static void main(String[] args) throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("192.168.150.101");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("itcast");
        factory.setPassword("123321");
        // 1.2.建立连接
        Connection connection = factory.newConnection();

        // 2.创建通道Channel
        Channel channel = connection.createChannel();

        // 3.创建队列
        String queueName = "simple.queue";
        channel.queueDeclare(queueName, false, false, false, null);

        // 4.订阅消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) throws IOException {
                // 5.处理消息
                String message = new String(body);
                System.out.println("接收到消息：【" + message + "】");
            }
        });
        System.out.println("等待接收消息。。。。");
    }
}
```



### 2.3.3、总结

基本消息队列的消息发送流程：

1. 建立connection

2. 创建channel

3. 利用channel声明队列

4. 利用channel向队列发送消息

基本消息队列的消息接收流程：

1. 建立connection
2. 创建channel
3. 利用channel声明队列
4. 定义consumer的消费行为handleDelivery()
5. 利用channel将消费者与队列绑定



# 3、SpringAMQP

SpringAMQP是基于RabbitMQ封装的一套模板，并且还利用SpringBoot对其实现了自动装配，使用起来非常方便。

SpringAmqp的官方地址：https://spring.io/projects/spring-amqp

![image-20220823214921627](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220823214921627.png)



SpringAMQP提供了三个功能：

- 自动声明队列、交换机及其绑定关系
- 基于注解的监听器模式，异步接收消息
- 封装了RabbitTemplate工具，用于发送消息 



## 3.1、Basic Queue 简单队列模型

**案例：**

利用SpringAMQP实现HelloWorld中的基础消息队列功能。

流程如下：

1. 在父工程中引入spring-amqp的依赖

2. 在publisher服务中利用RabbitTemplate发送消息到simple.queue这个队列

3. 在consumer服务中编写消费逻辑，绑定simple.queue这个队列



在父工程mq-demo中引入依赖

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



### 3.1.1、消息发送

首先配置MQ地址，在publisher服务的application.yml中添加配置：

```yaml
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itcast # 用户名
    password: 123321 # 密码
```



然后在publisher服务中编写测试类SpringAmqpTest，并利用RabbitTemplate实现消息发送：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAmqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    public void testSimpleQueue() {
        // 队列名称
        String queueName = "simple.queue";
        // 消息
        String message = "hello, spring amqp!";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```



### 3.1.2、消息接收

首先配置MQ地址，在consumer服务的application.yml中添加配置：

```yaml
spring:
  rabbitmq:
    host: 192.168.150.101 # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itcast # 用户名
    password: 123321 # 密码
```



然后在consumer服务的`cn.itcast.mq.listener`包中新建一个类SpringRabbitListener，代码如下：

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueueMessage(String msg) {
        System.out.println("spring 消费者接收到消息：【" + msg + "】");
    }
}
```



启动consumer服务，然后在publisher服务中运行测试代码，发送MQ消息。



### 3.1.4、总结

什么是AMQP？

- 应用间消息通信的一种协议，与语言和平台无关。

SpringAMQP如何发送消息？

- 引入amqp的starter依赖
- 配置RabbitMQ地址
- 利用RabbitTemplate的convertAndSend方法

SpringAMQP如何接收消息？

- 引入amqp的starter依赖
- 配置RabbitMQ地址
- 定义类，添加@Component注解
- 类中声明方法，添加@RabbitListener注解，方法参数就是消息

注意：消息一旦消费就会从队列删除，RabbitMQ没有消息回溯功能



## 3.2、WorkQueue

Work queues，也被称为（Task queues），任务模型。简单来说就是**让多个消费者绑定到一个队列，共同消费队列中的消息**。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828201723398.png" alt="image-20220828201723398" style="zoom: 67%;" />

当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。

此时就可以使用work 模型，多个消费者共同处理消息处理，速度就能大大提高了。



**案例：**

模拟WorkQueue，实现一个队列绑定多个消费者。

基本思路如下：

1. 在publisher服务中定义测试方法，每秒产生50条消息，发送到simple.queue

2. 在consumer服务中定义两个消息监听者，都监听simple.queue队列

3. 消费者1每秒处理50条消息，消费者2每秒处理10条消息



### 3.2.1、消息发送

这次我们循环发送，模拟大量消息堆积现象。

在publisher服务中的SpringAmqpTest类中添加一个测试方法：

```java
/**
 * workQueue
 * 向队列中不停发送消息，模拟消息堆积。
 */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "simple.queue";
    // 消息
    String message = "hello, message_";
    for (int i = 0; i < 50; i++) {
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20);
    }
}
```



### 3.2.2、消息接收

要模拟多个消费者绑定同一个队列，我们在consumer服务的SpringRabbitListener中添加2个新的方法：

```java
@RabbitListener(queues = "simple.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20);
}

@RabbitListener(queues = "simple.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200);
}
```

注意到这个消费者sleep了1000秒，模拟任务耗时。



### 3.2.3、测试

启动ConsumerApplication后，在执行publisher服务中刚刚编写的发送测试方法testWorkQueue。

可以看到消费者1很快完成了自己的25条消息。消费者2却在缓慢的处理自己的25条消息。

也就是说消息是平均分配给每个消费者，并没有考虑到消费者的处理能力。这样显然是有问题的。



### 3.2.4、能者多劳

在spring中有一个简单的配置，可以解决这个问题。我们修改consumer服务的application.yml文件，添加配置：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```



### 3.2.5、总结

Work模型的使用：

- 多个消费者绑定到一个队列，同一条消息只会被一个消费者处理
- 通过设置prefetch来控制消费者预取的消息数量



## 3.3、发布订阅

发布订阅模式与之前案例的区别就是允许将同一消息发送给多个消费者。实现方式是加入了exchange（交换机）。

发布订阅的模型如图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828203252392.png" alt="image-20220828203252392" style="zoom:67%;" />



可以看到，在订阅模型中，多了一个exchange角色，而且过程略有变化：

- Publisher：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
- Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有以下3种类型：
  - Fanout：广播，将消息交给所有绑定到交换机的队列
  - Direct：定向，把消息交给符合指定routing key 的队列
  - Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列
- Consumer：消费者，与以前一样，订阅队列，没有变化
- Queue：消息队列也与以前一样，接收消息、缓存消息。



**Exchange（交换机）只负责转发消息，不具备存储消息的能力**，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！



## 3.4、Fanout

Fanout，英文翻译是扇出，在MQ中叫广播更合适。

Fanout Exchange 会将接收到的消息广播到每一个跟其绑定的queue。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828204400090.png" alt="image-20220828204400090" style="zoom: 67%;" />



在广播模式下，消息发送流程是这样的：

- 可以有多个队列
- 每个队列都要绑定到Exchange（交换机）
- 生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定
- 交换机把消息发送给绑定过的所有队列
- 订阅队列的消费者都能拿到消息



**案例：**

1. 在consumer服务中，利用代码声明队列、交换机，并将两者绑定

2. 在consumer服务中，编写两个消费者方法，分别监听fanout.queue1和fanout.queue2

3. 在publisher中编写测试方法，向itcast.fanout发送消息

我们的计划是这样的：

- 创建一个交换机 itcast.fanout，类型是Fanout
- 创建两个队列fanout.queue1和fanout.queue2，绑定到交换机itcast.fanout

![image-20220828205120056](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828205120056.png)



### 3.4.1、声明队列和交换机

SpringAMQP提供了声明交换机、队列、绑定关系的API，例如：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828205310140.png" alt="image-20220828205310140" style="zoom: 67%;" />



在consumer服务创建一个类，添加@Configuration注解，并声明FanoutExchange、Queue和绑定关系对象Binding：

```java
@Configuration
public class FanoutConfig {
    // itcast.fanout
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("itcast.fanout");
    }

    // fanout.queue1
    @Bean
    public Queue fanoutQueue1() {
        return new Queue("fanout.queue1");
    }

    // 绑定队列1和交换机 会按照类型和名称进行注入
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    // fanout.queue2
    @Bean
    public Queue fanoutQueue2() {
        return new Queue("fanout.queue2");
    }

    // 绑定队列2和交换机 会按照类型和名称进行注入
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange) {
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```



### 3.4.2、消息发送

在publisher服务的SpringAmqpTest类中添加测试方法：

```java
@Test
public void testFanoutExchange() {
    // 交换机名称
    String exchangeName = "itcast.fanout";
    // 消息
    String message = "hello, every one";
    // 发送
    rabbitTemplate.convertAndSend(exchangeName, "", message);
}
```



### 3.4.3、消息接收

在consumer服务的SpringRabbitListener类中，添加两个方法，分别监听fanout.queue1和fanout.queue2：

```java
@RabbitListener(queues = "fanout.queue1")
public void listenFanoutQueue1(String msg) {
    System.out.println("spring 消费者接收到fanout.queue1的消息：【" + msg + "】");
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String msg) {
    System.out.println("spring 消费者接收到fanout.queue2的消息：【" + msg + "】");
}
```



### 3.4.4、总结

交换机的作用是什么？

- 接收publisher发送的消息
- 将消息按照规则路由到与之绑定的队列
- 不能缓存消息，路由失败，消息丢失
- FanoutExchange的会将消息路由到每个绑定的队列

声明队列、交换机、绑定关系的Bean是什么？

- Queue
- FanoutExchange
- Binding



## 3.5、Direct

在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

Direct Exchange 会将接收到的消息根据规则路由到指定的Queue，因此称为路由模式（routes）。

- 每一个Queue都与Exchange设置一个BindingKey

- 发布者发送消息时，指定消息的RoutingKey

- Exchange将消息路由到BindingKey与消息RoutingKey一致的队列

![image-20220828213520186](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828213520186.png)



**案例：**

1. 利用@RabbitListener声明Exchange、Queue、RoutingKey

2. 在consumer服务中，编写两个消费者方法，分别监听direct.queue1和direct.queue2

3. 在publisher中编写测试方法，向itcast. direct发送消息



### 3.5.1、基于注解声明队列和交换机

基于@Bean的方式声明队列和交换机比较麻烦，Spring还提供了基于注解方式来声明。

1. 在consumer服务中，编写两个消费者方法，分别监听direct.queue1和direct.queue2，

2. 并利用@RabbitListener声明Exchange、Queue、RoutingKey

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg) {
    System.out.println("spring 消费者接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "itcast.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}
))
public void listenDirectQueue2(String msg) {
    System.out.println("spring 消费者接收到direct.queue2的消息：【" + msg + "】");
}
```



### 3.5.2、消息发送

在publisher服务的SpringAmqpTest类中添加测试方法：

```java
@Test
public void testDirectExchange() {
    // 交换机名称
    String exchangeName = "itcast.direct";
    // 消息
    String message = "hello, blue";
    // 发送
    rabbitTemplate.convertAndSend(exchangeName, "blue", message);
}
```



### 3.5.3、总结

描述下Direct交换机与Fanout交换机的差异？

- Fanout交换机将消息路由给每一个与之绑定的队列
- Direct交换机根据RoutingKey判断路由给哪个队列
- 如果多个队列具有相同的RoutingKey，则与Fanout功能类似

基于@RabbitListener注解声明队列和交换机有哪些常见注解？

- @Queue
- @Exchange



## 3.6、Topic



### 3.6.1、说明

`Topic`类型的`Exchange`与`Direct`相比，都是可以根据`RoutingKey`把消息路由到不同的队列。只不过`Topic`类型`Exchange`可以让队列在绑定`Routing key` 的时候使用通配符！

`Routingkey` 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： `item.insert`

通配符规则：

- `#`：匹配零个或多个词

- `*`：代指一个单词

举例：

- `item.#`：能够匹配`item.spu.insert` 或者 `item.spu`

- `item.*`：只能匹配`item.spu`

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828221720645.png" alt="image-20220828221720645" style="zoom: 67%;" />

解释：

- Queue1：绑定的是`china.#` ，因此凡是以 `china.`开头的`routing key` 都会被匹配到。包括china.news和china.weather
- Queue2：绑定的是`#.news` ，因此凡是以 `.news`结尾的 `routing key` 都会被匹配。包括china.news和japan.news



**案例：**

1. 并利用@RabbitListener声明Exchange、Queue、RoutingKey

2. 在consumer服务中，编写两个消费者方法，分别监听topic.queue1和topic.queue2

3. 在publisher中编写测试方法，向itcast. topic发送消息

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828221915514.png" alt="image-20220828221915514" style="zoom: 67%;" />



### 3.6.2、消息接收

1. 在consumer服务中，编写两个消费者方法，分别监听topic.queue1和topic.queue2，

2. 并利用@RabbitListener声明Exchange、Queue、RoutingKey

```java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void topicQueue1(String msg) {
    System.out.println("spring 消费者接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "itcast.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void topicQueue2(String msg) {
    System.out.println("spring 消费者接收到direct.queue2的消息：【" + msg + "】");
}
```



### 3.6.3、消息发送

在publisher服务的SpringAmqpTest类中添加测试方法：

```java
@Test
public void testTopicExchange() {
    // 交换机名称
    String exchangeName = "itcast.topic";
    // 消息
    String message = "JAVA开发测试";
    // 发送
    rabbitTemplate.convertAndSend(exchangeName, "china.news", message);
}
```



### 3.6.4、总结

描述下Direct交换机与Topic交换机的差异？

- Topic交换机接收的消息RoutingKey必须是多个单词，以 `.` 分割
- Topic交换机与队列绑定时的bindingKey可以指定通配符
- `#`：代表0个或多个词
- `*`：代表1个词



## 3.7、消息转换器

在SpringAMQP的发送方法中，接收消息的类型是Object，也就是说我们可以发送任意对象类型的消息，SpringAMQP会帮我们序列化为字节后发送。

只不过，默认情况下Spring采用的序列化方式是JDK序列化。众所周知，JDK序列化存在下列问题：

- 数据体积过大
- 有安全漏洞
- 可读性差

我们来测试一下。



### 3.7.1、测试默认转换器

现在consumer中利用@Bean声明一个队列：

```java
@Bean
public Queue objectQueue() {
    return new Queue("object.queue");
}
```



在publisher中发消息测试：

```java
@Test
public void testObjectQueue() {
    Map<String, Object> msg = new HashMap<>();
    msg.put("name", "Jack");
    msg.put("age", 21);
    rabbitTemplate.convertAndSend("object.queue", msg);
}
```



发送消息后查看控制台：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828223906816.png" alt="image-20220828223906816" style="zoom:67%;" />

显然，JDK序列化方式并不合适。我们希望消息体的体积更小、可读性更高，因此可以使用JSON方式来做序列化和反序列化。



### 3.7.2、消息发送配置JSON转换器

Spring的对消息对象的处理是由org.springframework.amqp.support.converter.MessageConverter来处理的。

而默认实现是SimpleMessageConverter，基于JDK的ObjectOutputStream完成序列化。

如果要修改只需要定义一个MessageConverter 类型的Bean即可。推荐用JSON方式序列化，步骤如下：

- 在publisher和consumer两个服务中都引入依赖：

  ```xml
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
  </dependency>
  ```

- 在publisher服务声明MessageConverter

  ```java
  @Bean
  public MessageConverter messageConverter() {
      return new Jackson2JsonMessageConverter();
  }
  ```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220828224729802.png" alt="image-20220828224729802" style="zoom: 80%;" />



### 3.7.3、消息接收配置JSON转换器

- 在consumer服务定义MessageConverter

  ```java
  @Bean
  public MessageConverter messageConverter() {
      return new Jackson2JsonMessageConverter();
  }
  ```

- 定义一个消费者，监听object.queue队列并消费消息

  ```java
  @RabbitListener(queues = "object.queue")
  public void objectQueue(Map<String, Object> msg) {
      System.out.println("接收到object.queue的消息：" + msg);
  }
  ```



### 3.7.4、总结

SpringAMQP中消息的序列化和反序列化是怎么实现的？

- 利用MessageConverter实现的，默认是JDK的序列化
- 注意发送方与接收方必须使用相同的MessageConverter