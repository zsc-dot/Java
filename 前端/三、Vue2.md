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

### 2.1、练习

Vue 的组件文件以 .vue 结尾，每个组件由三部分组成

```vue
<template></template>
<script></script>
<style></style>
```

- template 模板部分，由它生成 html 代码
- script 代码部分，控制模板的数据来源和行为
- style 样式部分，一般不用考虑



入口组件是 App.vue

先删除原有代码，来个 Hello, World 例子

```vue
<template>
  <h1>Hello, World</h1>
</template>
<script></script>
```

使用 JavaScript 动态加载数据

```vue
<template>
  <h1>{{msg}}</h1>
</template>
<script>
const options = {
  data: function (){
    return {
      msg: '你好！'
    };
  }
};
export default options;
</script>
```

解释：

- export default 导出 options 组件对象，供 main.js 导入使用
- 这个对象有一个 data 方法，返回一个**对象**，给 template 提供数据
- `{{}}` 在 Vue 里称之为插值表达式，用来**绑定** data 方法返回的**对象**属性，**绑定**的含义是数据发生变化时，页面显示会同步变化



### 2.2、Vue 解析组件的原理

- main.js 负责解析组件

  ```js
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  import store from './store'
  
  Vue.config.productionTip = false
  
  new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount('#app')
  ```

- 导入 APP.vue 组件

  ```js
  import App from './App.vue'
  ```

  导入语句相当于把 APP.vue 的`<template></template>` 标签导入

  ```js
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  import store from './store'
  
  Vue.config.productionTip = false
  
  /* 
  <template>
    <h1>{{msg}}</h1>
  </template>
  */
  
  new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount('#app')
  ```

- 模板中需要解析`{{msg}}`，main.js 中的 `h` 函数负责解析模板，将 `{{msg}}` 替换为实际值，并生成一个虚拟节点

  - 虚拟节点：也是一种 html 的元素，只是没有和最终的页面结合到一起

- `$mount` 函数负责把解析好的 html 虚拟节点放到页面展示

  - `#app` 就是 id 选择器，在原始页面 index.html 文件中，有个 `<div></div>` 标签，它的 id 就是 app

    ```html
    <!DOCTYPE html>
    <html lang="">
      <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width,initial-scale=1.0">
        <link rel="icon" href="<%= BASE_URL %>favicon.ico">
        <title><%= htmlWebpackPlugin.options.title %></title>
      </head>
      <body>
        <noscript>
          <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
        </noscript>
        <div id="app"></div>
        <!-- built files will be auto injected -->
      </body>
    </html>
    ```

  - `$mount` 函数最终会把虚拟节点放到 `<div id="app"></div>`位置上



### 2.3、自定义新组件

- 在 `views` 目录下新建一个 Vue 组件，命名为 Example1View.vue

  ```vue
  <template>
      <h1>我是新组件</h1>
  </template>
  ```

- 在 main.js 中解析新建的组件

  ```js
  import Vue from 'vue'
  // import App from './App.vue'
  import e1 from './views/Example1View.vue'
  import router from './router'
  import store from './store'
  
  Vue.config.productionTip = false
  
  new Vue({
    router,
    store,
    render: h => h(e1)
  }).$mount('#app')
  ```

- 效果

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240420172028318.png" alt="image-20240420172028318" style="zoom: 80%;" />



### 2.4、文本插值

```vue
<template>
    <div>
        <h1>{{name}}</h1>
        <h1>{{age > 60 ? '老年' : '青年'}}</h1>
    </div>
</template>
<script>
const options = {
    data() {
        return {
            name: '张三',
            age: 18
        };
    }
};
export default options;
</script>
```

- `{{}}` 里只能绑定一个属性，绑定多个属性需要用多个 `{{}}` 分别绑定

  - `<h1>{{name}}  {{age}}</h1>`

- template 标签中只能有一个根元素，默认为最顶层的元素节点

  ```vue
  <template>
      <h1>{{name}}</h1>
      <h1>{{age > 60 ? '老年' : '青年'}}</h1>
  </template>
  ```

  这么写会报错，可以用 `<div></div>` 标签包起来，这样就只有一个根元素了

