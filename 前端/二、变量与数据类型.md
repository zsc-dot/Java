JavaScript是一种脚本语言，可以用来更改页面内容，控制多媒体，制作图像、动画等。

例子：

- 修改页面内容

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>js入门</title>
  </head>
  <body>
      <p id="p1">11111</p>
  </body>
  </html>
  ```

  在浏览器控制台中输入

  ```js
  document.getElementById("p1")
  ```

  显示：

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415111713702.png" alt="image-20240415111713702" style="zoom:80%;" />

  将该方法的返回值赋给变量a，然后通过a进行元素内容的修改：

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240415112012008-1713151294604.png" alt="image-20240415112012008" style="zoom:80%;" />

  注意：页面刷新后，内容会变成之前的，不会永久保存。



js代码位置：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>js入门</title>
</head>
<body>
    <p id="p1">11111</p>
    <script>
        let a = document.getElementById("p1");
        a.innerText = "33333";
    </script>
</body>
</html>
```

引入 js 脚本

```html
<script src="js脚本路径"></script>
```

- 注意：到了框架之后，引入方式会有不同

  ```js
  let a = document.getElementById("p1");
  a.innerText = "44444";
  ```

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>js入门</title>
  </head>
  <body>
      <p id="p1">11111</p>
      <script src="1.js"></script>
  </body>
  </html>
  ```



# 一、变量与数据类型



## 1、声明变量

### 1.1、let:star:

```js
let 变量名 = 值;
```

let声明的变量可以被多次赋值

```js
let a = 100;  // 初始值是 100
a = 200;	  // ok, 被重新赋值为 200
```



### 1.2、const:star:

- const修饰的叫常量，只能赋值一次

```js
const b = 300; // 初始值是 300
b = 400;	   // error, 不能再次赋值
```

- const 并不意味着它引用的内容不可修改，例如

```js
const c = [1,2,3];
c[2] = 4; 	        // ok, 数组内容被修改成 [1,2,4]
c = [5,6];			// error, 不能再次赋值
```



### 1.3、var:star:

var 声明的变量可以被多次赋值，例如

```js
var f = 100;
f = 200;
```



## 2、基本数据类型

### 2.1、undefined和null

- 执行表达式或函数，没有返回结果，出现undefined
- 访问数据不存在的元素，访问对象不存在的属性，出现undefined
- 定义变量，没有初始化，出现undefined

例如：

```js
console.log(1);  	// 函数没有返回值, 结果是  undefined
let a = 10;		 	// 表达式没有返回值, 结果是 undefined
let b = [1,2,3];
console.log(b[10]); // 数组未定义元素是 undefined
let c = {"name":"张三"};
console.log(c.age); // 对象未定义属性是 undefined
let d;
console.log(d);		// 变量未初始化是 undefined
```

二者共同点

* 都没有属性、方法
* 二者合称 Nullish

二者区别

* undefined 由 js 产生
* null 由程序员提供



### 2.2、string:star:

js字符串三种写法：

```js
let a = "hello"; // 双引号
let b = 'world'; // 单引号
let c = `hello`; // 反引号
```



html 代码如下，用 java 和 js 中的字符串如何表示？

```html
<a href="1.html">超链接</a>
```

java 显得比较繁琐

```java
String s1 = "<a href=\"1.html\">超链接</a>"
```

js 就比较灵活

```js
let s1 = '<a href="1.html">超链接</a>';
let s2 = `<a href="1.html">超链接</a>`;
```



模板字符串（Template strings）

需求：拼接 URI 的请求参数，如

```txt
/test?name=zhang&age=18
/test?name=li&age=20
```

传统方法拼接：

```js
let name = ; // zhang li ...
let age = ; // 18 20 ...

let uri = "/test?name=" + name + "&age=" + age;
```

模板字符串方式：

```js
let name = ; // zhang li ...
let age = ; // 18 20 ...

let uri = `/test?name=${name}&age=${age}`;
```



### 2.3、number和bigint:star:

number类型标识的是双精度浮动小数，例如

```js
10 / 3;   // 结果 3.3333333333333335
```

既然是浮点小数，那么可以除0

```js
10 / 0;	  // 结果 Infinity 正无穷大
-10 / 0;  // 结果 -Infinity 负无穷大
```

浮点小数都有运算精度问题，例如

```js
2.0 - 1.1; // 结果 0.8999999999999999
```

字符串转数字

```js
parseInt("10"); 	// 结果是数字 10 
parseInt("10.5");	// 结果是数字 10, 去除了小数部分
parseInt("10") / 3; // 结果仍视为 number 浮点数, 因此结果为 3.3333333333333335

parseInt("abc");	// 转换失败，结果是特殊值 NaN (Not a Number)
```

要表示真正的整数，需要用 bigint，数字的结尾用 n 表示它是一个 bigint 类型

```js
10n / 3n;			// 结果 3n, 按整数除法处理
```



### 2.4、boolean:star:

- Truthy
- Falsy

在 js 中，并不是 boolean 才能用于条件判断，你可以在 if 语句中使用【数字】、【字符串】... 作为判断条件

```js
let b = 1;

