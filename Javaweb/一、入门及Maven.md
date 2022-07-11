# 一、基本概念

## 1.1、前言

web开发：

- web，网页的意思
- 静态web
  - html、css等静态资源
  - 提供给所有人看的数据始终不会发生变化
- 动态web
  - 提供给所有人看的数据会发生变化，每个人在不同的时间不用的地点，看到的信息各不相同
  - 比如：淘宝，几乎所有的网站都是动态的
  - 技术栈：Servlet、JSP、ASP、PHP

在Java 中，动态web资源开发的技术统称为JavaWeb



## 1.2、web应用程序

web应用程序：可以提供浏览器访问的程序

- a.html、b.html....多个web资源，这些web资源可以被外界访问，对外界提供服务
- 能访问到的任何一个页面或者资源，都存在于某一台计算机上
- URL：统一资源定位符
- 这些统一的web资源会被放在同一个文件夹下，就是一个web应用程序。但需要Tomcat服务器启动和外界访问
- 一个web应用由多部分组成（静态web，动态web）
  - html、css、js
  - jsp、servlet
  - Java程序
  - jar包
  - 配置文件

web应用程序编写完毕后，若想提供给外界访问：需要一个服务器来统一管理



## 1.3、静态web

- *.html这些都是网页的后缀，如果服务器上一直存在这些东西，就可以通过网络直接进行读取。

![1630809223173](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630809223173.png)

- 静态web存在的缺点
  - web页面无法动态更新，所有用户看到的都是同一个页面
    - 轮播图，点击特效：伪动态
    - JavaScript
    - VBScript
  - 无法和数据库交互(数据无法持久化，用户无法交互)



## 1.4、动态web

页面会动态展示：web页面展示的效果因人而异

![1630809672524](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/%E5%8A%A8%E6%80%81web.png)

- 缺点
  - 假如服务器的动态web资源出现了错误，需要编写后台程序，重新发布
    - 停机维护
- 优点
  - web页面可以动态更新，所有用户看到的不是同一个页面
  - 可以与数据库交互(数据库持久化)



# 二、web服务器

## 2.1、技术讲解

**ASP**：

- 微软：国内最早流行的就是ASP
- 在HTML中嵌入了VB的脚本，ASP+COM
- 在ASP开发中，基本一个页面都有几千行的业务代码，页面极其换乱
- 维护成本高
- C#
- IIS

**PHP**：

- 开发速度快，功能强大，跨平台，代码简单
- 无法承载大访问量的情况

**JSP/Servlet**：

- B/S：浏览器和服务器

- C/S：客户端和服务器
- sun公司主推的B/S架构
- 基于Java语言的 (所有的大公司，或者一些开源的组件，都是用Java写的)
- 可以承载三高问题带来的影响
- 语法像ASP ， ASP-->JSP , 加强市场强度



## 2.2、web服务器

服务器是一种被动的操作，用来处理用户的一些请求和给用户一些响应信息

**IIS**：微软的，跑ASP...，windows中自带

**Tomcat**：

最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现。

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和 并发访问用户不是很多的场合下被普遍使用

Tomcat 实际上运行JSP 页面和Servlet



# 三、Tomcat

## 3.1、安装Tomcat

 tomcat官网：http://tomcat.apache.org/

解压后可以直接使用



## 3.2、Tomcat启动和配置

文件夹作用：

![1630931658911](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630931658911.png)



启动、关闭Tomcat

![1630931916570](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630931916570.png)



访问测试地址：http://localhost:8080

 可能遇到的问题：

1. Java环境变量没有配置
2. 闪退问题：需要配置兼容性
3. 乱码问题：配置文件中设置



## 3.3、配置

![1630932105177](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630932105177.png)



修改端口：修改8081位指定端口

![1630932851585](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630932851585.png)



修改localhost：需要在C:\Windows\System32\drivers\etc目录下映射127.0.0.1为自定义地址

![1630932684404](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630932684404.png)



