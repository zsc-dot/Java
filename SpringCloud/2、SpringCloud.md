# 1、Nacos配置管理

Nacos除了可以做注册中心，同样可以做配置管理来使用。



## 1.1、统一配置管理

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。

这个时候就需要一种统一配置管理方案，可以集中管理所有实例的配置。

**配置更改热更新**：完成配置文件改动后，微服务不需要重启，这些配置就能立马生效。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814194648349.png" alt="image-20220814194648349" style="zoom:67%;" />

Nacos一方面可以将配置集中管理，另一方可以在配置变更时，及时通知微服务，实现配置的热更新。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814194952564.png" alt="image-20220814194952564" style="zoom:67%;" />



### 1.1.1、在nacos中添加配置文件

![image-20220814210101113](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814210101113.png)



在弹出的表单中填写配置信息，Data ID必须是唯一的：

![image-20220814210128280](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814210128280.png)



> 注意：项目的核心配置，需要热更新的配置才有放到nacos管理的必要。基本不会变更的一些配置还是保存在微服务本地比较好。



### 1.1.2、从微服务拉取配置

微服务要拉取nacos中管理的配置，并且与本地的application.yml配置合并，才能完成项目启动。

但如果尚未读取application.yml，又如何得知nacos地址呢？

因此spring引入了一种新的配置文件：bootstrap.yaml文件，会在application.yml之前被读取，配置获取的步骤如下：

![image-20220814210218783](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814210218783.png)



> 注意：与Nacos地址和配置文件有关的信息都应该放在bootstrap.yaml文件中



1. 引入Nacos的配置管理客户端依赖：

   首先，在user-service服务中，引入nacos-config的客户端依赖：

   ```xml
   <!--nacos配置管理依赖-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

2. 在userservice中的resource目录添加一个bootstrap.yml文件，这个文件是引导文件，优先级高于application.yml：

   然后，在user-service中添加一个bootstrap.yaml文件，内容如下：

   ```yaml
   spring:
     application:
       name: userservice # 服务名称
     profiles:
       active: dev #开发环境，这里是dev
     cloud:
       nacos:
         server-addr: localhost:8848 # Nacos地址
         config:
           file-extension: yaml # 文件后缀名
   ```

   这里会根据spring.cloud.nacos.server-addr获取nacos地址，再根据`${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}`作为文件id，来读取配置。

   本例中，就是去读取`userservice-dev.yaml`。

3. 读取nacos配置

   在user-service中的UserController中添加业务逻辑，读取pattern.dateformat配置：

   ```java
   @Slf4j
   @RestController
   @RequestMapping("/user")
   public class UserController {
       @Autowired
       private UserService userService;
   
       @Value("${pattern.dateformat}")
       private String dateformat;
   
       @GetMapping("now")
       public String now() {
           return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateformat));
       }
   }
   ```

   在页面访问，可以看到当前时间已经被转为了指定格式。



### 1.1.3、总结

将配置交给Nacos管理的步骤：

1. 在Nacos中添加配置文件
2. 在微服务中引入nacos的config依赖
3. 在微服务中添加bootstrap.yml，配置nacos地址、当前环境、服务名称、文件后缀名。这些决定了程序启动时去nacos读取哪个文件



## 1.2、配置热更新

我们最终的目的，是修改nacos中的配置后，微服务中无需重启即可让配置生效，也就是**配置热更新**。

Nacos中的配置文件变更后，微服务无需重启就可以感知。不过需要通过下面两种配置实现：



### 1.2.1、方式一

在@Value注入的变量所在类上添加注解@RefreshScope：

```java
@Slf4j
@RestController
@RequestMapping("/user")
@RefreshScope
public class UserController {
    @Autowired
    private UserService userService;

    @Value("${pattern.dateformat}")
    private String dateformat;