- 插值内可以进行简单的表达式计算

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240420172854837.png" alt="image-20240420172854837" style="zoom:80%;" />



### 2.5、属性绑定

```vue
<template>
    <div>
        <div><input type="text" v-bind:value="name"></div>
        <div><input type="date" v-bind:value="birthday"></div>
        <div><input type="text" :value="age"></div>
    </div>
</template>
<script>
const options = {
    data: function () {
        return {
            name: '王五',
            birthday: '1995-05-01',
            age: 20
        };
    }
};
export default options;
</script>
```

- `<input>` 标签中赋值时的格式为`<input type="text" value="李四">`，可以通过 `v-bind:`进行属性绑定
- `v-bind:` 可以简写为只保留冒号 `:`
- 不加`:`时，`=` 后面的内容就是要显示的内容，加上 `:` 表示实现了动态绑定



### 2.6、事件绑定

```vue
<template>
    <div>
        <div><input type="button" value="点我执行m1" v-on:click="m1"></div>
        <div><input type="button" value="点我执行m2" @click="m2"></div>
        <div>{{count}}</div>
    </div>
</template>
<script>
const options = {
    data: function() {
        return {
            count: 0
        };
    },
    methods: {
        m1() {
            this.count++;
            console.log("m1");
        },
        m2() {
            this.count--;
            console.log("m2");
        }
    }
};
export default options;
</script>
```

- 标签绑定鼠标单击事件可以使用 `v-on:click` 
- `v-on:click` 简写可以写为 `@click`
- methods 方法中调用 data 方法的属性必须使用 this
- 在 methods 方法中的 this 代表的是 data 函数返回的数据对象



### 2.7、双向绑定

上面的绑定是单向绑定， template 中的标签绑定 data 中的属性或事件后，可以显示它们对应的值。

但是，输入框中输入的数据并不会绑定到 data 中

```vue
<template>
    <div>
        <div>
            <label for="">请输入姓名</label>
            <input type="text" v-model="name">
        </div>
        <div>
            <label for="">请输入年龄</label>
            <input type="text" v-model="age">
        </div>
        <div>
            <label for="">请选择性别</label>
            男 <input type="radio" value="男" v-model="sex">
            女 <input type="radio" value="女" v-model="sex">
        </div>
        <div>
            <label for="">请选择爱好</label>
            游泳 <input type="checkbox" value="游泳" v-model="fav">
            打球 <input type="checkbox" value="打球" v-model="fav">
            健身 <input type="checkbox" value="健身" v-model="fav">
        </div>
    </div>
</template>
<script>
const options = {
    data() {
        return{
            name: 'aaa',
            age: null,
            sex: '男',
            fav: ['打球']
        };
    }
};
export default options;
</script>
<style scoped>
div,label {
    font-size: 14px;
    margin-right: 6px;
}
div {
    margin-bottom: 6px;
}
</style>
```

- 用 `v-model` 实现双向绑定，即
  - JavaScript 数据可以同步到表单标签
  - 反过来用户在表单标签输入的新值也会同步到 JavaScript 这边
- 双向绑定只适用于表单这种带【输入】功能的标签，其他标签的数据绑定，单项就足够了
- 复选框这种标签，双向绑定的 JavaScript 数据类型一般用数组



### 2.7、计算属性

```vue
<!-- 计算属性 -->
<template>
    <div>
        <h2>{{lastName}}{{firstName}}</h2>
        <h2>{{lastName + firstName}}</h2>
        <h2>{{fullName()}}</h2>
        <h2>{{fullName1}}</h2>
    </div>
</template>
<script>
const options = {
    data() {
        return{
            firstName: '三',
            lastName: '张'
        };
    },
    methods: {
        fullName() {
            return this.lastName + this.firstName;
        }
    },
    computed: {
        fullName1() {
            console.log("进入了 fullNam1");
            return this.lastName + this.firstName;
        }
    }
};
export default options;
</script>
```

- 使用 `computed` 实现计算属性
- 普通方法调用时必须加 ()，并且没有缓存功能
- 计算属性使用时就把它当属性来用，不加 ()，有缓存功能
  - 一次计算后，会将结果缓存，下次再计算时，只要数据没有变化，就不会重新计算，直接返回缓存结果