**面试题**：

请你谈谈网站是如何进行访问的

1. 输入一个域名，回车
2. 检查本机的 C:\Windows\System32\drivers\etc\hosts配置文件下有没有这个域名映射
   1. 有：直接返回对应的ip地址，这个地址中，有我们需要访问的web程序，可以直接访问
   2. 没有：去DNS服务器找，找到的话就返回，找不到就返回找不到

![1630933171508](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630933171508.png)

3. 可以配置一下环境变量（可选性）



## 3.4、发布一个web网站

将自己写的网站，放到服务器(Tomcat)中指定的web应用的文件夹（webapps）下，就可以访问了

网站该有的结构：

```
-- webapps ：Tomcat服务器的web目录
    - ROOT
    - ssl：网站的目录名
        - WEB-INF
            -classes : java程序
            -lib：web应用所依赖的jar包
            -web.xml ：网站配置文件
    - index.html 默认的首页
    - static
        -css
        	-style.css
    -js
    -img
    -.....
```



# 四、HTTP

## 4.1、什么是HTTP

HTTP（超文本传输协议）是一个简单的请求-响应协议，它通常运行在TCP之上。

- 文本有html、字符串...
- 超文本有图片、音乐、视频、定位、地图...它们都可以通过http来传输
- http默认端口为80

HTTPS

- 默认端口：443



## 4.2、两个时代

- http1.0
  - HTTP/1.0：客户端可以与web服务器连接后，只能获得一个web资源，断开连接
- http2.0
  - HTTP/1.1：客户端可以与web服务器连接后，可以获得多个web资源。



## 4.3、HTTP请求

- 客户端---发请求(Request)---服务器



### 4.3.1、请求行

```
Request URL:https://www.baidu.com      请求地址
Request Method:GET get方法/post        请求方法
Status Code:200 OK                     状态码：200
Remote（远程） Address:14.215.177.39:443    远程地址(真实访问的地址)
```

- 请求行中的请求方式：GET
- 请求方式：Get、Post、HEAD、DELETE、PUT、TRACT…
- get：请求能够携带的参数比较少，大小有限制，会在浏览器的URL地址栏显示数据内容，不安全，但高效 
- post：请求能够携带的参数没有限制，大小没有限制，不会在浏览器的URL地址栏显示数据内容，安全，但不高效



### 4.3.2、请求头

```
Accept:text/html     告诉服务器，浏览器所支持的数据类型
Accept-Encoding:gzip, deflate, br  告诉服务器，浏览器支持哪种编码格式
Accept-Language:zh-CN,zh;q=0.9    告诉服务器，浏览器的语言环境
Cache-Control:max-age=0    缓存控制
Connection:keep-alive   告诉服务器，请求完成是断开还是保持连接
```



## 4.4、HTTP响应

- 服务器---响应---客户端

### 4.4.1、响应头

```
Cache-Control:private   服务器通过这个头，可以控制浏览器不缓存数据
Connection:Keep-Alive   服务器通过这个头，响应完是保持链接还是关闭链接
Content-Encoding:gzip   文档的编码格式
Content-Type:text/html  表示后面的文档属于什么MIME类型
```

### 4.4.2、响应体

```
Accept：告诉浏览器，它所支持的数据类型
Accept-Encoding：支持哪种编码格式 GBK UTF-8 GB2312 ISO8859-1
Accept-Language：告诉浏览器，它的语言环境
Cache-Control：缓存控制
Connection：告诉浏览器，请求完成是断开还是保持连接
HOST：主机..../.
Refresh：告诉客户端，多久刷新一次；
Location：让网页重新定位；
```



### 4.4.3、响应状态码

- 200：请求响应成功 200
- 3xx：请求重定向
  - 重定向：重新到给的新位置去
- 4xx：找不到资源  404
  - 资源不存在
- 5xx：服务器代码错误  500   
  - 502：网关错误



# 五、Maven

在Javaweb开发中，需要使用大量的jar包，我们手动去导入