    @GetMapping("now")
    public String now() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateformat));
    }
}
```



### 1.2.2、方式二

使用@ConfigurationProperties注解代替@Value注解。

在user-service服务中，添加一个类，读取patterrn.dateformat属性：

```java
@Component
@Data
@ConfigurationProperties(prefix = "pattern")  // 前缀名加上变量名，就可以去找到对应的配置文件
public class PatternProperties {
    private String dateformat;
}
```



在UserController中使用这个类代替@Value：

```java
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @Autowired
    private PatternProperties patternProperties;

    @GetMapping("now")
    public String now() {
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(patternProperties.getDateformat()));
    }
}
```



### 1.2.3、总结

Nacos配置更改后，微服务可以实现热更新，方式：

- 通过@Value注解注入，结合@RefreshScope来刷新
- 通过@ConfigurationProperties注入，自动刷新

注意事项：

- 不是所有的配置都适合放到配置中心，维护起来比较麻烦
- 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置

配置更新后，服务会有日志提示已经变更。



## 1.3、配置共享

其实微服务启动时，会去nacos读取多个配置文件，例如：

- `[spring.application.name]-[spring.profiles.active].yaml`，例如：userservice-dev.yaml
- `[spring.application.name].yaml`，例如：userservice.yaml

而`[spring.application.name].yaml`不包含环境，因此可以被多个环境共享。

无论profile如何变化，`[spring.application.name].yaml`这个文件一定会加载，因此多环境共享配置可以写入这个文件

![image-20220814220220038](D:\Markdown笔记\asssets\image-20220814220220038.png)



下面我们通过案例来测试配置共享



### 1.3.1、添加一个环境共享配置

我们在nacos中添加一个userservice.yaml文件：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814220723588.png" alt="image-20220814220723588" style="zoom:67%;" />



### 1.3.2、在user-service中读取共享配置

在user-service服务中，修改PatternProperties类，读取新添加的属性：

```java
@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
    private String envShareValue;
}
```

在user-service服务中，修改UserController，添加一个方法：

```java
@Slf4j
@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    @Autowired
    private PatternProperties patternProperties;

    @GetMapping("prop")
    public PatternProperties properties() {
        return patternProperties;
    }
}

