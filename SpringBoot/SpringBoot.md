

# 1、第一个SpringBoot程序

idea创建SpringBoot项目时，集成了SpringBoot官网

导包：

```xml
<!-- 导入web包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.1</version>
</dependency>
```

启动类：

```java
// 本身就是spring的一个组件
// 程序主入口
@SpringBootApplication
public class HellworldApplication {
    public static void main(String[] args) {
        SpringApplication.run(HellworldApplication.class, args);
    }
}
```

配置端口号：

```
server.port=8081
```

# 2、运行原理

自动配置的原理分析

## 1、pom.xml

**父依赖：**

其中它主要是依赖一个父项目，主要是管理项目的资源过滤及插件。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

点`spring-boot-starter-parent`进去，发现还有一个父依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.3</version>
</parent>
```

点`spring-boot-dependencies`，里面才是真正管理SpringBoot应用里面所有依赖版本的地方，SpringBoot的版本控制中心

- spring-boot-dependencies：核心依赖在父工程中
- 在写或者引入一些Springboot依赖的时候，不需要指定版本，就是因为有这些版本仓库

**启动器：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

- 启动器就是Springboot的启动场景

- 比如spring-boot-starter-web，会自动导入web环境所有依赖

- Springboot会将所有的功能场景，都变成一个个的启动器

- 如果需要使用什么功能，就只需要找到对应的启动器就可以了 starter，可以在官网找到

## 2、主启动类

```java
// 标注这个类是一个springboot的应用：启动类下的所有资源被导入
@SpringBootApplication
public class Springboot01HelloworldApplication {
	public static void main(String[] args) {
		// 将springboot应用启动
		SpringApplication.run(Springboot01HelloworldApplication.class, args);
	}
}
```

### 1、@SpringBootApplication

作用：标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用。

进入这个注解，还可以看到上面有很多其他注解。

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {...}
```

> **@ComponentScan**

这个注解在Spring中很重要，它对应XML配置中的元素。

作用：自动扫描并加载符合条件的组件或者bean ， 将这个bean定义加载到IOC容器中。

> **@SpringBootConfiguration**

作用：SpringBoot的配置类 ，标注在某个类上 ， 表示这是一个SpringBoot的配置类。

进去这个注解

```java
@Configuration
public @interface SpringBootConfiguration {}

// 再点进去可以得到@Component
@Component
public @interface Configuration {}
```

@Configuration：说明这是一个配置类 ，配置类就是对应Spring的xml 配置文件。

@Component ：说明启动类本身也是Spring中的一个组件，负责启动应用。

回到 SpringBootApplication 注解中继续看。

> **@EnableAutoConfiguration**

开启自动配置功能

以前需要自己配置的东西，现在SpringBoot可以自动配置。

 @EnableAutoConfiguration告诉SpringBoot开启自动配置功能，这样自动配置才能生效。