if(b) { // true
    console.log("进入了");
}
```



这时就有一个规则，当需要条件判断时，这个值被当作 true 还是 false，当作 true 的值归类为 truthy，当作 false 的值归类为 falsy

下面值都是 falsy

* `false`
* `Nullish (null, undefined)`
* `0, 0n, NaN`
* ``` "" '' `` ```  即长度为零的字符串

剩余的值绝大部分都是 truthy

有几个容易被当作 falsy 实际是 truthy 的

* `"false", "0"` 即字符串的 false 和 字符串的零
* `[]` 空数组
* `{}` 空对象



### 2.5、symbol

很少用



## 3、对象类型

### 3.1、Function:star::star:

#### 1、定义函数

```js
function 函数名(参数) {
    // 函数体
    return 结果;
}
```

例

```js
function add(a, b) {
    return a + b;
}
```



#### 2、调用函数

```js
函数(实参)
```

例子

```js
add(1, 2); // 3
```

js中的函数调用特点：对参数的**类型**和**个数**都没限制，例如

```js
add('a', 'b');  // 返回 ab
add(4, 5, 6);   // 返回 9, 第三个参数没有被用到, 不会报错
add(1);			// 返回 NaN, 这时 b 没有定义是 undefined, undefined 做数学运算结果就是 NaN
```



#### 3、默认参数

java 中（spring）要实现默认参数的效果得这么做：

```java
@RestController 
public class MyController {
    
    @RequestMapping("/page")
    @ResponseBody
    public void page(
        @RequestParam(defaultValue="1") int page, 
        @RequestParam(defaultValue="10") int size
    ){
        // ...
    }
}
```

js

```js
function pagination(page = 1, size = 10) {
    console.log(page, size);
}
```



#### 4、匿名函数

语法

```js
(function (参数) {
    // 函数体
    return 结果;
})
```

例

```js
(function(a, b) {
    return a+b;
})
```

第一种场景：定义完毕后立刻调用

```js
(function(a, b){
    return a+b;
})(1, 2)
```

第二种场景：作为其他对象的方法

页面有元素

```html
<p id="p1">点我啊</p>
```

此时有一个onclick方法，会在鼠标单击这个元素后被执行，onclick方法刚开始是null，需要赋值后才能使用

```js
document.getElementById("p1").onclick = (function(){
    console.log("鼠标单击了...");
})
```



#### 5、箭头函数

```js
(参数) => {
    // 函数体
    return 结果;
}
```

- 如果没有参数，()还是要保留
- 如果只有一个参数，()可以省略
- 如果函数体内只有一行代码，{}可以省略
- 如果这一行代码就是结果，return可以省略

例

```js
document.getElementById("p1").onclick = () => console.log("aa");
```



#### 6、函数是对象

以下形式在js中非常常见！

1. 可以参与赋值，例如：具体函数也能参与赋值

   ```js
   function abc() {
       console.log("bb");
   }
   document.getElementById("p1").onclick = abc;
   ```

2. 有属性、有方法，执行`console.dir(abc)`，输出结果如下：

   ```js
   ƒ abc()
       arguments: null
       caller: null
       length: 0
       name: "abc"
       ➡prototype: {constructor: ƒ}
       [[FunctionLocation]]: VM1962:1
       ➡[[Prototype]]: ƒ ()
       ➡[[Scopes]]: Scopes[1]
   ```

   - 其中带有 f 标记的是方法，不是属性
   - 带有 ➡ 符号的可以继续展开，限于篇幅省略了
   - 带有`[[ ]]`的是内置属性，不能访问，只能查看
   - 相对重要的是`[[Prototype]]`和`[[Scopes]]`会在后面继承和作用域时讲到

3. 可以作为方法参数

   ```js
   function a() {
       console.log('a')
   }
   function b(fn) {      // fn将来可以是一个函数对象
       console.log('b')
       fn() // 调用函数对象
   }
   b(a)
   ```

4. 可以作为方法返回值

   ```js
   function c() {
       console.log("c");
       function d() {
           console.log("d");
       }
       return d;
   }
   c()()
   ```

注意：

如果一个函数A的参数可以接受函数或返回值是一个函数，那么函数A被称为高阶函数




#### 7、函数作用域

函数可以嵌套（js 代码中很常见，只是嵌套的形式更多是匿名函数，箭头函数）

```js
function a() {
    function b() {
    }
}
```

看下面的例子

```js
function c() {
    var z = 30;
}
var x = 10;
function a() {
    var y = 20;
    function b() {
        // 看这里
        console.log(x, y);
    }
    b();
    console.dir(b);
}
a();
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240416100209115.png" alt="image-20240416100209115" style="zoom:80%;" />

b函数有两层作用域，下标为0的是a函数作用域，Global是全局作用域。注意，[[Scopes]]作用域不会打印b函数本身的作用域。

- 以函数为分界线划定作用域，所有函数之外是全局作用域
- 查找变量时，由内向外查找
  - 在内层作用域找到变量，就会停止查找，不会再找外层
  - 所有作用域都找不到变量，报错
- 作用域本质上是函数对象的属性，可以通过console.dir来查看调试



#### 8、闭包

```js
var x = 10;
function a() {
    var y = 20;
    function b() {
        console.log(x, y);
    }
    return b;
}
a()(); // 在外面执行了b
```