Maven可以自动导入和配置这个jar包



## 5.1、Maven项目架构管理工具

Maven的核心思想：约定大于配置

有约束，就不要违反



## 5.2、下载安装Maven

官网：https://maven.apache.org/

![1630936464720](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630936464720.png)

下载完成后解压即可



## 5.3、配置环境变量

 在系统环境变量中

配置如下配置：

- M2_HOME       maven目录下的bin目录
- MAVEN_HOME      maven的目录
- 在系统的path中配置       %MAVEN_HOME%\bin 

![1630936596169](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630936596169.png)

测试Maven是否安装成功



## 5.4、阿里云镜像

![1630936642579](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630936642579.png)

- 镜像：mirrors
  - 作用：加速我们的下载
- 国内建议使用阿里云的镜像

```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*,!jeecg,!jeecg-snapshots</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```



## 5.5、本地仓库

在本地的仓库，远程仓库

建立一个本地仓库：localRepository，路径为本地仓库地址

```xml
<localRepository>D:\Environment\apache-maven-3.6.2\maven-repo</localRepository>
```



## 5.6、在IDEA中使用Maven

1. 启动IDEA

2. 创建一个Maven项目

   ![1630937560795](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630937560795.png)

   ![1630937736134](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630937736134.png)

   ![1630937797972](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630937797972.png)

   ![1630937849159](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630937849159.png)

3. 等待项目搭建完成

   ![1630938072942](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630938072942.png)

4. IDEA中的Maven设置

   ![1630938334879](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630938334879.png)

   ![1630938419665](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630938419665.png)

5. 到这里Maven在IDEA中的配置就结束了



## 5.7、创建一个普通的Maven项目

在创建Maven项目时不勾选模板

![1630938808700](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630938808700.png)



只有在web应用下才会有的

![1630938914167](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630938914167.png)



## 5.8、标记文件夹功能

![1630939092388](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939092388.png)

![1630939156136](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939156136.png)

![1630939262543](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939262543.png)

![1630939301047](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939301047.png)



## 5.9、在IDEA中配置Tomcat

![1630939368828](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939368828.png)

![1630939408784](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939408784.png)

![1630939457036](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939457036.png)

![1630939724271](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939724271.png)

![1630939777587](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939777587.png)



解决警告问题

**警告产生的原因：访问一个网站，必须要指定一个文件夹的名字**

![1630939990295](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630939990295.png)

![1630940030390](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630940030390.png)

![1630940227167](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630940227167.png)



## 5.10、pom文件

pom.xml是Maven的核心配置文件

![1630940459007](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630940459007.png)



```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- Maven版本和头文件 -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!-- 这里就是之前配置的GAV -->
  <groupId>com.study</groupId>
  <artifactId>javaweb-01-maven</artifactId>
  <version>1.0-SNAPSHOT</version>
  <!-- 项目的打包方式
        jar：Java应用
        war：JavaWeb应用
   -->
  <packaging>war</packaging>

  <!-- 配置 -->
  <properties>
    <!-- 项目的默认构建编码 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 编译版本 -->
    <maven.compiler.source>9.0</maven.compiler.source>
    <maven.compiler.target>9.0</maven.compiler.target>
  </properties>

  <!-- 项目依赖 -->
  <dependencies>
    <!-- 具体依赖的jar包 -->
    <!-- Maven的高级之处在于，会自动导入这个jar包所依赖的其他jar包 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <!-- 项目构建用的东西 -->
  <build>
    <finalName>javaweb-01-maven</finalName>
    <pluginManagement>
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>
```



由于Maven的约定大于配置，之后可能会遇到写的配置文件无法被导出或者生效的问题

比如：配置文件没有写在maven声明的目录中，打包的时候就会打不进去

解决方案：

```xml
<!--在build中配置resources，来防止资源导出失败的问题-->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```



![1630941223753](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630941223753.png)

![1630941255595](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1630941255595.png)
