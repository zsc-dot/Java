# HTML与CSS

HTML：即 HyperText Markup language 超文本标记语言，HTML 的作用是定义网页的内容和结构。

* HyperText 是指用超链接的方式组织网页，把网页联系起来
* Markup 是指用 `<标签>` 的方式赋予内容不同的功能和含义

CSS：即 Cascading  Style  Sheets 级联（层叠）样式表，它描述了网页的表现与展示效果



# 一、HTML元素

HTML 由一系列元素 `elements` 组成，例如

```html
<p>Hello, world!</p>
```

* 整体称之为元素
* `<p>` 和 `</p>` 分别称为起始和结束标签
* 标签包围起来的 Hello, world 称之为内容

* p 是预先定义好的 html 标签，作用是将内容作为一个单独的段落



元素还可以有属性，如

```html
<p id="p1">Hello, world!</p>
```

- 属性一般是预先定义好的，这里的 id 属性是给元素一个唯一的标识



元素之间可以嵌套，如

```html
<p>HTML 是一门非常<b>强大</b>的语言</p>
```



不包含内容的元素称之为空元素，如

```html
<img src="1.png">
<img src="1.png"/>
```

* img 作用是用来展示图片
* src 属性用来指明图片路径



# 二、HTML页面

前面介绍的只是单独的 HTML 元素，它们可以充当一份完整的 HTML 页面的组成部分

```html
<!DOCTYPE html> --文档类型声明
<html>
  <head>
    <meta charset="utf-8">
    <title>测试页面</title>
  </head>
  <body>
    <p id="p1">Hello, world!</p>
    <img src="1.png">
  </body>
</html>
```

* `html` 元素囊括了页面中所有其它元素，整个页面只需一个，称为根元素
* `head` 元素包含的是那些不用于展现内容的元素，如 `title`，`link`，`meta` 等
* `body` 元素包含了对用户展现内容的元素，例如后面会学到的用于展示文本、图片、视频、音频的各种元素



# 三、常见元素

## 1、文本

**Heading**

```html
<h1>1号标题</h1>
<h2>2号标题</h2>
<h3>3号标题</h3>
<h4>4号标题</h4>
<h5>5号标题</h5>
<h6>6号标题</h6>
```



**Paragraph**

```html
<p>段落</p>
```



**List**

无序列表 unordered list

```html
<ul>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ul>
```

有序列表

```html
<ol>
    <li>列表项1</li>
    <li>列表项2</li>
    <li>列表项3</li>
</ol>
```

多级列表

```html
<ul>
    <li>
    	北京市
        <ul>
            <li>海淀区</li>
            <li>朝阳区</li>
            <li>昌平区</li>
        </ul>
    </li>
    <li>
    	河北省
        <ul>
            <li>石家庄</li>
            <li>保定</li>
        </ul>
    </li>
</ul>
```



**Anchor**

锚，超链接

```html
<a href="网页地址">超链接文本</a>
```

#+标签id可以实现页面内跳转



## 2、多媒体

**Image**

```html
<img src="文件路径">
```

src 格式有 3 种

* 文件地址

* data URL，格式如下

  ```
  data:媒体类型;base64,数据
  ```

* object URL，需要配合 javascript 使用



**Video**

```html
<video src="文件路径"></video>
```



**Audio**

```html
<audio src="文件路径"></audio>
```



## 3、表单

**作用与语法**

表单的作用：**收集**用户填入的**数据**，并将这些数据**提交给服务器**

表单的语法

```html
<form action="服务器地址" method="请求方式" enctype="数据格式">
    <!-- 表单项 -->
    
    <input type="submit" value="提交按钮">
</form>
```

* method 请求方式有 
  * get （默认）提交时，数据跟在 URL 地址之后
  * post 提交时，数据在请求体内
* enctype 在 post 请求时，指定请求体的数据格式
  * application/x-www-form-urlencoded（默认）
  * multipart/form-data
* 其中表单项提供多种收集数据的方式
  * 有 name 属性的表单项数据，才会被发送给服务器



**常见的表单项**

文本框

```html
<input type="text" name="uesrname">
```

密码框

```html
<input type="password" name="password">
```

隐藏框

```html
<input type="hidden" name="id">
```

日期框

```html
<input type="date" name="birthday">
```

单选

```html
<input type="radio" name="sex" value="男" checked>
<input type="radio" name="sex" value="女">
```

多选

```html
<input type="checkbox" name="fav" value="唱歌">
<input type="checkbox" name="fav" value="逛街">
<input type="checkbox" name="fav" value="游戏">
```

文件上传

```html
<input type="file" name="avatar">
```



# 四、HTTP请求

### 1、请求组成

请求由三部分组成

1. 请求行
2. 请求头
3. 请求体

可以用 telnet 程序测试



### 2、请求方式与数据格式