- 函数定义时，它的作用域已经确定好了，因此无论函数将来去了哪，都能从它的作用域中找到当时那些变量
- 闭包就是指**函数能够访问自己的作用域中变量**



#### 9、let、var 与作用域

如果函数外层引用的是 let 变量，那么外层普通的 {} 也会作为作用域边界，最外层的 let 也占一个script 作用域

```js
let x = 10;
if (true) {
    let y = 20;
    function b() {
        console.log(x, y);
    }
    console.dir(b); // 三层作用域，let y一层，let x一层，global一层
}
```

如果函数外层引用的是var变量，外层普通的 {} 不会视为边界

```js
var x = 10;
if (true) {
    var y = 20;
    function b() {
        console.log(x, y);
    }
    console.dir(b); // 一层作用域，因为函数 b 外层没有其他函数了，就只有一层global作用域
}
```

如果var变量出现了重名，则他俩会被视为同一作用域中的同一个变量

```js
var e = 10;
if (true) {
    var e = 20;
    console.log(e); // 打印20
}
console.log(e); // 因为是同一个变量，还是打印20
```

如果是let，则视为两个作用域中的两个变量

```js
let e = 10;
if (true) {
    let e = 20;
    console.log(e); // 打印20
}
console.log(e); // 打印10
```

要想里面的 e 和外面的 e 能区分开来，最简单的办法是改成let，或者用函数来界定作用域范围

```js
var e = 10;
if (true) {
    function b() {
        var e = 20;
        console.log(e);
    }
    b();
}
console.log(e);
```



### 3.2、Array:star:

语法

```js
// 创建数组
let arr = [1, 2, 3];
//获取数组元素
console.log(arr[0]);
// 修改数组元素
arr[0] = 5;
// 遍历数组元素，其中 length 是数组属性，代表数组长度
for(let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
}
```



API

- push、shift、splice

  ```js
  let arr = [1, 2, 3];
  
  arr.push(4); // 向数组尾部(右侧)添加元素，结果[1, 2, 3, 4]
  arr.shift(); // 从数组头部(左侧)移除元素，结果[2, 3, 4]
  arr.splice(1, 1); // 删除【参数1】下标位置的【参数2】个元素，结果[2, 4]
  ```

- join

  ```js
  let arr = ['a', 'b', 'c'];
  
  arr.join(); // 默认使用【,】作为连接符，结果'a, b, c'
  arr.join(''); // 结果'abc'
  arr.join('-'); // 结果'a-b-c'
  ```

- map、filter、forEach

  ```js
  let arr = [1, 2, 3, 6]
  
  function a(i) { // 代表新旧元素之间的变换规则
      return i * 10;
  }
  
  // arr.map(a) // 具名函数，结果[10, 20, 30, 60]
  // arr.map( (i) => return i * 10 ); // 箭头函数
  arr.map( i => i * 10); // 箭头函数
  ```

- 传给 map 的函数，参数代表旧元素，返回值代表新元素

  map的内部实现(伪代码)

  ```js
  function map(a) { // 参数是一个函数
      let arr = [];
      for (let i = 0; i < arr.length; i++) {
          let o = arr[i]; // 旧元素
          let n = a(o); // 新元素
          narr.push(n);
      }
      return arr;
  }
  ```

  filter例子

  ```js
  let arr = [1, 2, 3, 6];
  arr.filter( (i) => i % 2 == 1 ); // 结果[1, 3]
  ```

- 传给 filter 的函数，参数代表旧元素，返回 true 表示要留下的元素

  forEach例子
  
  ```js
let arr = [1,2,3,6];
  
  /*for(let i = 0; i < arr.length; i++) {
      console.log(arr[i]);
  }*/
  
  arr.forEach( (i) => console.log(i) );
  ```
  

两个例子：

- 高阶函数，map，filter，forEach
- 回调函数，例如作为参数传入的函数



### 3.3、Object:star::star:

#### 1、语法

```js
let obj = {
    属性名: 值，
    方法名: 函数,
    get 属性名() {},
    set 属性名(新值) {}
}
```

例1

```js
let stu1 = {
    name: "小明",
    age: 18,
    study: function() {
        console.log(this.name + "爱学习");
    }
}

stu1.name; // 小明
stu1.gae; // 18
stu1.study(); // 控制台打印日志
```

例2

```js
let name = "小黑";
let age = 20;
let study = function() {
    console.log(this.name + "爱学习");
}

let stu2 = {name, age, study}
```

例3

```js
let stu3 = {
    name: "小白",
    age: 18,
    study() {
        console.log(this.name + "爱学习");
    }
}
```

- **注意**：对象方法这么写，仅限于对象内部

例4

```js
let stu4 = {
    _name: null, // 类似于java中的私有成员变量
    get name() {
        console.log("进入了get");
        return this._name; // this不能省略
    },
    set name(name) {
        console.log("进入了set");
        this._name = name;
    }
}
```

调用get，set

```js
stu4.name = "小白"
console.log(stu4.name)
```



#### 2、特色：属性增删

对比一下Java中的Object

- Java的Object是以类作为模板来创建，对象不能脱离类模板的范围，一个对象的属性、能用的方法都是确定好的
- js的对象，不需要模板，它的属性和方法可以随时加减