点`@EnableAutoConfiguration`进去

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
```

- @AutoConfigurationPackage：自动配置包

  点这个注解进去

  ```java
  @Import({Registrar.class}) // 导入选择器
  public @interface AutoConfigurationPackage {}
  // @import：Spring底层注解@import，给容器中导入一个组件
  // Registrar.class 作用：将主启动类的所在包及包下面所有子包里面的所有组件扫描到Spring容器 ；
  ```

- @Import({AutoConfigurationImportSelector.class})：给容器导入组件

  AutoConfigurationImportSelector：自动配置导入选择器。

  1. 这个类中有一个这样的方法

     ```java
     // 获得候选的配置
     protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
         //这里的getSpringFactoriesLoaderFactoryClass（）方法，返回的就是最开始看的启动自动导入配置文件的注解类；EnableAutoConfiguration
         List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
         Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
         return configurations;
     }
     ```
     
2. 这个方法又调用了 SpringFactoriesLoader 类的静态方法，进入SpringFactoriesLoader类 loadFactoryNames() 方法
  
   ```java
     public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
         ClassLoader classLoaderToUse = classLoader;
         if (classLoader == null) {
             classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
         }
     
         String factoryTypeName = factoryType.getName();
         // 这里又调用了 loadSpringFactories 方法
         return (List)loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
     }
   ```

  3. 继续点击查看 loadSpringFactories 方法
  
     ```java
     private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {
         // 获得classLoader，我们返回可以看到这里得到的就是EnableAutoConfiguration标注的类本身
         Map<String, List<String>> result = (Map)cache.get(classLoader);
         if (result != null) {
             return result;
         } else {
             HashMap result = new HashMap();
     
             try {
                 // 去获取一个资源 "META-INF/spring.factories"
                 Enumeration urls = classLoader.getResources("META-INF/spring.factories");
     
                 // 将读取到的资源遍历，封装成为一个Properties
                 while(urls.hasMoreElements()) {
                     URL url = (URL)urls.nextElement();
                     UrlResource resource = new UrlResource(url);
                     Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                     Iterator var6 = properties.entrySet().iterator();
     
                     while(var6.hasNext()) {
                         Entry<?, ?> entry = (Entry)var6.next();
                         String factoryTypeName = ((String)entry.getKey()).trim();
                         String[] factoryImplementationNames = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                         String[] var10 = factoryImplementationNames;
                         int var11 = factoryImplementationNames.length;
     
                         for(int var12 = 0; var12 < var11; ++var12) {
                             String factoryImplementationName = var10[var12];
                             ((List)result.computeIfAbsent(factoryTypeName, (key) -> {
                                 return new ArrayList();
                             })).add(factoryImplementationName.trim());
                         }
                     }
                 }
     
                 result.replaceAll((factoryType, implementations) -> {
                     return (List)implementations.stream().distinct().collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList));
                 });
                 cache.put(classLoader, result);
                 return result;
             } catch (IOException var14) {
                 throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var14);
             }
         }
     }
     ```
  
  4. 发现一个多次出现的文件： META-INF/spring.factories ，这是自动配置的核心文件。全局搜索。
  
     打开spring.factories ， 可以看到很多自动配置的文件，这就是自动配置根源。
  
     ![image-20220807095912373](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220807095912373.png)
  
     我们在上面的自动配置类随便找一个打开看看，比如：WebMvcAutoConfiguration
  
     ![image-20220807100222650](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220807100222650.png)
  
     可以看到这些一个个的都是JavaConfig配置类，而且都注入了一些Bean。



**思考：**为什么这么多自动配置，有的却没有生效？

需要导入对应的start才能有作用。配置类中的核心注解：@ConditionalOnCXXX，如果这个里面的所有条件都满足，才会生效。

![自动配置原理分析](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.png)

**结论：**springboot所有的自动配置都是在启动的时候扫描并加载：spring.factories所有的自动配置类都在这里面，但是不一定生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，自动装配就会生效，然后就配置成功了

1、springboot在启动的时候，从类路径下/META-INF/spring.factories获取指定的值

2、将这些自动配置的类导入容器，自动配置类就会生效，进行自动配置

3、以前需要自动配置的东西，现在springboot做了

4、整个JavaEE解决方案和自动配置的东西spring-boot-autoconfigure-2.2.0.RELEASE.jar这个包下

5、它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器

6、容器中也会存在非常多的xxxautoConfiguration文件，就是这些类给容器中导入了这个场景需要的所有组件，并自动配置

7、有了自动配置类，就免去了手动编写配置文件的操作



### 2、SpringApplication

看似只是运行了一个main方法，实际是开启了一个服务。

```java
@SpringBootApplication
public class SpringbootApplication {
    public static void main(String[] args) {
    	SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

这个类主要做了四件事情：

1. 推断应用的类型是普通的项目还是Web项目
2. 查找并加载所有可用初始化器 ， 设置到initializers属性中
3. 找出所有的应用程序监听器，设置到listeners属性中
4. 推断并设置main方法的定义类，找到运行的主类

关于SpringBoot的理解：

1. 自动装配
2. run()方法



全面接管SpringMVC的配置



## 3、配置文件和spring.factories

### 1、源码案例分析

我们以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理；

```java
// 表示这是一个配置类
@Configuration(proxyBeanMethods = false)
// 自动配置属性
//启动指定类的ConfigurationProperties功能；
    //进入这个HttpProperties查看，将配置文件中对应的值和HttpProperties绑定起来；
    //并把HttpProperties加入到ioc容器中
@EnableConfigurationProperties({HttpProperties.class})
// Spring底层@Conditional注解
	// 根据不同的条件判断，如果满足指定的条件，整个配置类里面的配置就会生效；
	// 这里的意思就是判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication(type = Type.SERVLET)
// 判断当前项目有没有这个类CharacterEncodingFilter：SpringMVC中解决乱码的过滤器；
@ConditionalOnClass({CharacterEncodingFilter.class})
// 判断配置文件中是否存在某个配置：spring.http.encoding.enabled；
	// 如果不存在，判断也是成立的
	// 即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
@ConditionalOnProperty(
    prefix = "server.servlet.encoding",
    value = {"enabled"},
    matchIfMissing = true
)
public class HttpEncodingAutoConfiguration {
    // 它已经和SpringBoot的配置文件映射了
    private final Encoding properties;

    // 只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(HttpProperties properties) {
        this.properties = properties.getEncoding();
    }

    // 给容器中添加一个组件，这个组件的某些值需要从properties中获取
    @Bean
    @ConditionalOnMissingBean // 判断容器没有这个组件
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.RESPONSE));
        return filter;
    }

    @Bean
    public HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
        return new HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer(this.properties);
    }

    static class LocaleCharsetMappingsCustomizer implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {
        private final Encoding properties;

        LocaleCharsetMappingsCustomizer(Encoding properties) {
            this.properties = properties;
        }

        public void customize(ConfigurableServletWebServerFactory factory) {
            if (this.properties.getMapping() != null) {
                factory.setLocaleCharsetMappings(this.properties.getMapping());
            }

        }

        public int getOrder() {
            return 0;
        }
    }
}
```



```java
//从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "spring.http")
public class HttpProperties {
    // .....
}
```

在yaml配置文件中打出前缀试试：

![image-20220807153143266](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220807153143266.png)

在配置文件里能配置的东西，都存在一个固有的规律。

xxxAutoConfiguration：自动配置类，有默认值，给容器添加组件；

xxxProperties ：封装配置文件中相关属性，就可以使用自定义的配置了



### 2、自动装配精髓

1. SpringBoot启动会加载大量的自动配置类
2. 要注意需要的功能有没有在SpringBoot默认写好的自动配置类当中
3. 这个自动配置类中到底配置了哪些组件；（只要要用的组件存在在其中，就不需要再手动配置了）
4. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。只需要在配置文件中指定这些属性的值即可。



### 3、了解@Conditional

了解完自动装配的原理后，我们来关注一个细节问题，自动配置类必须在一定的条件下才能生效。

@Conditional 派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

![image-20220807155837123](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220807155837123.png)

那么多的自动配置类，必须在一定的条件下才能生效；也就是说，我们加载了这么多的配置类，但不是所有的都生效了。

怎么知道哪些自动配置类生效？

可以通过启用 debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪 些自动配置类生效；

```yaml
#开启springboot的调试类
debug: true
```

打印出的日志分类：

- Positive matches:（自动配置类启用的：正匹配）
- Negative matches:（没有启动，没有匹配成功的自动配置类：负匹配）
- Unconditional classes: （没有条件的类）



# 3、SpringBoot配置

SpringBoot使用一个全局的配置文件 ， 配置文件名称是固定的

- application.properties
- 语法结构 ：key=value
- application.yml
- 语法结构 ：key：空格 value

## 3.1、yaml基础语法

语法要求严格

1、空格不能省略

2、以缩进来控制层级关系，只要是左边对齐的一列数据都是同一个层级的。

3、属性和值的大小写都是十分敏感的。

```yaml
# 普通的key: value
name: zhang

# 对象
student:
  name: zhang
  age: 3

# 行内写法
student: {name: zhang, age: 3}

# 数组
pets:
  -cat
  -dog
  -pig

pets: [cat,dog,pig]
```

```properties
# 只能保存键值对
name=zahng

student.name = zhang
student.age = 3
```

## 3.2、属性赋值

**第一种：**yaml

yaml可以直接给实体类赋值

Dog.java

```java
@Component
public class Dog {
    @Value("王五")
    private String name;
    @Value("3")
    private Integer age;
}
```

person

```java
@Component
/*
@ConfigurationProperties作用：
将配置文件中配置的每一个属性的值，映射到这个组件中；
告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定
参数 prefix = “person” : 将配置文件中的person下面的所有属性一一对应
*/
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> list;
    private Dog dog;
}
```

yaml文件注入

```yaml
person:
  name: zhang
  age: 3
  happy: false
  birth: 2020/02/02
  maps: {k1: v1, K2: v2}
  list: [code,music,girl]
  dog:
    name: wangcai
    age: 3
```

**第二种：**properties

properties文件

```properties
name=zhang
```

实体类

```java
@Component
// 加载指定的配置文件
@PropertySource(value = "classpath:application.properties")
public class Person {
    // SPEL表达式取出配置文件的值
    @Value("${name}")
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> list;
    private Dog dog;
}
```

## 3.3、JSR303校验

```java
@Component
// 把yml中的值导入
@ConfigurationProperties(prefix = "person")
@Validated // 数据校验
public class Person {
	// 表示该属性只能注入邮箱格式的数据
    @Email(message = "邮箱格式错误")
    private String name;
}
```

## 3.4、激活yaml配置文件

```yaml
# springboot的多环境配置，可以选择激活哪一个配置文件
spring:
  profiles:
    active:
```



# 4、SpringBoot Web开发

jar：webapp

自动装配

- xxxAutoConfiguration：向容器中自动配置组件
- xxxProperties：自动配置类，装配配置文件中自定义的一些内容

要解决的问题：

- 导入静态资源
- 首页
- jsp，模板引擎Thymeleaf
- 装配和扩展SpringMVC
- 增删改查
- 拦截器
- 国际化



总结：

1. 在SpringBoot中，可以使用以下方式处理静态资源
   - webjars
   - public，static，/**，resources     localhost:8080/
2. 优先级：resources > static(默认) > public



# 5、Thymeleaf模板引擎

用来写一个模板，jsp就是一个模板引擎

1、导包

```xml
<!-- Thymeleaf模板引擎，都是基于3.x开发 -->
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
</dependency>
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-java8time</artifactId>
</dependency>
```

结论：只要需要使用thymeleaf，只需要导入对应的依赖就可以了。将html页面放在templates目录下即可

2、html导入约束

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```



# 6、SpringData

配置文件：

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.jdbc.Driver
```



# 7、整合Mybatis

整合包

mybatis-spring-boot-starter



# 8、SpringSecurity

**安全框架：Shiro、SpringSecurity**：除了类和名字不一样，其他类似

认证，授权

权限：

- 功能权限
- 访问权限
- 菜单权限

以前需要用拦截器和过滤器，使用大量的原生代码，冗余



Aop：横切 -- 配置类

Spring Security 是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型，他可以实现强大的Web安全控制，对于安全控制，我们仅需要引入spring-boot-starter-security 模块，进行少量的配置，即可实现强大的安全管理！

记住几个类：

- WebSecurityConfigurerAdapter：自定义Security策略
- AuthenticationManagerBuilder：自定义认证策略
- @EnableWebSecurity：开启WebSecurity模式    @Enablexxx开启某个功能

Spring Security的两个主要目标是 “认证” 和 “授权”（访问控制）。

**“认证”（Authentication）**

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。

身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。

 **“授权” （Authorization）**

授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。

这个概念是通用的，而不是只在Spring Security 中存在。





# 9、Shiro（以后再看）

## 1.1、概述

- ApacheShiro是java的安全（权限）框架
- 可以完成认证，授权，加密，会话管理，web集成，缓存等



# 10、任务

## 10.1、异步任务

```
@Service
public class AsyncService {

    // 告诉Spring这是一个异步的方法
    @Async
    public void hello() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据正在处理");
    }

}
```

```
@RestController
public class AsyncController {
    @Autowired
    AsyncService asyncService;

    @RequestMapping("/hello")
    public String hello() {
        asyncService.hello(); // 停止三秒
        return "ok";
    }
}
```

```
// 开启异步注解功能
@EnableAsync
@SpringBootApplication
public class Springboot09TestApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot09TestApplication.class, args);
	}

}
```

返回给前端的数据不需要等线程的睡眠三秒，调用接口时可以立即返回数据。后台的打印需要等三秒

## 10.2、邮件发送

1、导包

```xml
<!-- 导入邮件启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

2、

```java
class Springboot09TestApplicationTests {
	@Autowired
	JavaMailSenderImpl mailSender;

	@Test
	void contextLoads() {
		// 简单的邮件
		SimpleMailMessage mailMessage = new SimpleMailMessage();
		mailMessage.setSubject("通知");// 邮件主题
		mailMessage.setText("你下午不用去考试了");// 内容
		mailMessage.setTo("941700809@qq.com");
		mailMessage.setFrom("1584095767@qq.com");
		mailSender.send(mailMessage);
	}


	@Test
	void contextLoads2() throws MessagingException {
		// 复杂的邮件
		MimeMessage mimeMessage = mailSender.createMimeMessage();
		// 组装
		MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,true); // 开启支持文件发送
		helper.setSubject("憨憨");
		helper.setText("<p style = 'color:red'>新年快乐</p>", true); // 开启支持解析html
		helper.addAttachment("红包.jpg", new File("E:\\money.jpg"));// 附件
		helper.setTo("941700809@qq.com");
		helper.setFrom("1584095767@qq.com");
		mailSender.send(mimeMessage);
	}

}
```

## 10.3、定时任务

```
核心接口：
	TaskScheduler  任务调度者
	TaskExecutor  任务执行者
@EnableScheduling // 开启定时功能的注解
@Scheduled  // 什么时候执行

Cron表达式
```

```
// 开启异步注解功能
@EnableAsync
@EnableScheduling // 开启定时功能的注解
@SpringBootApplication
public class Springboot09TestApplication {

	public static void main(String[] args) {
		SpringApplication.run(Springboot09TestApplication.class, args);
	}

}
```

```
@Service
public class ScheduledService {

    // 在特定的时间执行这个方法
    // cron表达式
    // 秒 分 时 日 月 周几
    /*
        0 24 16 * * ?  每天16点24分0秒执行一次
        0 0/5 10,18 * * ?  每天的10点和18点，每个5分钟执行一次
        0 15 10 ? * 1-6 每个月的周一到周六的10点15分执行一次
     */
    @Scheduled(cron = "0 24 16 * * ?")
    public void hello () {
        System.out.println("你被执行了");
    }

}
```



# 11、分布式 Dubbo+Zookeeper

## 1.1、分布式

分布式系统是由一组通过网络进行通信、为了完成共同的任务而协调工作的计算及节点组成的系统。

它的出现是为了用廉价的、普通的机器完成单个计算机无法完成的计算、存储任务。

其目的是利用更多的机器，处理更多的数据

## 1.2、RPC

Http是基于网络的通信协议

RPC是远程过程调用，，是一种技术思想。允许程序调用另一个地址空间（即另一台电脑）的过程或函数

例如两台服务器A，B。一个应用部署在A服务器上，要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络表达调用的语义和传达调用的数据。

RPC两个核心：通讯，序列化

序列化：数据传输需要转换



## 2.1、Zookeeper

是一个分布式的，开放源码的分布式应用程序协调服务。是一个为分布式应用提供一致性服务的软件，提供的功能包括： 配置维护、名字服务、分布式同步、组服务等 

即一个注册中心



dubbo-admin是一个监控管理后台，可以查看哪些服务被消费了

Dubbo：是一个jar包



生产者：

1. 导包

   ```xml
   <!-- 导入dubbo和zookeeper的依赖 -->
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.3</version>
   </dependency>
   <!-- zkclient -->
   <dependency>
       <groupId>com.github.sgroschupf</groupId>
       <artifactId>zkclient</artifactId>
       <version>0.1</version>
   </dependency>
   