### 2.8、axios

#### 2.8.1、入门

axios 它的底层是用了 XMLHttpRequest（xhr）方式发送请求和接收响应，xhr 相对于之前讲过的 fetch api 来说，功能更强大，但由于是比较老的 api，不支持 Promise，axios 对 xhr 进行了封装，使之支持 Promise，并提供了对请求、响应的统一拦截功能

安装

```cmd
npm install axios -S
```

导入

```js
import axios from 'axios'
```

- axios 默认导出一个对象，这里的 import 导入的就是它默认导出的对象

练习：

```vue
<template>
    <div>
        <div><input type="button" value="获取远程数据" @click="sendReq"></div>
    </div>
</template>
<script>
import axios from 'axios'
const options = {
    methods: {
        async sendReq() {
            const resp = await axios.get('/api/students');
            console.log(resp);
        }
    }
};
export default options;
</script>
```

- axios 调用接口时，可以不写 `http://localhost:8080` ，因为已经在 `vue.config.js` 文件中配置了代理，详情见 `1.6`



#### 2.8.2、axios请求

方法

| 请求                               | 备注   |
| ---------------------------------- | ------ |
| axios.get(url[, config])           | :star: |
| axios.delete(url[, config])        |        |
| axios.head(url[, config])          |        |
| axios.options(url[, config])       |        |
| axios.post(url[, data[, config]])  | :star: |
| axios.put(url[, data[, config]])   |        |
| axios.patch(url[, data[, config]]) |        |

- config：选项对象，例如查询参数、请求头...
- data：请求体数据，最常见的是 json 格式数据
- get、head 请求无法携带请求体，这应当是浏览器的限制所致（xhr、fetch api均有限制）

练习：

```vue
<!-- axios -->
<template>
    <div>
        <div><input type="button" value="获取远程数据" @click="sendReq"></div>
    </div>
</template>
<script>
import axios from 'axios'
const options = {
    methods: {
        async sendReq() {
            // 1. get、post演示
            // const resp = await axios.post('/api/a2');

            // 2. 发送请求头
            // const resp = await axios.post('/api/a3', {}, {
            //     headers: {
            //         Authorization: 'abc'
            //     }
            // });

            // 3. 发送请求时携带查询参数 ?name=xxx&age=xxx
            // const name = encodeURIComponent('&&&');
            // const age = 18;
            // const resp = await axios.post(`/api/a4?name=${name}&age=${age}`);

            // 不想自己拼串、处理特殊字符，就用下面的方法
            // const resp = await axios.post('/api/a4', {}, {
            //     params: {
            //         name: '&&&&',
            //         age: 20
            //     }
            // });

            // 4. 用请求体发送数据，格式为 urlencoded
            // 请求体传参时，格式如果不是 json，就要用 URLSearchParams
            // const params = new URLSearchParams();
            // params.append("name", "张三");
            // params.append("age", 24);
            // const resp = await axios.post('/api/a4', params);

            // 5.用请求体发送数据，格式为 multipart
            // const params = new FormData();
            // params.append("name", "李四");
            // params.append("age", 30);
            // const resp = await axios.post('/api/a5', params);

            // 6. 用请求体发送数据，格式为 json
            const resp = await axios.post('/api/a5json', {
                name: '王五',
                age: 50
            });
            console.log(resp);
        }
    }
};
export default options;
</script>
```



#### 2.8.3、默认配置

创建实例

```js
const _axios = axios.create(config);
```

- axios 对象可以直接使用，但使用的是默认设置

- 用 axios.create 创建的对象，可以覆盖默认设置，config 见下面说明

  | 名称            | 含义                                                       |
  | --------------- | ---------------------------------------------------------- |
  | baseURL         | 将自动加在 url 前面                                        |
  | headers         | 请求头，类型为简单对象                                     |
  | params          | 跟在 URL 后的请求参数，类型为简单对象或 URLSearchParams    |
  | data            | 请求体，类型有简单对象、FormData、URLSearchParams、File 等 |
  | withCredentials | 跨域时是否携带 Cookie 等凭证，默认为 false                 |
  | responseType    | 响应类型，默认为 json                                      |