```js
let stu = {name: "张三"};
stu.age = 18; // 添加属性
delete stu.age; // 删除属性

stu.study = function() { // 添加方法
    console.log(this.name + "爱学习");
}
```

添加get、set，需要借助Object.definePropery

```js
let stu = {_name: null};
Object.definePropery(stu, "name", {
    get() {
        return this._name;
    }
    set(name) {
    	this._name = name;
	}
})
```

- 参数1：目标对象
- 参数2：属性名
- 参数3：get、set的定义



#### 3、特色：this

先来对Java中的this有个理解

```java
public class TestMethod {

    static class Student {
        private String name;

        public Student(String name) {
            this.name = name;
        }

        public void study(Student this, String subject) {
            System.out.println(this.name + "在学习 " + subject);
        }
    }

    public static void main(String[] args) {
        Student stu = new Student("小明");
        
        // 下面的代码，本质上是执行 study(stu, "java")，因此 this 就是 stu
        stu.study("java"); 
    }
}
```

- Java中的this是个隐式参数
- Java中，我们说this代表的就是调用方法的那个对象



js中的this也是隐式参数，但它与函数运行时上下文相关

例如，一个“落单”的函数

```js
function study(subject) {
    console.log(this.name + "在学习 " + subject);
}
```

测试一下

```js
study("js");
```

这是因为此时函数执行，全局对象window被当作了this，window对象的name属性是空串



同样的函数，如果作为对象的方法

```js
let stu = {
    name: "小白",
    syudy
}
```

这种情况下，会将当前对象作为this

```js
stu.study("js"); // 输出 小白在学习 js
```



还可以动态改变this

```js
let stu = {name: "小黑"};
study.call(stu, "js"); // // 输出 小黑在学习 js
```

这回执行study时，就把call的第一个参数stu作为this



一个例外是，在**箭头函数**内出现的this，以最外层this理解

用匿名函数

```js
let stu = {
    name: "小花",
    friends: ["小白", "小黑", "小明"],
    play() {
        this.friends.forEach(function(e) {
            console.log(this.name + "与" + e + "在玩耍");
        });
    }
}
stu.play();
```

- this.name所在的函数是【落单】的函数，因此this代表window

输出结果为

```txt
与小白在玩耍
与小黑在玩耍
与小明在玩耍
```

用箭头函数

```js
let stu = {
    name: "小花",
    friends: ["小白","小黑","小明"],
    play() {
        this.friends.forEach(e => {
            console.log(this.name + "与" + e + "在玩耍");
        })
    }    
}
```

- this.name所在函数是箭头函数，因此 this 要看它最外层的 play 函数，play 又是属于 stu 的方法，因此 this 代表 stu 对象

输出结果为

```txt
小花与小白在玩耍
小花与小黑在玩耍
小花与小明在玩耍
```

不用箭头函数的做法

```js
let stu = {
    name: "小花",
    friends: ["小白","小黑","小明"],
    play() {
        let me = this;
        this.friends.forEach(function(e) {
            console.log(me.name + "与" + e + "在玩耍");
        });
    }
}
```



#### 4、特色：原型继承

java中的继承是类之间的，js中的继承是对象之间的

```js
let father = {
    f1: '父属性',
    m1: function() {
        console.log("父方法");
    }
}
let son = Object.create(father);
console.log(son.f1); // 打印 父属性
son.m1(); // 调用父方法
```

- father 是父对象，son 去调用 .m1 或.f1 时，自身对象没有，就到父对象找
- son 自己可以添加自己的属性和方法
- son 里有特殊属性`__proto__`代表它的父对象，js术语：son的原型对象
- 不同浏览器对打印son的`__proto__`属性时显示不同
  - Edge打印console.dir(son) 显示 `[[Prototype]]`
  - FireFox打印console.dir(son) 显示`<prototype>`



#### 5、特色：基于函数的原型继承

处于方便的原因，js又提供了一种基于函数的原型继承

> **函数职责**
>
> 1. 负责创建子对象，给子对象提供属性、方法，功能上相当于构造方法
> 2. 函数有个特殊的属性 prototype，它就是函数创建的子对象的父对象
>
> 注意：名字有差异，这个属性的作用就是为新对象提供原型

```js
function cons(f2) {
    // 创建子对象(this)，给子对象提供属性和方法
    this.f2 = f2;
    this.m2 = function() {
        console.log("子方法");
    }
}
// cons.prototype 就是父对象
cons.prototype.f1 = "父属性";
cons.prototype.m1 = function() {
    console.log("父方法");
}
```

配合new关键字，创建子对象

```js
let son = new cons("子属性");
```

子对象的`__proto__`就是函数的`prototype`属性



#### 6、json与js对象

之前我们讲 http 请求格式时，讲过 json 这种数据格式，它的语法看起来与 js 对象非常相似，例如：

一个 json 对象可以长这样：

```json
{
    "name":"张三",
    "age":"18"
}
```

一个 js 对象长这样：

```js
{
    name: "张三",
    age: 18
}
```

那么他们的区别在哪儿呢？我总结了这么几点

1. 本质不同
   - json对象本质上是个字符串，它的职责是作为客户端和服务器之间传递数据的一种格式，它的属性只是样子货
   - js 对象是切切实实的对象，可以有属性方法