#### 2.1、get 请求示例

```
GET /test2?name=%E5%BC%A0&age=20 HTTP/1.1
Host: localhost
```

- %E5%BC%A0 是【张】经过 URL 编码后的结果

#### 2.2、post 请求示例

```
POST /test2 HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 21

name=%E5%BC%A0&age=18
```

application/x-www-form-urlencoed 格式细节：

* 参数分成名字和值，中间用 = 分隔
* 多个参数使用 & 进行分隔
* 【张】等特殊字符需要用 encodeURIComponent() 编码为 【%E5%BC%A0】后才能发送

#### 2.3、json请求示例

```
POST /test3 HTTP/1.1
Host: localhost
Content-Type: application/json
Content-Length: 25

{"name":"zhang","age":18}
```

其中属性值可以是

* 字符串 ""
* 数字
* true, false
* null
* 对象
* 数组

#### 2.4、multipart 请求示例

```
POST /test2 HTTP/1.1
Host: localhost
Content-Type: multipart/form-data; boundary=123
Content-Length: 125

--123
Content-Disposition: form-data; name="name"

lisi
--123
Content-Disposition: form-data; name="age"

30
--123--
```

* boundary=123 用来定义分隔符
* 起始分隔符是 `--分隔符`
* 结束分隔符是 `--分隔符--`
* 长度为125是因为字符串长度为116，但包含了9个换行符



#### 2.5、数据格式小结

客户端发送

* 编码 
  * application/x-www-form-urlencoded ：url 编码
  * application/json：utf-8 编码
  * multipart/form-data：每部分编码可以不同
* 表单只支持以 application/x-www-form-urlencoded 和 multipart/form-data 格式发送数据
* 文件上传需要用 multipart/form-data 格式
* js 代码可以支持任意格式发送数据 

服务端接收

* 对 application/x-www-form-urlencoded 和 multipart/form-data 格式的数据，Spring 接收方式是统一的，只需要用 java bean 的属性名对应请求参数名即可
* 对于 applicaiton/json 格式的数据，Spring 接收需要使用 @RequestBody 注解 + java bean 的方式



### 3、session 原理

Http 无状态，有会话

* 无状态是指，请求之间相互独立，第一次请求的数据，第二次请求不能重用。比如第一次请求发送了一个参数“name=zhang”，第二次请求还想发这个参数，不能从第一次请求里找出来重用。
* 有会话是指，客户端和服务端都有相应的技术，可以暂存数据，让数据在请求间共享

服务端使用了 session 技术来暂存数据

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415100203414.png" alt="image-20240415100203414" style="zoom:80%;" />

存

```
GET /s1?name=zhang HTTP/1.1
Host: localhost
```

取

```
GET /s2 HTTP/1.1
Host: localhost
Cookie: JSESSIONID=560FA845D02AE09B176E1BC5D9816A5D
```



session 技术实现身份验证

```mermaid
sequenceDiagram
participant Client
participant L as LoginController
participant i as LoginInterceptor
participant Session
rect rgb(200, 223, 255)
Client ->> +L : 登录请求
L ->> L : 检查用户名，密码，验证通过
L ->> +Session : 存入用户名
Session -->> -L: 
L -->> -Client: 登录成功
end
rect rgb(200, 190, 255)
Client ->> +i : 其它请求
i ->> +Session : 获取用户名
Session -->> -i : 
i ->> i: 用户名存在，放行
i -->> -Client : 
end
```

### 4、JWT原理

jwt技术实现身份验证

```mermaid
sequenceDiagram
participant Client
participant L as LoginController
participant i as LoginInterceptor

rect rgb(200, 223, 255)
Client ->> +L : 登录请求
L ->> L : 检查用户名，密码，验证通过
L -->> -Client : 登录成功，返回token
end

rect rgb(150, 190, 155)
Client ->> +i : 其它请求，携带token
i ->> i : 校验token，校验无误，放行
i -->> -Client : 
end
```

生成 token

```
GET /j1?name=zhang&pass=123 HTTP/1.1
Host: localhost
```

校验token

```
GET /j2 HTTP/1.1
Host: localhost
Authorization: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiJ9._1-P_TLlzQPb1_lCyGwplMZaKQ8Mcw_plBbYPZ3OX28
```

token被`.`分隔，由三部分组成：

- header：签名算法
- payload：数据
- 签名

header和payload没有被加密，只是json字符串使用了base4编码，千万不能放敏感数据

签名可以保证token的数据不被篡改，签名是根据前面两部分和密钥生成的



# 五、CSS

即 Cascading  Style  Sheets，它描述了网页的表现与展示效果



## 1、选择器