baseURL

```js
const _axios = axios.create({
	baseURL: 'http://localhost:8080'
});
const resp = await _axios.post('/api/a5json', {
    name: '王五',
    age: 50
});
```

- 生产环境希望 xhr 请求不走代理，直接访问后端地址，可以用 baseURL 统一修改，此时跨域问题需要后端解决



withCredentials

```js
const _axios = axios.create({
    baseURL: 'http://localhost:8080',
    withCredentials: true
});
await _axios.post('/api/a6set', {});
await _axios.post('/api/a6get', {});
```

- 如果不为 true，两次请求的 sessionId 会不一致。虽然第一次请求后端会返回一个 JSESSIONID 的 cookie，但是第二次请求时浏览器并不会携带该 sessionId，设置为 true 即可
- Java 后端也需要修改，`@CrossOrigin` 中需要将 `allowCredentials` 设置为 true，否则浏览器获取跨域返回的 cookie 时会报错



#### 2.8.4、响应格式

| 名称    | 含义              |
| ------- | ----------------- |
| data    | 响应体数据 :star: |
| status  | 状态码 :star:     |
| headers | 响应头            |

- 200 表示响应成功
- 400 请求数据不正确，如 age=abc
- 401 身份验证未通过
- 403 没有权限
- 404 资源不存在
- 405 不支持请求方式 post
- 500 服务器内部错误

Java 后端统一处理跨域

```java
@SpringBootApplication
public class App implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:7070")
                .allowCredentials(true);
    }

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```



#### 2.8.5、拦截器

请求拦截器

```js
_axios.interceptors.request.use(
    function(config) {
        // 比如在这里添加统一的 headers
        config.headers = {
            Authorization: 'aaaaa.bbbb.cccc'
        }
        return config;
    },
    function(error) {
        return Promise.reject(error);
    }
);
```

响应拦截器

```js
_axios.interceptors.response.use(
    function(response) {
        // 2xx 范围走这里
        return response;
    },
    function(error) {
        // 超出 2xx，比如 4xx、5xx 走这里
        if (error.response.status === 400) {
            console.log('请求参数不正确');
            return Promise.resolve(400);
        }else if (error.response.status === 401) {
            console.log('跳转至登陆页面');
            return Promise.resolve(401);
        }else if (error.response.status === 404) {
            console.log('资源未找到');
            return Promise.resolve(404);
        }
        return Promise.reject(error);
    }
);
```



#### 2.8.6、自定义axios组件

在 `src` 目录下，新建 `util` 文件夹，将 axios 的配置放入

```js
import axios from 'axios'

// 7. 默认配置
const _axios = axios.create({
    baseURL: 'http://localhost:8080',
    withCredentials: true
});

// 9. 拦截器
_axios.interceptors.request.use(
    function(config) {
        // 比如在这里添加统一的 headers
        config.headers = {
            Authorization: 'aaaaa.bbbb.cccc'
        }
        return config;
    },
    function(error) {
        return Promise.reject(error);
    }
);
_axios.interceptors.response.use(
    function(response) {
        // 2xx 范围走这里
        return response;
    },
    function(error) {
        // 超出 2xx，比如 4xx、5xx 走这里
        if (error.response.status === 400) {
            console.log('请求参数不正确');
            return Promise.resolve(400);
        }else if (error.response.status === 401) {
            console.log('跳转至登陆页面');
            return Promise.resolve(401);
        }else if (error.response.status === 404) {
            console.log('资源未找到');
            return Promise.resolve(404);
        }
        return Promise.reject(error);
    }
);

export default _axios;
```

demo 中将导入的 axios 换为我们自定义的 axios，即可正常使用

```js
import _axios from '../util/myaxios'
```



### 2.9、条件渲染

`v-if` 进行标签判断