2. 语法细节不同
   - json 中只能有 null、true|false、数字、字符串（只有双引号）、对象、数组
   - json 中不能有除以上的其它 js 对象的特性，如方法等
   - json 中的属性必须用双引号引起来

json字符串与js对象的转换

```js
JSON.parse(json字符串);  // 返回js对象
JSON.stringify(js对象);  // 返回json字符串
```



#### 7、动态类型

静态类型语言，如Java，值有类型，变量也有类型。赋值给变量时，类型要相符。

```java
int a = 10;
String b = "abc";
int c = "abc"; // 会提示错误
```

而js 属于动态类型语言，值有类型，但变量没有类型，赋值给变量时，没有要求

```js
let a = 200;

let b = 100;
b = 'abc';
b = true;
```

动态类型看起来比较灵活，但变量没有类型，会给后期维护带来困难

```js
function test(obj) {
    // obj 的类型未知，必须根据不同类型做出相应的容错处理
}
```



# 二、运算符与表达式

- `+ - * / % ** `
- `+= -= *= /= %= **=`
- `++ --`
- 位运算、移位运算
- `== != > >= < <=`
- `=== !==` :star:
- `&& || !` :star:
- `?? ?.` :star:
- `...` :star:
- 解构赋值 :star:



## 1、===

严格相等运算符，用作逻辑判等

```js
1 == 1    	// 返回 true 
1 == '1'	// 返回 true，会先将右侧的字符串转为数字，再做比较
1 === '1'	// 返回 false，类型不等，直接返回 false
```

typeof 查看某个值的类型

```js
typeof 1	// 返回 'number'
typeof '1'	// 返回 'string'
```



## 2、||

需求，如果参数 n 没有传递，给它一个【男】

**推荐**做法

```js
function test(n = '男') {
    console.log(n);
}
```

你可能的做法

```js
function test(n) {
    if (n === undefined) {
        n = '男';
    }
    console.log(n);
}
```

还可能是这样

```js
function test(n) {
    n = (n === undefined) ? '男' : n;
    console.log(n);
}
```

一些老旧代码中可能的做法（不推荐）

```js
function test(n) {
    n = n || '男';
    console.log(n);
}
```

它的语法是

```js
值1 || 值2
```

如果值1 是Truthy，返回值1。如果值1 是Falsy，返回值2。



## 3、?? 与 ?.

### 3.1、??

需求，如果参数n 没有传递或是null，给它一个【男】

如果用传统办法

```js
function test(n) {
    if (n === undefined || n === null) {
        n = '男';
    }
    console.log(n);
}
```

用??

```js
function test(n) {
    n = n ?? '男';
    console.log(n);
}
```

语法

```js
值1 ?? 值2
```

- 值1 是nullish，返回值2
- 值1 不是nullish，返回值1



### 3.2、?.

需求，函数参数是一个对象，可能包含有子属性

例如，参数可能是

```js
let stu1 = {
    name: "张三",
    address: {
        city: "北京"
    }
};

let stu2 = {
    name: "李四"
}

let stu3 = {
    name: "李四",
    address: null
}
```

现在要访问子属性（有问题）

```js
function test(stu) {
    console.log(stu.address.city);
}
```

现在希望当某个属性是nullish时，短路并返回undefined，可以用?.

```js
function test(stu) {
    console.log(stu.address?.city);
}
```

用传统办法

```js
function test(stu) {
    if(stu.address === undefined || stu.address === null) {
        console.log(undefined);
        return;
    }
    console.log(stu.address.city);
}
```



## 4、...

展开运算符

#### 4.1、打散数组

打散数组，把元素传递给多个参数

```js
let arr = [1, 2, 3];

function test(a, b ,c) {
    console.log(a, b, c);
}
```

需求：把数组元素依次传递给函数参数

传统写法

```js
test(arr[0], arr[1], arr[2]); // 输出 1, 2, 3
```

展开符写法

```js
test(...arr); // 输出 1, 2, 3
```

- 打散可以理解为【去掉了】数组外侧的中括号，只剩下数组元素



#### 4.2、复制数组或对象

数组

```js
let arr1 = [1, 2, 3];
let arr2 = [...arr1]; // 复制数组
```

对象

```js
let obj1 = {name: "张三", age: 18};
let obj2 = {...obj1}; // 复制对象
```

**注意**：展开运算符复制属于浅拷贝，例如：

```js
let o1 = {name: '张三', address: {city: '北京'}};
let o2 = {...o1};
```



#### 4.3、合并数组或对象

合并数组

```js
let a1 = [1, 2];
let a2 = [3, 4];

let b1 = [...a1, ...a2]; // 结果[1, 2, 3, 4]
let b2 = [...a2, 5, ...a1]; // 结果[3,4,5,1,2]
```

合并对象

```js
let o1 = {name: '张三'};
let o2 = {age: 18};
let o3 = {name: '李四'};

let n1 = {...o1, ...o2}; // 结果{name:'张三',age:18}
let n2 = {...o1, ...o2, ...o3}; // 结果{name:'李四',age:18}
```

- 复制对象时出现同名属性，后面的会覆盖前面的



## 5、[] 与 {}

解构赋值

### 5.1、[]

用在声明变量时

```js
let arr = [1, 2, 3];
let [a, b, c] = arr; // 结果 a=1  b=2  c=3
```

