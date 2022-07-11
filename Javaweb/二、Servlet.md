# 六、Servlet

Servlet接口在Sun公司有两个默认的实现类：HttpServlet和GenericServlet

## 6.1、Servlet简介

- Servlet是sun公司开发动态web的一门技术
- Sun公司在这些API中提供一个接口，叫做Servlet。如果想开发一个Servlet程序，只需要完成两个步骤：
  - 编写一个类，实现Servlet接口
  - 把开发好的Java类部署到Web服务器中

把实现了Servlet接口的Java程序叫做Servlet



## 6.2、Hello Servlet

1. 构建一个普通的Maven项目，删掉里面的`src`目录，以后的项目都在这个里面建立`Moudel`，这个空的工程就是Maven主工程

2. 关于Maven父子工程的理解：

   父项目中会有

   ```xml
   <modules>
       <module>servlet-01</module>
   </modules>
   ```

   子项目中会有

   ```xml
   <parent>
       <artifactId>javaweb-02-servlet</artifactId>
       <groupId>com.study</groupId>
       <version>1.0-SNAPSHOT</version>
   </parent>
   ```

   父项目中的java子项目可以直接使用

   ```java
   // 用java中的关系来描述：就是继承
   son extends father 
   ```

3. Maven环境优化

   1. 修改`web.xml`为最新的
   2. 将maven的结构搭建完整

4. 编写一个Servlet程序

   1. 编写一个普通类
   2. 实现Servlet接口，这里直接继承HttpServlet

   ![1635772350406](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635772350406.png)

   3. 代码

      ```java
      public class HelloServlet extends HttpServlet {
          // 由于 Get或者 Post只是请求实现的不同方式，可以相互调用，业务逻辑都一样
          @Override
          protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
      //        ServletOutputStream outputStream = resp.getOutputStream();
              PrintWriter writer = resp.getWriter(); // 响应流
              writer.print("Hello Servlet");
          }
      
          @Override
          protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
              doGet(req, resp);
          }
      }
      ```

5. 编写Servlet的映射

   为什么需要映射：我们写的是JAVA程序，但是要通过浏览器访问，而浏览器需要连接web服务器，所以我们需要在web服务中注册我们写的Servlet，还需要给它一个浏览器能够访问的路径

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee     http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
            version="3.1">
       
       <!-- 注册Servlet -->
       <servlet>
           <servlet-name>hello</servlet-name>
           <servlet-class>com.study.servlet.HelloServlet</servlet-class>
       </servlet>
       
       <!-- Servlet的请求路径 -->
       <servlet-mapping>
           <servlet-name>hello</servlet-name>
           <url-pattern>/hello</url-pattern>
       </servlet-mapping>
       
   </web-app>
   ```

   

6. 配置Tomcat

   注意：配置项目发布的路径就可以了

7. 启动测试



## 6.3、Servlet原理

Servlet是由Web服务器调用，Web服务器在收到浏览器请求后，会：

![1635778386198](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635778386198.png)



## 6.4、Mapping

1. 一个Servlet可以指定一个映射路径

   ```xml
   <!-- 注册Servlet -->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.study.servlet.HelloServlet</servlet-class>
   </servlet>
   
   <!-- Servlet的请求路径 -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   ```

2. 一个Servlet可以指定多个映射路径

   ```xml
   <!-- 注册Servlet -->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.study.servlet.HelloServlet</servlet-class>
   </servlet>
   
   <!-- Servlet的请求路径 -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello1</url-pattern>
   </servlet-mapping>
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello2</url-pattern>
   </servlet-mapping>
   ```

3. 一个Servlet可以指定通用映射路径

   ```xml
   <!-- 注册Servlet -->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.study.servlet.HelloServlet</servlet-class>
   </servlet>
   
   <!-- Servlet的请求路径 -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello/*</url-pattern>
   </servlet-mapping>
   
   <!-- 默认请求，会替代掉index.xml -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/*</url-pattern>
   </servlet-mapping>
   ```

4. 指定一些后缀或者前缀等等...

   ```xml
   <!-- 注册Servlet -->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.study.servlet.HelloServlet</servlet-class>
   </servlet>
   
   <!-- 可以自定义后缀实现请求
   	注意：*前面不能加项目映射的路径
   -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>*.study</url-pattern>
   </servlet-mapping>
   ```

5. 指定了固有的映射路径优先级最高，如果找不到就会走默认的处理请求

   ```xml
   <!-- 注册Servlet -->
   <servlet>
       <servlet-name>hello</servlet-name>
       <servlet-class>com.study.servlet.HelloServlet</servlet-class>
   </servlet>
   
   <!-- Servlet的请求路径 -->
   <servlet-mapping>
       <servlet-name>hello</servlet-name>
       <url-pattern>/hello</url-pattern>
   </servlet-mapping>
   
   <servlet>
       <servlet-name>error</servlet-name>
       <servlet-class>com.study.servlet.ErrorServlet</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>error</servlet-name>
       <url-pattern>/*</url-pattern>
   </servlet-mapping>
   ```




## 6.5、ServletContext

web容器在启动的时候，它会为每个web程序都创建一个对应的ServletContext对象，它代表了当前的web应用。

### 1、共享数据

在这个Servlet中保存的数据，可以在另一个Servlet中拿到

![1635858050159](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635858050159.png)

```java
/**
	放置数据的类
 */
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        // this.getInitParameter();  初始化参数
        // this.getServletConfig();  Servlet配置
        // this.getServletContext(); Servlet上下文
        ServletContext context = this.getServletContext();

        String username = "用户"; // 数据
        context.setAttribute("username", username); // 将一个数据保存在了ServletContext中，名字为username，值为username


    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