- type 选择器：根据标签名进行匹配（元素选择器）

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>CSS示例</title>
      <link rel="stylesheet" href="style.css">
  </head>
  <body>
      <p>111111111111111111111</p>
      <p>222222222222222222222</p>
      <p>333333333333333333333</p>
  </body>
  </html>
  ```

  ```css
  p {
      background-color: forestgreen;
  }
  ```

  效果：

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415103253769.png" alt="image-20240415103253769" style="zoom:80%;" />

- class 选择器：根据元素的 class 属性进行匹配

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>CSS示例</title>
      <link rel="stylesheet" href="style.css">
  </head>
  <body>
      <p>111111111111111111111</p>
      <p class="c1">222222222222222222222</p>
      <p class="c1">333333333333333333333</p>
  </body>
  </html>
  ```

  ```css
  .c1 {
      background-color: cyan;
  }
  ```

  效果：

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415103658983.png" alt="image-20240415103658983" style="zoom:80%;" />

- id 选择器：根据元素的 id 属性进行匹配

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>CSS示例</title>
      <link rel="stylesheet" href="style.css">
  </head>
  <body>
      <p id="p1">111111111111111111111</p>
      <p class="c1" id="p2">222222222222222222222</p>
      <p class="c1" id="p3">333333333333333333333</p>
  </body>
  </html>
  ```

  ```css
  #p3 {
      background-color: cyan;
  }
  ```



优先级：如果一个元素同时使用了三种选择器，优先级为id选择器 > class选择器 > 元素(type)选择器



## 2、属性和值

- background-color : red;

- ...

- display

  none：隐藏元素，block：显示元素

  ```css
  #p3 {
      background-color: cyan;
      display: none;
  }
  ```

  p3元素在页面上就会被隐藏



## 3、布局

与布局相关的 html 元素

- div：容器标签，可以在它内部写其他标签

  ```html
  <!DOCTYPE html>
  <html lang="zh">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>布局</title>
      <style>
          html,body {
              margin:0;
              width: 100%;
              height: 100%;
              text-align: center;
              font-size: 30px;
              font-weight: bold;            
          }
          div{
              box-sizing: border-box;
          }
          .container {
              height: 100%;            
              position: relative;
          }
          #header {
              background-color:rgb(152, 152, 255);
              width: 100%;
              height: 80px;
              padding-top: 10px;
          }
          #aside {
              background-color:aquamarine;
              float: left;
              width: 200px;
              height: calc(100% - 140px);
              padding-top: 10px;
          }
          #main {
              background-color:honeydew;
              float: left;
              width: calc(100% - 200px);
              height: calc(100% - 140px);
              padding-top: 10px;
              padding-left: 20px;
              text-align: left;
          }
          #footer {
              background-color:darksalmon;
              height: 60px;
              padding-top: 10px;
          }
      </style>
  </head>
  <body>
      <div class="container">
          <div id="header">#header</div>
          <div id="aside">#aside</div>
          <div id="main">#main</div>
          <div style="clear: both;"></div>
          <div id="footer">#footer</div>
      </div>
  </body>
  </html>
  ```

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415104921953.png" alt="image-20240415104921953" style="zoom: 67%;" />

- template

  定义模板，通过JavaScript可以实现每点击一次按钮就生成一个模板

  ```html
  <!DOCTYPE html>
  <html lang="zh">
  
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>模板</title>
      <style>
          html,
          body {
              margin: 0;
              width: 100%;
              height: 100%;
          }
  
          .btn {
              padding: 10px;
          }
  
          .out {
              width: 100%;
              height: 100%;
              box-sizing: border-box;
              background-color:darkgrey;
          }
  
          .in {
              width: 200px;
              box-sizing: border-box;
              height: 200px;
              border: solid 2px black;
              padding: 10px;
              background-color: antiquewhite;
              margin: 10px;
              float: left;
          }
      </style>
  </head>
  
  <body>    
      <div class="out">
          <div class="btn">
              <input type="button" value="根据模板创建" id="add">
          </div>
          
      </div>
      <template id="t">
          <div class="in">
              <form action="">
                  <p><label>姓名</label> <input type="text"></p>
                  <p><label>年龄</label> <input type="text"></p>
                  <p><input type="submit" value="添加"></p>
              </form>
          </div>
      </template>
      <script>
          document.getElementById("add").onclick = () => {
              let t = document.getElementById("t");
              let inputs = t.content.querySelectorAll("input");
              inputs[0].value = randomGenerator("abcdefghijklmnopqrstuvwxyz", 5);
              inputs[1].value = randomGenerator("1234567890", 2);
              const c = document.importNode(t.content, true);
              document.querySelector(".out").appendChild(c);
          }
          function randomGenerator(str, n) {
              const result = [];
              for (let i = 0; i < n; i++) {
                  result.push(str.charAt(Math.floor(Math.random() * str.length)))
              }
              return result.join("");
          }
      </script>
  </body>
  
  </html>
  ```

  效果图：

  ![image-20240415105151335](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415105151335.png)