```vue
<template>
    <div>
        <input type="button" value="获取远程数据" @click="sendReq">
        <div class="title">学生列表</div>
        <div class="thead">
            <div class="row bold">
                <div class="col">编号</div>
                <div class="col">姓名</div>
                <div class="col">性别</div>
                <div class="col">年龄</div>
            </div>
        </div>
        <div class="tbody">
            <div class="row" v-if="students.length > 0">显示学生数据</div>
            <div class="row" v-else>暂无学生数据</div>
        </div>
    </div>
</template>
<script>
import axios from '../util/myaxios';
const options = {
    data() {
        return {
            students: []
        };
    },
    methods: {
        async sendReq() {
            const resp = await axios.get("/api/students");
            console.log(resp.data.data);
            this.students = resp.data.data;
        }
    }
};
export default options;
</script>
<style scoped>
    div {
        font-family: 华文行楷;
        font-size: 20px;
    }

    .title {
        margin-bottom: 10px;
        font-size: 30px;
        color: #333;
        text-align: center;
    }

    .row {
        background-color: #fff;
        display: flex;
        justify-content: center;
    }

    .col {
        border: 1px solid #f0f0f0;
        width: 15%;
        height: 35px;
        text-align: center;
        line-height: 35px;
    }

    .bold .col {
        background-color: #f1f1f1;
    }
</style>
```

- style 标签中的 `scoped` 表示样式只在当前页面有效



### 2.10、列表渲染

`v-for` 进行标签循环渲染

```vue
<template>
    <div>
        <input type="button" value="获取远程数据" @click="sendReq">
        <div class="title">学生列表</div>
        <div class="thead">
            <div class="row bold">
                <div class="col">编号</div>
                <div class="col">姓名</div>
                <div class="col">性别</div>
                <div class="col">年龄</div>
            </div>
        </div>
        <div class="tbody">
            <div v-if="students.length > 0">
                <div class="row" v-for="s of students" :key="s.id">
                    <div class="col">{{s.id}}</div>
                    <div class="col">{{s.name}}</div>
                    <div class="col">{{s.sex}}</div>
                    <div class="col">{{s.age}}</div>
                </div>
            </div>
            <div class="row" v-else>暂无学生数据</div>
        </div>
    </div>
</template>
<script>
import axios from '../util/myaxios';
const options = {
    data() {
        return {
            students: []
        };
    },
    methods: {
        async sendReq() {
            const resp = await axios.get("/api/students");
            console.log(resp.data.data);
            this.students = resp.data.data;
        }
    },
    mounted() {
        this.sendReq();
    }
};
export default options;
</script>
<style scoped>
    div {
        font-family: 华文行楷;
        font-size: 20px;
    }

    .title {
        margin-bottom: 10px;
        font-size: 30px;
        color: #333;
        text-align: center;
    }

    .row {
        background-color: #fff;
        display: flex;
        justify-content: center;
    }

    .col {
        border: 1px solid #f0f0f0;
        width: 15%;
        height: 35px;
        text-align: center;
        line-height: 35px;
    }

    .bold .col {
        background-color: #f1f1f1;
    }
</style>
```

注意：

- v-if 和 v-for 不能用于同一个标签
- v-for 需要配合特殊的标签属性 key 一起使用，并且 key 属性要绑定到一个能起到唯一标识作用的数据上，比如绑定学生编号
- options 的 mounted 属性对应一个函数，此函数会在组件挂载后（准备就绪）被调用，可以在它内部发起请求，去获取学生数据



### 2.11、重用组件

按钮组件

```vue
<template>
    <div class="button" :class="[type,size]">
        a<slot></slot>b
    </div>
</template>
<script>
const options = {
    props: [
        "type",
        "size"
    ]
};
export default options;
</script>
<style scoped>
.button {
    display: inline-block;
    text-align: center;
    border-radius: 30px;
    margin: 5px;
    font: bold 12px/25px Arial, sans-serif;
    padding: 0 2px;
    text-shadow: 1px 1px 1px rgba(255, 255, 255, .22);
    box-shadow: 1px 1px 1px rgba(0, 0, 0, .29), inset 1px 1px 1px rgba(255, 255, 255, .44);
    transition: all 0.15s ease;
}

.button:hover {
    box-shadow: 1px 1px 1px rgba(0, 0, 0, .29), inset 1px 1px 2px rgba(0, 0, 0, .5);
}

.button:active {
    box-shadow: inset 1px 1px 2px rgba(0, 0, 0, .8);
}

.primary {
    background-color: #1d6ef9;
    color: #b5e3f1;
}

.danger {
    background-color: rgb(196, 50, 50);
    color: white;
}

.success {
    background-color: #a5cd4e;
    ;
    color: #3e5706;
}

.small {
    width: 40px;
    height: 20px;
    font-size: 10px;
    line-height: 20px;
}

.middle {
    width: 50px;
    height: 25px;
    font-size: 14px;
    line-height: 25px;
}

.large {
    width: 60px;
    height: 30px;
    font-size: 18px;
    line-height: 30px;
}
</style>
```