用在声明参数时

```js
let arr = [1, 2, 3];

function test([a, b, c]) {
    console.log(a, b, c); // 结果 1, 2, 3
}
test(arr);
```



### 5.2、{}

用在声明变量时

```js
let obj = {name:"张三", age:18};

let {name, age} = obj; // 结果 name=张三, age=18
```

用在声明参数时

```js
let obj = {name:"张三", age:18};
function test({name, age}) {
    console.log(name, age); // 张三, 18
}
test(obj);
```



# 三、控制语句

- `if ... else`
- `switch`
- `while`
- `do ... while`
- `for` 
- `for ... in` :star:
- `for ... of` :star:
- `try ... catch` :star:



## 1、for  in

主要用来遍历对象

```js
let father = {name: '张三', study: function() {}};
for(const n in father) {
    console.log(n);
}
```

- 其中const n代表遍历出来的属性名

- 方法名也能被遍历出来（它其实算一种特殊属性）

- 遍历子对象时，父对象的属性会跟着遍历出来

  ```js
  let son = Object.create(father);
  son.sex = "男";
  for(const n in son) {
      console.log(n);
  }
  ```

- 在for  in 内获取属性值，要使用 [] 语法，而不能用 . 语法

  ```js
  for(const n in son) {
      console.log(n, son[n]);
  }
  ```



## 2、for  of

主要用来遍历数组，也可以是其他可迭代对象，如Map、Set等

```js
let a1 = [1, 2, 3];

for(const i of a1) {
    console.log(i);
}

let a2 = [
    {name:'张三', age:18},
    {name:'李四', age:20},
    {name:'王五', age:22}
];

for(const obj of a2) {
    console.log(obj.name, obj.age);
}

for(const {name, age} of a2) {
    console.log(name, age);
}
```



## 3、try  catch

```js
let stu1 = {name:'张三', age:18, address: {city:'北京'}};
let stu2 = {name:'张三', age:18};

function test(stu) {
    try{
        console.log(stu.address.city);
    }catch(e) {
        console.log('出现了异常', e.message);
    }finally{
        console.log('finally');
    }
}
```



# 四、API

## 1、环境准备

### 1.1、安装nvm

nvm 即 (node version manager)，好处是方便切换node.js版本

安装注意事项

1. 要卸载掉现有的 nodejs
2. 提示选择 nvm 和 nodejs 目录时，一定要避免目录中出现空格
3. 选用【以管理员身份运行】cmd 程序来执行 nvm 命令
4. 首次运行前设置好国内镜像地址

```cmd
nvm node_mirror http://npm.taobao.org/mirrors/node/
nvm npm_mirror https://npm.taobao.org/mirrors/npm/
```

首先查看有哪些可用版本

```cmd
nvm list available
```

输出

```txt

|   CURRENT    |     LTS      |  OLD STABLE  | OLD UNSTABLE |
|--------------|--------------|--------------|--------------|
|    18.7.0    |   16.16.0    |   0.12.18    |   0.11.16    |
|    18.6.0    |   16.15.1    |   0.12.17    |   0.11.15    |
|    18.5.0    |   16.15.0    |   0.12.16    |   0.11.14    |
|    18.4.0    |   16.14.2    |   0.12.15    |   0.11.13    |
|    18.3.0    |   16.14.1    |   0.12.14    |   0.11.12    |
|    18.2.0    |   16.14.0    |   0.12.13    |   0.11.11    |
|    18.1.0    |   16.13.2    |   0.12.12    |   0.11.10    |
|    18.0.0    |   16.13.1    |   0.12.11    |    0.11.9    |
|    17.9.1    |   16.13.0    |   0.12.10    |    0.11.8    |
|    17.9.0    |   14.20.0    |    0.12.9    |    0.11.7    |
|    17.8.0    |   14.19.3    |    0.12.8    |    0.11.6    |
|    17.7.2    |   14.19.2    |    0.12.7    |    0.11.5    |
|    17.7.1    |   14.19.1    |    0.12.6    |    0.11.4    |
|    17.7.0    |   14.19.0    |    0.12.5    |    0.11.3    |
|    17.6.0    |   14.18.3    |    0.12.4    |    0.11.2    |
|    17.5.0    |   14.18.2    |    0.12.3    |    0.11.1    |
|    17.4.0    |   14.18.1    |    0.12.2    |    0.11.0    |
|    17.3.1    |   14.18.0    |    0.12.1    |    0.9.12    |
|    17.3.0    |   14.17.6    |    0.12.0    |    0.9.11    |
|    17.2.0    |   14.17.5    |   0.10.48    |    0.9.10    |
```

建议安装 LTS（长期支持版）

```cmd
nvm install 16.16.0
nvm install 14.20.0
```

执行 `nvm list` 会列出已安装版本

切换到16.16.0

```cmd
nvm use 16.16.0
```

切换到 14.20.0

```cmd
nvm use 14.20.0
```

安装后 nvm 自己的环境变量会自动添加，但可能需要手工添加 nodejs 的 PATH 环境变量



### 1.2、检查npm

npm是 js 的包管理器，就类似于 Java 界的 maven，要确保它使用的是国内镜像

检查镜像

```cmd
npm get registry
```