   <!-- 日志会冲突 -->
   <!-- 引入zookeeper -->
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-framework</artifactId>
       <version>2.12.0</version>
   </dependency>
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-recipes</artifactId>
       <version>2.12.0</version>
   </dependency>
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.4.14</version>
       <!--排除这个slf4j-log4j12-->
       <exclusions>
           <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

2. 接口和实现类

```
public interface TicketService {
    public String getTicket();
}
```

```
// zookeeper：服务注册与发现

@Service // 可以被扫描到，在项目一启动，就自动注册到注册中心
@Component // 使用了dubbo，尽量不要用service
public class TicketServiceImpl implements TicketService {
    @Override
    public String getTicket() {
        return "hello";
    }
}
```

3.配置文件

```
server.port=8081

# 服务应用名字
dubbo.application.name=provider-server
# 注册中心地址
dubbo.registry.address=zookeeper://127.0.0.1:2181
# 哪些服务要被注册
dubbo.scan.base-packages=com.study.service
```



消费者：

1.导包

2.实现类

```
@Service
public class UserService {
    // 想拿到provider提供的票，要去注册中心拿到服务
    @Reference // 引用，pom坐标，或定义路径相同的接口名
    TicketService ticketService;

    public void byTicket() {
        String ticket = ticketService.getTicket();
        System.out.println("在注册中心拿到一张票" + ticket);
    }
}
```

3.配置

```
server.port=8082

# 消费者去哪里拿服务，需要暴露自己的名字
dubbo.application.name=consumer-server
# 注册中心的地址
dubbo.registry.address=zookeeper://127.0.0.1:2181
```



步骤：

前提：注册中心开启

1. 提供者提供服务
   1. 导入依赖
   2. 配置注册中心的地址，以及服务发现名，和要扫描的包
   3. 在想要被注册的服务上面增加一个dubbo包下的注解Service
2. 消费者如何消费
   1. 导入依赖
   2. 配置注册中心的地址，配置自己的服务名
   3. 从远程注入服务 @Reference