- 定义 props 数组，在 div 标签中绑定 class 属性
- `<slot>`标签表示占位，这样才能显示下面的1、2、3，a和b则会显示在两边，按钮的文字就是`a1b`...



使用组件

```vue
<template>
    <div>
        <h1>父组件</h1>
        <my-button type="primary" size="small">1</my-button>
        <my-button type="danger" size="middle">2</my-button>
        <my-button type="success" size="large">3</my-button>
    </div>
</template>
<script>
import MyButton from '../components/MyButton'
const options = {
    components: {MyButton}
};
export default options;
</script>
<style scoped>
    div {
        background-color: gainsboro;
        width: 100%;
        height: 400px;
    }
    h1 {
        font-family: 华文行楷;
    }
</style>
```



# 二、Vue进阶

## 1、ElementUI

### 1.1、入门练习

安装

```cmd
npm install element-ui -S
```

在 main.js 中引入组件

```js
import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.use(Element)
```

例子

```js
import Vue from 'vue'
import e8 from './views/Example8View.vue'
import router from './router'
import store from './store'
import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.config.productionTip = false

Vue.use(Element)

new Vue({
  router,
  store,
  render: h => h(e8)
}).$mount('#app')
```



测试，在自己的组件中使用 ElementUI 的组件

```html
<template>
    <div>
        <el-button>我是按钮</el-button>
    </div>
</template>
<script>
const options = {

};
export default options;
</script>
```



### 1.2、表格组件

```vue
<!-- ElementUI -->
<template>
    <div>
        <el-table :data="students">
            <el-table-column label="编号" prop="id"></el-table-column>
            <el-table-column label="姓名" prop="name"></el-table-column>
            <el-table-column label="性别" prop="sex"></el-table-column>
            <el-table-column label="年龄" prop="age"></el-table-column>
        </el-table>
    </div>
</template>
<script>
import axios from "../util/myaxios";
const options = {
    data() {
        return {
            students: []
        };
    },
    async mounted() {
        const resp = await axios.get('/api/students');
        this.students = resp.data.data;
    }
};
export default options;
</script>
```

- `<el-table>` 标签中对 data 属性赋值
- label 属性是列表名，prop 值



### 1.3、分页组件

#### 1.3.1、介绍

```vue
<!-- ElementUI -->
<template>
    <div>
        <el-table :data="students">
            <el-table-column label="编号" prop="id"></el-table-column>
            <el-table-column label="姓名" prop="name"></el-table-column>
            <el-table-column label="性别" prop="sex"></el-table-column>
            <el-table-column label="年龄" prop="age"></el-table-column>
        </el-table>
        <el-pagination
            :total="50"
            :page-size="10"
            :current-page="2"
            layout="prev,pager,next,sizes,->,total"
            :page-sizes="[5,10,15,20]"
         ></el-pagination>
    </div>
</template>
<script>
import axios from "../util/myaxios";
const options = {
    data() {
        return {
            students: []
        };
    },
    async mounted() {
        const resp = await axios.get('/api/students');
        this.students = resp.data.data;
    }
};
export default options;
</script>
```

- `<el-pagination>` 代表分页插件
- total：总记录数
- page-size：每页显示的条数
- current-page：当前页码
- layout：分页组件显示的控件
  - prev：前一页
  - pager：页码列表
  - next：下一页
  - jumper：跳转到指定页
  - sizes：自定义每页的条数
    - 可以通过 page-sizes 数组属性自定义
  - ->：让总记录数在最右侧，其余在最左侧
  - total：总记录数



有的标签属性在赋值时有 `:` ，有的没有：

