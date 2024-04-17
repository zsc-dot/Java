# 一、Vue基础

## 1、环境准备

### 1.1、安装脚手架

```cmd
npm install -g @vue/cli
```

- -g 参数表示全局安装，这样在任意目录都可以使用 vue 脚本创建项目



### 1.2、创建项目

```cmd
vue ui
```

使用图形向导来创建 vue 项目，如下图，输入项目名

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417223948127.png" alt="image-20240417223948127" style="zoom:80%;" />

选择手动配置项目

![image-20240417224124321](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417224124321.png)

添加vue router 和 vuex

![image-20240417224211452](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417224211452.png)

选择版本，创建项目

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417224254934.png" alt="image-20240417224254934" style="zoom:80%;" />



### 1.3、安装 devtools

- devtools 插件网址：https://devtools.vuejs.org/guide/installation.html

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417224537126.png" alt="image-20240417224537126" style="zoom:80%;" />



### 1.4、运行项目

进入项目目录，执行

```cmd
npm run serve
```



### 1.5、修改端口

前端服务器默认占用了8080端口，需要修改一下

- 文档地址：[DevServer | webpack](https://webpack.js.org/configuration/dev-server/#devserverport)

- 打开 vue.config.js 添加

  ```js
  const { defineConfig } = require('@vue/cli-service')
  module.exports = defineConfig({
    // ...
    devServer: {
      port: 7070
    }
  })
  ```



### 1.6、添加代理

为了避免前后端服务器联调时，fetch、xhr请求产生跨域问题，需要配置代理

- 文档地址同上

- 打开vue.config.js添加

  ```js
  const { defineConfig } = require('@vue/cli-service')
  module.exports = defineConfig({
    // ...
    devServer: {
      port: 7070,
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true
        }
      }
    }
  })
  ```



### 1.7、Vue项目结构

```txt
PS D:\study-code\Front\Vue2\client> tree src
D:\study-code\Front\Vue2\CLIENT\SRC
├─assets
├─components
├─router
├─store
└─views
```

- assets：静态资源
- compoents：可重用组件
- router：路由
- store：数据共享
- views：视图组件

以后还会添加

- api：跟后台交互，发送fetch、xhr请求，接收响应
- plugins：插件



## 2、Vue组件