如果返回的不是 `https://registry.npm.taobao.org/`，需要做如下设置

```cmd
npm config set registry https://registry.npm.taobao.org/
```



### 1.3、搭建前端服务器

新建一个保存项目的client文件夹，进入文件夹执行

```cmd
npm install express --save-dev
```

- 类似Java中的maven导入依赖，--save-dev表示express只在开发过程中保存，打包时不会打进去

修改package.json文件，package.json 文件相当于 Java 中的 pom.xml 文件

```json
{
  "type": "module",
  "devDependencies": {
    "express": "^4.18.1"
  }
}
```

- 其中 devDependencies 是 npm install --save-dev添加的
- 加`module`是为了一会可以使用 import 语法

编写main.js代码

```js
import express from 'express'
const app = express() // 返回一个express服务器对象

app.use(epress.static('./')) // 指定静态资源目录
app.listen(7070) // 设置服务器监听端口
```

在当前目录下编写静态资源

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>首页</title>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>
```

执行 js 代码(运行前端服务器)

```cmd
node main.js
```



## 2、前端案例

初步效果

![image-20220812103323220](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220812103323220.png)

​	

架构

![image-20220812103219916](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220812103219916.png)

- 前端只有静态页面，使用Express服务器
- 后端使用Tomcat服务器，通过SpringBoot、Mybatis等框架获取数据库数据



### 2.1、查找元素

- document.getElementById：根据 id 值查找一个元素
- [document | 元素].querySelector：根据选择器查找第一个匹配的元素
- [document | 元素].querySelectorAll：根据选择器查找所有匹配元素

例如，有下面的html代码

```html
<div>
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
        <div class="row">
            <div class="col">1</div>
            <div class="col">张三</div>
            <div class="col">男</div>
            <div class="col">18</div>
        </div>
    </div>
</div>
```

执行

```js
document.querySelector('.title'); // <div class="title">学生列表</div>
document.querySelector('.col'); // <div class="col">编号</div>
```

执行

```js
document.querySelectorAll('.col');
/*
  找到的是一个集合
  <div class="col">编号</div>
  <div class="col">姓名</div>
  <div class="col">性别</div>
  <div class="col">年龄</div>
  <div class="col">1</div>
  <div class="col">张三</div>
  <div class="col">男</div>
  <div class="col">18</div>
*/
```

执行

```js
const thead = document.querySelector('.thead');

// 只在 thead 元素范围内找
thead.querySelectorAll('.col');

/*
  找到的是一个集合
  <div class="col">编号</div>
  <div class="col">姓名</div>
  <div class="col">性别</div>
  <div class="col">年龄</div>
*/
```

根据 id 属性查找既可以用

```js
document.getElementById("id值");
```

也可以用

```js
document.querySelector('#id值');
```



### 2.2、修改元素内容

- 元素.innerHTML
- 元素.textContent

例如

```js
document.querySelector('.title').innerHTML = '侠客列表'
```

效果

![image-20240417175328891](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20240417175328891.png)

innerHTML  会解析内容中的标签，显示标签中对应的样式

textContext 不会解析内容中的标签，内容的标签会以字符串形式展示

给 innerHTML 或 textContent 赋值空串，可以实现清空标签内容的效果



### 2.3、利用模板

通过js数组生成前端页面

```html
    <div>
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
        </div>
    </div>

    <template id="tp">
        <div class="row">
            <div class="col">xxx</div>
            <div class="col">xxx</div>
            <div class="col">xxx</div>
            <div class="col">xxx</div>
        </div>
    </template>

    <script>
        let array = [
            {id: 1, name: '张三', sex: '男', age: 18},
            {id: 2, name: '李四', sex: '女', age: 17},
        ];
        const tp = document.getElementById('tp');
        // 获取tp 标签中的内容
        const row = tp.content;
        const [c1, c2, c3, c4] = row.querySelectorAll('.col');
        const tbody = document.querySelector('.tbody');
        for(const {id, name, sex, age} of array) {
            c1.textContent = id;
            c2.textContent = name;
            c3.textContent = sex;
            c4.textContent = age;
            // 复制一份节点，true代表复制内部的所有子元素
            const newNode = document.importNode(row, true);
            // 为tbody 添加子标签
            tbody.appendChild(newNode);
        }
    </script>
```



### 2.4、Fetch API

Fetch API 可以用来获取远程数据，它有两种方式接收结果，同步方式与异步方式

格式

```js
fetch(url, options) // 返回 Promise
```

同步方式

```js
const 结果 = await Promise
// 后续代码
```

- await 关键字必须在一个标记了 async 的function 内来使用
- 后续代码不会在结果返回前执行

异步方式

```js
Promise
	.then(结果 => {...})