- `<el-table>` 标签中的 `:data="students"` 表示从 JavaScript 中获取值
- `<el-table-column>` 标签中都的 `label="编号" prop="id"` 表示字符串就是值，如果加了 `:` 也会从 JavaScript 中取值
- `<el-pagination>` 标签中的 `:total="50"` 这些都加了 `:`，也是属性绑定。不同的是，如果在 JavaScript 中找不到对应的属性，就会将值作为表达式进行计算。比如 `total` 属性的数据类型是 50，就会把 50 转成数字类型。
  - 如果加 `:` ，就会认为是一个普通字符串，不会再进行解析了



#### 1.3.2、练习

```vue
<!-- ElementUI -->
<template>
    <div>
        <el-table :data="students">
            <el-table-column label="编号" prop="id"></el-table-column>
            <el-table-column label="姓名" prop="name"></el-table-column>
            <el-table-column label="性别" prop="sex"></el-table-column>
            <el-table-column label="年龄" prop="age"></el-table-column>
        </el-table>
        <el-pagination
            :total="total"
            :page-size="queryDto.size"
            :current-page="queryDto.page"
            layout="prev,pager,next,sizes,->,total"
            :page-sizes="[5,10,15,20]"
            @current-change="currentChange"
            @size-change="sizeChange"
         ></el-pagination>
    </div>
</template>
<script>
import axios from "../util/myaxios";
const options = {
    data() {
        return {
            students: [],
            total: 0,
            queryDto: {
                page: 1,
                size: 5
            }
        };
    },
    methods: {
        currentChange(page) {
            this.queryDto.page = page;
            this.query();
        },
        sizeChange(size) {
            this.queryDto.size = size;
            this.query();
        },
        async query() {
            const resp = await axios.get('/api/students/q', {
                params: this.queryDto
            });
            this.students = resp.data.data.list;
            this.total = resp.data.data.total;
        }
    },
    mounted() {
        this.query();
    }
};
export default options;
</script>
```

- 触发查询的三种情况
  - mounted 组件挂载完成后
  - 页号变化时
  - 页大小变化时
- 查询传参应该根据后台需求，灵活采用不同方式
  - 本例中是 get 请求，无法采用请求体，只能通过 params 方式传参
- 返回响应的格式也许会很复杂，需要掌握【根据返回的响应结构，获取数据】的能力



### 1.4、分页搜索

```vue
<!-- ElementUI -->
<template>
    <div>
        <el-input placeholder="请输入姓名" size="mini" v-model="queryDto.name"></el-input>
        <el-select placeholder="请选择性别" size="mini" v-model="queryDto.sex" clearable>
            <el-option value="男"></el-option>
            <el-option value="女"></el-option>
        </el-select>
        <el-select placeholder="请选择年龄" size="mini"  v-model="queryDto.age" clearable>
            <el-option value="0,20" label="0到20岁"></el-option>
            <el-option value="21,30" label="21到30岁"></el-option>
            <el-option value="31,40" label="31到40岁"></el-option>
            <el-option value="41,120" label="41到120岁"></el-option>
        </el-select>
        <el-button type="primary" size="mini" @click="search">搜索</el-button>
        <el-divider></el-divider>
        <el-table :data="students">
            <el-table-column label="编号" prop="id"></el-table-column>
            <el-table-column label="姓名" prop="name"></el-table-column>
            <el-table-column label="性别" prop="sex"></el-table-column>
            <el-table-column label="年龄" prop="age"></el-table-column>
        </el-table>
        <el-pagination
            :total="total"
            :page-size="queryDto.size"
            :current-page="queryDto.page"
            layout="prev,pager,next,sizes,->,total"
            :page-sizes="[5,10,15,20]"
            @current-change="currentChange"
            @size-change="sizeChange"
         ></el-pagination>
    </div>
</template>
<script>
import axios from "../util/myaxios";
const options = {
    data() {
        return {
            students: [],
            total: 0,
            queryDto: {
                name: '',
                sex: '',
                age: '',
                page: 1,
                size: 5
            }
        };
    },
    methods: {
        currentChange(page) {
            this.queryDto.page = page;
            this.query();
        },
        sizeChange(size) {
            this.queryDto.size = size;
            this.query();
        },
        async query() {
            const resp = await axios.get('/api/students/q', {
                params: this.queryDto
            });
            this.students = resp.data.data.list;
            this.total = resp.data.data.total;
        },
        search() {
            this.query();
        }
    },
    mounted() {
        this.query();
    }
};
export default options;
</script>
<style scoped>
.el-input--mini,
.el-select--mini {
    width: 193px;
    margin: 10px 10px 0 0;
}
</style>
```