```java
/**
	读取数据的类
 */
public class GetServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String username = (String) context.getAttribute("username");

        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        resp.getWriter().print("名字" + username);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee     http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.study.servlet.HelloServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

    <servlet>
        <servlet-name>getc</servlet-name>
        <servlet-class>com.study.servlet.GetServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>getc</servlet-name>
        <url-pattern>/getc</url-pattern>
    </servlet-mapping>

</web-app>
```

测试访问结果



### 2、获取初始化参数

```xml
<!-- 在web.xml中配置 -->
<!-- 配置一些web应用的初始化参数 -->
<context-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
</context-param>
```

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    ServletContext context = this.getServletContext();
    String url = context.getInitParameter("url");
    resp.getWriter().print(url);
}
```



### 3、请求转发

![1635859981960](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635859981960.png)

转发后路径不会发生改变

![1635860178500](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635860178500.png)

A只请求了B，从始至终都没有直接和C产生关系



重定向中发生了实质性的变化

![1635860310787](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635860310787.png)



```java
// 转发
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    ServletContext context = this.getServletContext();
    RequestDispatcher requestDispatcher = context.getRequestDispatcher("/gp"); // 设置转发的路径
    requestDispatcher.forward(req, resp);// 调用forward实现请求转发
}
```



### 4、读取资源文件

![1635861179126](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/1635861179126.png)

将文件放至resources目录下，打包后会和com代码包在classes目录下生成。

这个classes目录被称为classpath，也就是类路径

如果把配置文件放在了其他目录，比如com.study.servlet目录下，打包时就不会被整合进包，需要在pom文件中配置静态资源导出。这样就可以把配置文件打入com.study.servlet中了



```java
// 使用Properties对象读取配置文件
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    InputStream is = this.getServletContext().getResourceAsStream("/WEB-INF/classes/db.properties");
    Properties prop = new Properties();
    prop.load(is);
    String username = prop.getProperty("username");
    String password = prop.getProperty("password");

    resp.getWriter().print(username + ": " + password);
}
```

关于代码中的`/WEB-INF/classes/db.properties`：

1. 第一个`/`代表打包后的`servlet-02-1.0-SNAPSHOT`，代表当前web程序根目录
2. ``/WEB-INF/classes`是打包后的class文件和配置文件的文件夹