// 后续代码
```

- 后续代码不必等待结果返回就可以执行

例：

在 express 服务器上有 student.json 文件

```js
[
    { "id": 1, "name": "张三", "sex": "男", "age": 18 },
    { "id": 2, "name": "李四", "sex": "女", "age": 17 }
]
```

现在用 fetch api 获取这些数据，并展示

同步方式

首先将`let array`数组中的数据放到同级目录下的`students.json`文件中

```json
[
    {"id": 1, "name": "张三", "sex": "男", "age": 18},
    {"id": 2, "name": "李四", "sex": "女", "age": 17},
    {"id": 3, "name": "王五", "sex": "女", "age": 22}
]
```

```html
<script>
    async function findStudents() {
        try {
            // 获取响应对象
            const resp = await fetch('students.json');
            // 获取响应体，按json格式转换为json数组
            const array = await resp.json();

            // 显示数据
            const tp = document.getElementById('tp');
            // 获取tp 标签中的内容
            const row = tp.content;
            const [c1, c2, c3, c4] = row.querySelectorAll('.col');
            const tbody = document.querySelector('.tbody');
            for(const {id, name, sex, age} of array) {
                c1.textContent = id;
                c2.textContent = name;
                c3.textContent = sex;
                c4.textContent = age;
                // 复制一份节点，true代表复制内部的所有子元素
                const newNode = document.importNode(row, true);
                // 为tbody 添加子标签
                tbody.appendChild(newNode);
            }
        } catch (e) {
            console.log(e);
        }
    }
    findStudents();
</script>
```

- fetch('student.json') 内部会发送请求，但响应结果不能立刻返回，因此 await 就是等待响应结果返回
- 其中 resp.json() 也不是立刻能返回结果，它返回的也是 Promise 对象，也要配合 await 获取结果

异步方式

```html
<script>
    fetch('students.json')
        .then(resp => resp.json())
        .then(array => {
        const tp = document.getElementById('tp');
        // 获取tp 标签中的内容
        const row = tp.content;
        const [c1, c2, c3, c4] = row.querySelectorAll('.col');
        const tbody = document.querySelector('.tbody');
        for(const {id, name, sex, age} of array) {
            c1.textContent = id;
            c2.textContent = name;
            c3.textContent = sex;
            c4.textContent = age;
            // 复制一份节点，true代表复制内部的所有子元素
            const newNode = document.importNode(row, true);
            // 为tbody 添加子标签
            tbody.appendChild(newNode);
        }
    })
    .catch(e => console.log(e))
</script>
```

- 第一个 then 是在响应之返回后，才会调用它里面的箭头函数，箭头函数参数即 resp 响应对象
- 第二个 then 是在 json 解析完成后，才会调用它里面的箭头函数，箭头函数参数即解析结果（本例是 array 数组）
- 上一个 then 返回的是 Promise 对象时，才能链式调用下一个 then



#### 1、跨域问题

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814105448882.png" alt="image-20220814105448882" style="zoom:80%;" />

- 只要协议、主机、端口之一不同，就不同源，例如
  - http://localhost:7070/a 和 https://localhost:7070/b 就不同源
- 同源检查是浏览器行为，而且只针对 fetch、xhr 请求
  - 如果是其它客户端，例如 java http client，postman，它们是不做同源检查的
  - 通过表单提交、浏览器直接输入 url 地址这些方式发送的请求，也不会做同源检查
- 更多相关知识请参考
  * [跨源资源共享（CORS） - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)



请求响应头解决

![image-20220814144040703](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814144040703.png)

- fetch 请求跨域，会携带一个 Origin 头，代表【发请求的资源源自何处】，目标通过它就能辨别是否发生跨域
  - 我们的例子中：student.html 发送 fetch 请求，告诉 tomcat，我源自 localhost:7070
- 目标资源通过返回 Access-Control-Allow-Origin 头，告诉浏览器【允许哪些源使用此响应】
  * 我们的例子中：tomcat 返回 fetch 响应，告诉浏览器，这个响应允许源自 localhost:7070 的资源使用

可以在 Java 中使用`@CrossOrigin`解决

```java
@GetMapping("/api/students")
@CrossOrigin("http://localhost:7070")
public R all() {
    return R.ok(studentService.findAll());
}
```



代理解决

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814161532141.png" alt="image-20220814161532141" style="zoom:80%;" />

```cmd
npm install http-proxy-middleware --save-dev
```

在 express 服务器启动代码中加入

```js
import {createProxyMiddleware} from 'http-proxy-middleware'

// ...

app.use('/api', createProxyMiddleware({ target: 'http://localhost:8080', changeOrigin: true }));
```

fetch 代码改为

```js
const resp = await fetch('http://localhost:7070/api/students')
```

或

```js
const resp = await fetch('/api/students')
```



### 2.5、模块化

单个导出const、let、function

```js
export const a = 10;
export let b = 20;
export function c() {
    console.log('c');
}
```



一起导出

```js
const a = 10;
let b = 20;
function c() {
    console.log('c')
}

export {a,b,c}
```



导出default，只能有一个

```js
export const a = 10;
export let b = 20;
export function c() {
    console.log('c')
}

export default b;
```



import语法

```html
<script type="module">
	import 语句 from '绝对路径/相对路径'
</script>

<script type="moudle">
	import {a, b, c} from './1.js'
	console.log(a);
	console.log(b);
	c();
</script>
```

- import 需要遵循同源策略



整个导入

```js
import * as module from './1.js'
console.log(module.a)		// 输出10
console.log(module.b)		// 输出20
module.c()					// 输出c
```



部分导入

```js
import {a,c} from './1.js'
console.log(a)				// 输出10
c()							// 输出c
```



默认导入

```js
import x from './1.js'
console.log(x)				// 输出20
```