- sex 与 age 均用 `''` 表示用户没有选择的情况
- age 取值 `0,20` 会被 spring 转换为 new int[]{0,20}
- age 取值 `''` 会被 spring 转换为 `new int[0]`



### 1.5、级联选择

级联选择器中选项的数据结构为

```js
[
    {value:100, label:'主页',children:[
        {value:101, label:'菜单1', children:[
            {value:105, label:'子项1'},
            {value:106, label:'子项2'}
        ]},
        {value:102, label:'菜单2', children:[
            {value:107, label:'子项3'},
            {value:108, label:'子项4'},
            {value:109, label:'子项5'}
        ]},
        {value:103, label:'菜单3', children:[
            {value:110, label:'子项6'},
            {value:111, label:'子项7'}
        ]},
        {value:104, label:'菜单4'}
    ]}
]
```

下面将后端返回的一维数组【树化】

```vue
<!-- cascader -->
<template>
    <el-cascader :options="ops" clearable></el-cascader>
</template>
<script>
import axios from "../util/myaxios";
const options = {
    data() {
        return {
            ops: []
        }
    },
    async mounted() {
        const resp = await axios.get('/api/menu');
        console.log(resp.data.data);
        const array = resp.data.data;
        const map = new Map(); // get set
        // 1. 将所有数据存入 map 集合(为了接下来查找效率)
        for (const {id, name, pid} of array) {
            map.set(id, {value: id, label: name, pid: pid});
        }
        // 2. 建立父子关系
        // 3. 找到顶层对象
        const top = [];
        for (const obj of map.values()) {
            const parent = map.get(obj.pid);
            if (parent !== undefined) {
                parent.children ??= [];
                parent.children.push(obj);
            }else {
                top.push(obj);
            }
            console.log(map);
        }
        this.ops = top;
    }
};
export default options;
</script>
```



## 2、Vue-Router

vue 属于单页面应用，所谓路由，就是根据浏览路径不同，用不同的**视图组件**替换这个页面内容展示



### 2.1、配置路由

新建一个路由 js 文件，例如 src/router/example14.js，内容如下：

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import ContainerView from '@/views/example14/ContainerView.vue'
import LoginView from '@/views/example14/LoginView.vue'
import NotFoundView from '@/views/example14/NotFoundView.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    component: ContainerView
  },
  {
    path: '/login',
    component: LoginView
  },
  {
    path: '/404',
    component: NotFoundView
  }
]

const router = new VueRouter({
  routes
})

export default router

```

- 最重要的就是建立了【路径】与【视图组件】之间的映射关系
- 本例中映射了 3 个路径与对应的视图组件

在 main.js 中采用我们的路由 js

```js
import Vue from 'vue'
import e14 from './views/Example14View.vue'
import router from './router/example14'
import store from './store'
import Element from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.config.productionTip = false

Vue.use(Element)

new Vue({
  router,
  store,
  render: h => h(e14)
}).$mount('#app')
```

根组件是 Example14View.vue，内容为

```vue
<template>
    <div class="all">
        <router-view></router-view>
    </div>
</template>
<style scoped>
.all {
    height: 100%;
    background-color: darksalmon;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg'%3E%3Ctext x='22' y='10' font-size='14' font-family='system-ui, sans-serif' text-anchor='middle' dominant-baseline='middle'%3E最外层%3C/text%3E%3C/svg%3E");
    padding: 20px;
    box-sizing: border-box;
}
</style>
```

- 其中 `<router-view>` 起到占位作用，改变路径后，这个路径对应的视图组件就会占据 `<router-view>` 的位置，替换掉之前的内容



### 2.2、动态导入