```



### 1.3.3、运行两个UserApplication，使用不同的profile

修改UserApplication2这个启动项，改变其profile值：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814220937975.png" alt="image-20220814220937975" style="zoom:67%;" />



这样，UserApplication(8081)使用的profile是dev，UserApplication2(8082)使用的profile是test。

启动UserApplication和UserApplication2。

访问http://localhost:8081/user/prop，结果：

![image-20220814221048302](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814221048302.png)

访问http://localhost:8082/user/prop，结果：

![image-20220814221112099](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814221112099.png)



可以看出来，不管是dev，还是test环境，都读取到了envSharedValue这个属性的值。



### 1.3.4、配置共享的优先级

当nacos、服务本地同时出现相同属性时，优先级有高低之分：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814221240542.png" alt="image-20220814221240542" style="zoom:67%;" />



### 1.3.5、指定不同微服务之间共享配置文件

1. 方式一

   ```yaml
   spring:
     application:
       name: userservice # 服务名称
     profiles:
       active: dev # 环境
     cloud:
       nacos:
         server-addr: localhost:8848 # Nacos地址
       config:
         file-extension: yaml # 文件后缀名
         shared-configs: # 多微服务间共享的配置列表
           - dataId: common.yaml # 要共享的配置文件id
   ```

2. 方式二

   ```yaml
   spring:
     application:
       name: userservice # 服务名称
     profiles:
       active: dev # 环境
     cloud:
       nacos:
         server-addr: localhost:8848 # Nacos地址
       config:
         file-extension: yaml # 文件后缀名
         extends-configs: # 多微服务间共享的配置列表
           - dataId: extend.yaml # 要共享的配置文件id
   ```



### 1.3.6、总结

微服务会从nacos读取的配置文件：

- [服务名]-[spring.profile.active].yaml，环境配置

- [服务名].yaml，默认配置，多环境共享

优先级：

- [服务名]-[环境].yaml >[服务名].yaml > 本地配置



## 1.4、搭建Nacos集群

Nacos生产环境下一定要部署为集群状态。



### 1.4.1、集群结构图

官方给出的Nacos集群图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814221658456.png" alt="image-20220814221658456" style="zoom: 80%;" />

其中包含3个nacos节点，然后一个负载均衡器代理3个Nacos。这里负载均衡器可以使用nginx。

我们计划的集群结构：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814221736904.png" alt="image-20220814221736904" style="zoom: 50%;" />

三个nacos节点的地址：

| 节点   | ip            | port |
| ------ | ------------- | ---- |
| nacos1 | 192.168.150.1 | 8845 |
| nacos2 | 192.168.150.1 | 8846 |
| nacos3 | 192.168.150.1 | 8847 |



### 1.4.2、搭建集群

搭建集群的基本步骤：

- 搭建数据库，初始化数据库表结构
- 下载nacos安装包
- 配置nacos
- 启动nacos集群
- nginx反向代理



1. 初始化数据库

   Nacos默认数据存储在内嵌数据库Derby中，不属于生产可用的数据库。

   官方推荐的最佳实践是使用带有主从的高可用数据库集群，主从模式的高可用数据库可以参考**传智教育**的后续高手课程。

   这里我们以单点的数据库为例来讲解。

   首先新建一个数据库，命名为nacos，而后导入下面的SQL：

   ```sql
   CREATE TABLE `config_info` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(255) DEFAULT NULL,
     `content` longtext NOT NULL COMMENT 'content',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
     `app_name` varchar(128) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     `c_desc` varchar(256) DEFAULT NULL,
     `c_use` varchar(64) DEFAULT NULL,
     `effect` varchar(64) DEFAULT NULL,
     `type` varchar(64) DEFAULT NULL,
     `c_schema` text,
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_aggr   */
   /******************************************/
   CREATE TABLE `config_info_aggr` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(255) NOT NULL COMMENT 'group_id',
     `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
     `content` longtext NOT NULL COMMENT '内容',
     `gmt_modified` datetime NOT NULL COMMENT '修改时间',
     `app_name` varchar(128) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
   
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_beta   */
   /******************************************/
   CREATE TABLE `config_info_beta` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL COMMENT 'content',
     `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_info_tag   */
   /******************************************/
   CREATE TABLE `config_info_tag` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
     `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL COMMENT 'content',
     `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
     `src_user` text COMMENT 'source user',
     `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = config_tags_relation   */
   /******************************************/
   CREATE TABLE `config_tags_relation` (
     `id` bigint(20) NOT NULL COMMENT 'id',
     `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
     `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
     `data_id` varchar(255) NOT NULL COMMENT 'data_id',
     `group_id` varchar(128) NOT NULL COMMENT 'group_id',
     `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
     `nid` bigint(20) NOT NULL AUTO_INCREMENT,
     PRIMARY KEY (`nid`),
     UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
     KEY `idx_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = group_capacity   */
   /******************************************/
   CREATE TABLE `group_capacity` (
     `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
     `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
     `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
     `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
     `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
     `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
     `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
     `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_group_id` (`group_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = his_config_info   */
   /******************************************/
   CREATE TABLE `his_config_info` (
     `id` bigint(64) unsigned NOT NULL,
     `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
     `data_id` varchar(255) NOT NULL,
     `group_id` varchar(128) NOT NULL,
     `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
     `content` longtext NOT NULL,
     `md5` varchar(32) DEFAULT NULL,
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
     `src_user` text,
     `src_ip` varchar(50) DEFAULT NULL,
     `op_type` char(10) DEFAULT NULL,
     `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
     PRIMARY KEY (`nid`),
     KEY `idx_gmt_create` (`gmt_create`),
     KEY `idx_gmt_modified` (`gmt_modified`),
     KEY `idx_did` (`data_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
   
   
   /******************************************/
   /*   数据库全名 = nacos_config   */
   /*   表名称 = tenant_capacity   */
   /******************************************/
   CREATE TABLE `tenant_capacity` (
     `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
     `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
     `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
     `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
     `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
     `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
     `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
     `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
     `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
     `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
   
   
   CREATE TABLE `tenant_info` (
     `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
     `kp` varchar(128) NOT NULL COMMENT 'kp',
     `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
     `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
     `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
     `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
     `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
     `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
     PRIMARY KEY (`id`),
     UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
     KEY `idx_tenant_id` (`tenant_id`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
   
   CREATE TABLE `users` (
   	`username` varchar(50) NOT NULL PRIMARY KEY,
   	`password` varchar(500) NOT NULL,
   	`enabled` boolean NOT NULL
   );
   
   CREATE TABLE `roles` (
   	`username` varchar(50) NOT NULL,
   	`role` varchar(50) NOT NULL,
   	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
   );
   
   CREATE TABLE `permissions` (
       `role` varchar(50) NOT NULL,
       `resource` varchar(255) NOT NULL,
       `action` varchar(8) NOT NULL,
       UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
   );
   
   INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
   
   INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
   ```

2. 下载nacos

   nacos在GitHub上有下载地址：https://github.com/alibaba/nacos/tags，可以选择任意版本下载。

   本例中才用1.4.1版本。

3. 配置Nacos

   解压nacos压缩包后，进入nacos的conf目录，修改配置文件cluster.conf.example，重命名为cluster.conf。

   然后添加内容：

   ```yaml
   127.0.0.1:8845 # 由于是在本机搭建，所以ip就是127.0.0.1，如果是生产环境，就要是三个真实的服务器ip
   127.0.0.1.8846
   127.0.0.1.8847
   ```

   然后修改application.properties文件，添加数据库配置：

   ```properties
   spring.datasource.platform=mysql
   
   db.num=1
   
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
   db.user.0=root
   db.password.0=123
   ```

4. 启动

   将nacos文件夹复制三份，分别命名为：nacos1、nacos2、nacos3。然后分别修改三个文件夹中的application.properties。

   ```properties
   server.port=8845
   server.port=8846
   server.port=8847
   ```

   然后分别启动三个nacos节点：

   ```sh
   startup.cmd
   ```

5. nginx反向代理

   修改conf/nginx.conf文件，配置如下：

   ```nginx
   upstream nacos-cluster {
       server 127.0.0.1:8845;
   	server 127.0.0.1:8846;
   	server 127.0.0.1:8847;
   }
   
   server {
       listen       80;
       server_name  localhost;
   
       location /nacos {
           proxy_pass http://nacos-cluster;
       }
   }
   ```

   而后在浏览器访问：http://localhost/nacos即可。

   代码中application.yml文件配置如下：

   ```yaml
   spring:
     cloud:
       nacos:
         server-addr: localhost:80 # Nacos地址
   ```



### 1.4.3、优化

- 实际部署时，需要给做反向代理的nginx服务器设置一个域名，这样后续如果有服务器迁移nacos的客户端也无需更改配置。
- Nacos的各个节点应该部署到多个不同服务器，做好容灾和隔离



### 1.4.4、总结

集群搭建步骤：

- 搭建MySQL集群并初始化数据库表
- 下载解压nacos
- 修改集群配置（节点信息）、数据库配置
- 分别启动多个nacos节点
- nginx反向代理



# 2、Feign远程调用

**RestTemplate方式调用存在的问题**

先来看我们以前利用RestTemplate发起远程调用的代码：

```java
String url = "http://userservice/user/" + order.getUserId();
User user = restTemplate.getForObject(url, User.class);
```

存在下面的问题：

- 代码可读性差，编程体验不统一
- 参数复杂的URL难以维护



Feign是一个声明式[]的http客户端，官方地址：https://github.com/OpenFeign/feign

其作用就是帮助我们优雅的实现http请求的发送，解决上面提到的问题。

声明式的优势：在事务中，最开始需要手动开启事务。而在Spring声明式事务中，只需要在配置文件中告诉Spring要给谁加事务，定义好规则就可以了。



## 2.1、Feign替代RestTemplate

Fegin的使用步骤如下：

### 2.1.1、引入依赖

我们在order-service服务的pom文件中引入feign的依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



### 2.1.2、添加注解

在order-service的启动类添加`@EnableFeignClients`注解开启Feign的功能：

```java
@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```



### 2.1.3、编写Feign的客户端

在order-service中新建一个接口，内容如下：

```java
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

这个客户端主要是基于SpringMVC的注解来声明远程调用的信息，比如：

- 服务名称：userservice
- 请求方式：GET
- 请求路径：/user/{id}
- 请求参数：Long id
- 返回值类型：User

这样，Feign就可以帮助我们发送http请求，无需自己使用RestTemplate来发送了。



### 2.1.4、测试

修改order-service中的OrderService类中的queryOrderById方法，使用Feign客户端代替RestTemplate：

```java
@Service
public class OrderService {
    @Autowired
    private OrderMapper orderMapper;
    @Autowired
    private UserClient userClient;

        public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2. 利用Fegin远程调用
        User user = userClient.findById(order.getUserId());
        // 3. 封装 user对象到 order
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```



### 2.1.5、总结

Feign的使用步骤：

- 引入依赖
- 添加@EnableFeignClients注解
- 编写FeignClient接口
- 使用FeignClient中定义的方法代替RestTemplate



## 2.2、自定义配置

Feign运行自定义配置来覆盖默认配置，可以修改的配置如下：

| 类型                   | 作用             | 说明                                                   |
| ---------------------- | ---------------- | ------------------------------------------------------ |
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign. Contract        | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign. Retryer         | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的@Bean覆盖默认Bean即可。

一般我们需要配置的就是日志级别。

下面以日志为例来演示如何自定义配置。

配置Feign日志有两种方式。



### 2.2.1、配置文件方式

基于配置文件修改feign的日志级别可以针对单个服务：

```yaml
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

也可以针对所有服务：

```yaml
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```



而日志的级别分为四种：

- NONE：不记录任何日志信息，这是默认值。
- BASIC：仅记录请求的方法，URL以及响应状态码和执行时间
- HEADERS：在BASIC的基础上，额外记录了请求和响应的头信息
- FULL：记录所有请求和响应的明细，包括头信息、请求体、元数据。



### 2.2.2、Java代码方式

也可以基于Java代码来修改日志级别，先声明一个Bean，然后声明一个Logger.Level的对象：

```java
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```



如果要**全局生效**，将其放到启动类的@EnableFeignClients这个注解中：

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) 
```



如果是**局部生效**，则把它放到对应的@FeignClient这个注解中：

```java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) 
```



### 2.2.3、总结

Feign的日志配置：

1. 方式一是配置文件，feign.client.config.xxx.loggerLevel
   - 如果xxx是default则代表全局
   - 如果xxx是服务名称，例如userservice则代表某服务
2. 方式二是java代码配置Logger.Level这个Bean
   - 如果在@EnableFeignClients注解声明则代表全局
   - 如果在@FeignClient注解中声明则代表某服务



## 2.3、Feign使用优化

Feign底层发起http请求，依赖于其它的框架。其底层客户端实现包括：

- URLConnection：默认实现，不支持连接池
- Apache HttpClient ：支持连接池
- OKHttp：支持连接池

因此提高Feign的性能主要手段就是使用**连接池**代替默认的URLConnection。

因此优化Feign的性能主要包括：

- 使用连接池代替默认的URLConnection
- 日志级别，最好用basic或none



这里我们用Apache的HttpClient来演示。



### 2.3.1、引入依赖

在order-service的pom文件中引入Apache的HttpClient依赖：

```xml
<!--httpClient的依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```



### 2.3.2、配置连接池

在order-service的application.yml中添加配置：

```yaml
feign:
  client:
    config:
      default: # default全局的配置
        loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息
  httpclient:
    enabled: true # 开启feign对HttpClient的支持
    max-connections: 200 # 最大的连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
```

接下来，在FeignClientFactoryBean中的loadBalance方法中打断点：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816213753004.png" alt="image-20220816213753004" style="zoom:80%;" />

Debug方式启动order-service服务，可以看到这里的client，底层就是Apache HttpClient：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816213911077.png" alt="image-20220816213911077" style="zoom: 80%;" />



### 2.3.3、总结

Feign的优化：

- 日志级别尽量用basic
- 使用HttpClient或OKHttp代替URLConnection
  - 引入feign-httpClient依赖
  - 配置文件开启httpClient功能，设置连接池参数



## 2.4、最佳实践

所谓最近实践，就是使用过程中总结的经验，最好的一种使用方式。

自习观察可以发现，Feign的客户端与服务提供者的controller代码非常相似：

feign客户端：

```java
@FeignClient("userservice")
public interface UserClient {
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

UserController：

```java
@GetMapping("/{id}")
public User queryById(@PathVariable("id") Long id) {
    return userService.queryById(id);
}
```

有没有一种办法简化这种重复的代码编写呢？



### 2.4.1、继承方式

给消费者的FeignClient和提供者的controller定义统一的父接口作为标准：

- 定义一个API接口，利用定义方法，并基于SpringMVC注解做声明
- Feign客户端和Controller都集成改接口

![image-20220816214603379](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816214603379.png)



优点：

- 简单
- 实现了代码共享

缺点：

- 服务提供方、服务消费方紧耦合
- 参数列表中的注解映射并不会继承，因此Controller中必须再次声明方法、参数列表、注解



### 2.4.2、抽取方式

将Feign的Client抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用。

例如，将UserClient、User、Feign的默认配置都抽取到一个feign-api包中，所有微服务引用该依赖包，即可直接使用。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816215146148.png" alt="image-20220816215146148" style="zoom:80%;" />



### 2.4.3、实现基于抽取的最佳实践

实现最佳实践方式二的步骤如下：

1. 创建一个module，命名为feign-api，然后引入feign的starter依赖

2. 将order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中

3. 在order-service中引入feign-api的依赖

4. 修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包

5. 重启测试，会发现服务报失败了

   ![image-20220816220636021](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816220636021.png)

   这是因为UserClient现在在cn.itcast.feign.clients包下，而order-service的@EnableFeignClients注解是在cn.itcast.order包下，不在同一个包，无法扫描到UserClient。



### 2.4.4、解决扫描包问题

当定义的FeignClient不在SpringBootApplication的扫描包范围时，这些FeignClient无法使用。有两种方式解决：

- 方式一：

  指定Feign应该扫描的包：

  ```java
  @EnableFeignClients(basePackages = "cn.itcast.feign.clients")
  ```

- 方式二：

  指定需要加载的Client接口：

  ```java
  @EnableFeignClients(clients = {UserClient.class})
  ```



### 2.4.5、总结

Feign的最佳实践：

- 让controller和FeignClient继承同一接口
- 将FeignClient、POJO、Feign的默认配置都定义到一个项目中，供所有消费者使用

不同包的FeignClient的导入有两种方式：

- 在@EnableFeignClients注解中添加basePackages，指定FeignClient所在的包
- 在@EnableFeignClients注解中添加clients，指定具体FeignClient的字节码



# 3、Gateway服务网关

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等响应式编程和事件流技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。



## 3.1、为什么需要网关

Gateway网关是我们服务的守门神，所有微服务的统一入口。

网关的**核心功能特性**：

- 身份认证、权限校验
- 服务路由、负载均衡
- 请求限流

架构图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816221503694.png" alt="image-20220816221503694" style="zoom: 67%;" />

**身份认证、权限校验**：网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**服务路由、负载均衡**：一切请求都必须先经过gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**请求限流**：当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大。



在SpringCloud中网关的实现包括两种：

- gateway
- zuul

Zuul是基于Servlet的实现，属于阻塞式编程。

而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程的实现，具备更好的性能。



## 3.2、gateway快速入门

下面，我们就演示下网关的基本路由功能。基本步骤如下：

1. 创建SpringBoot工程gateway，引入网关依赖
2. 编写启动类
3. 编写基础配置和路由规则
4. 启动网关服务进行测试



### 3.2.1、创建gateway服务，引入依赖

创建新的module，引入SpringCloudGateway的依赖和nacos的服务发现依赖：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816222128762.png" alt="image-20220816222128762" style="zoom: 80%;" />

引入依赖：

```xml
<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<!--nacos服务发现依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```



### 3.2.2、编写启动类

```java
@SpringBootApplication
public class GatewayApplication {
	public static void main(String[] args) {
		SpringApplication.run(GatewayApplication.class, args);
	}
}
```



### 3.2.3、编写基础配置和路由规则

创建application.yml文件，内容如下：

```yaml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: user-service # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡(loadBalance缩写)，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
```



我们将符合`Path` 规则的一切请求，都代理到 `uri`参数指定的地址。

本例中，我们将 `/user/**`开头的请求，代理到`lb://userservice`，lb是负载均衡，根据服务名拉取服务列表，实现负载均衡。



### 3.2.4、重启测试

重启网关，访问http://localhost:10010/user/1时，符合`/user/**`规则，请求转发到uri：http://userservice/user/1，得到了结果：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816223337850.png" alt="image-20220816223337850" style="zoom:80%;" />



### 3.2.5、网关路由的流程图

整个访问的流程如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220816223505848.png" alt="image-20220816223505848" style="zoom: 67%;" />



### 3.2.6、总结

网关搭建步骤：

1. 创建项目，引入nacos服务发现和gateway依赖
2. 配置application.yml，包括服务基本信息、nacos地址、路由

路由配置包括：

1. 路由id：路由的唯一标示
2. 路由目标（uri）：路由的目标地址，http代表固定地址，lb代表根据服务名负载均衡
3. 路由断言（predicates）：判断路由的规则，
4. 路由过滤器（filters）：对请求或响应做处理