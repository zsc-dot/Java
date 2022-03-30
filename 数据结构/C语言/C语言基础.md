# 一、简介

## 1、机器语言

通过查找指令表，CPU 能将 0 和 1 的组合跟具体的指令挂钩，那么这些 0 和 1 的组合我们称之为机器码，也叫机器语言，属于第一代编程语言，也是 CPU 唯一可以直接读得懂的语言。



## 2、汇编语言

在汇编语言中，引入了大量的助记符来帮助人们编程，然后由汇编编译器将这些助记符转换为机器码，这个转化的过程我们称之为编译。



## 3、C语言

C 语言属于第三代编程语言，第三代编程语言称之为高级语言。 C++、C#、JAVA、Delphi、Python、Object-C、Swift 这些都属于第三代编程语言。

事实上，用 C 语言进行编程，编译器会将 C 语言代码编译成汇编语言，再由汇编语言的编译器编译为机器语言，通常我们看到的可执行文件事实上就是机器语言的形式，进而让 CPU 理解和执行。



## 4、C语言的优势

- 效率高

  我们说 C 语言效率高是针对其他第三代编程语言来讲的，C 语言是编译型语言，源代码最终编译成机器语言，也就是我们所说的可执行文件，从此 CPU 就可以直接执行。

  ![image-20220328094518524](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328094518524.png)

  除了编译型语言，流行的还有解释型语言，像 JAVA，Python，Ruby 这类都是解释型语言。解释型语言不直接编译成机器码，而是将源码转换成中间代码，然后发送给解释器，由解释器逐句翻译给 CPU 来执行。这样做的一个好处就是可以实现跨平台的特性，而缺点就是效率相对要低一些，因为每执行一次都要翻译一次。

  ![image-20220328094625258](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328094625258.png)

- 灵活度高

  C 语言不仅提供多种运算符，还可以完成类似于计算机底层操作的位运算；语法简单，约束少；丰富多变的结构和数据类型；还拥有可以直接操作计算机硬件能力。指针可以说是 C 语言的灵魂，C 语言有多灵活和强大，完全取决于你对指针这一知识点的掌握程度。

- 可移植性高

  可移植性高是指源代码在不需要做改动或只需稍加修改，就能够在其他机器上编译后正确运行。



# 二、打印输出

## 1、HelloWorld

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("Hello world!\n");
    return 0;
}
```

**printf 是格式化输出函数**，C 语言为我们提供了很多基本函数，它们用于实现不同的功能，比如 printf 函数，就是实现格式化输出的功能。没有它，我们压根儿不可能仅用六行代码，就将文本打印到屏幕上。



## 2、转义字符

在 C 语言中，用双引号括起来的内容我们称之为字符串，也就是我们平时所说的文本。

**字符串可以由可见字符和转义字符组成**，像课堂上演示那条鱼的主要组成部分——星号（*），就是可见字符。可见字符就是你输入什么，显示出来就是什么。

而你如果想将一个字符串分为两行来显示，那么你就需要使用到转义字符。

转义字符一般是表示特殊含义的非可见字符，以反斜杠开头：

![image-20220328102321730](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328102321730.png)



## 3、反斜杠的含义

在字符串中反斜杠 + 字符是转义字符，表示特殊含义。

但反斜杠如果后边不带任何字符（直接换行），表示我们希望 C 语言将该行以及下一行看做是一个整体。

比如：

```c
#include <stdio.h>

int main()
{
        printf("Hello World!\n");
        return 0;
}
```

可以写成：

```c
#in\
clude \
<stdio.h>

int ma\
in()
{
        print\
f("Hello World!\n");
        return \
0;
}
```

这样写毫无意义，这个方法主要用在当你的字符串或语句太长（一行不足以存放，或严重影响阅读），那么你可以通过反斜杠（\）进行“断行”。



# 三、变量

## 1、定义

变量和常量是程序处理的两种基本数据对象。

我们把要让 CPU 处理的数据都放在内存中，但如果你没有给他安排一个位置，而是随意存放，那么你在后边需要再次用到这个数据的时候，就再也找不到它了。

所以变量的意义就是确定目标并提供存放的空间。

![image-20220328104617568](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328104617568.png)

## 2、变量名

为了确定目标，我们需要给变量命名，一旦变量有了名字，我们就可以通过直呼其名的方式来获取它里边存放的数据。

![image-20220328105005432](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328105005432.png)



## 3、命名规范

- C语言变量名只能是英文字母（A-Z，a-z）和数字（0-9）或者下划线（_）组成，其他特殊字母不行。下横线通常用于连接一个比较长的变量名，比如i_love_fishC
- 第一个字母必须是由英文字母或者下划线开头，也就是不能用数字开头。
- 变量名区分大小写。因为C语言是大小写敏感的编程语言，也就是大写的FISHC跟小写的fishc会被认为是不同的两个名字。在传统的命名习惯中，我们用小写字母来命名变量，用大写字母来表示符号常量名。
- 不能使用关键字来命名变量。



## 4、关键字

关键字就是 C 语言内部使用的名字，这些名字都具有特殊的含义。如果你把变量命名为这些名字，那么 C 语言就搞不懂你到底想干嘛了。

传统的 C 语言（ANSI C）有 32 个关键字：

![image-20220328105337641](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328105337641.png)

1999年，ISO 发布 C99，添加了 5 个关键字：

![image-20220328105413223](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328105413223.png)

2011年，ISO 发布 C11，添加了 7 个关键字：

![image-20220328105428665](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328105428665.png)



## 5、常用数据类型

- char —— 字符型，占用一个字节
- int —— 整型，通常反映了所用机器中整数的最自然长度
- float —— 单精度浮点型
- double —— 双精度浮点型



## 6、声明变量的语法

- int a; // 在内存中找到一个整型大小的位置，然后霸占起来，并给它命名叫 a
- char b; //在内存中找到一个字节大小的位置，然后霸占起来，并给它命名叫 b
- float c; //在内存中找到一个单精度浮点型数据大小的位置，然后霸占起来，并给它命名叫 c
- double d; //在内存中找到一个双精度浮点型数据大小的位置，然后霸占起来，并给它命名叫 d



## 7、printf函数用法

参考地址：http://bbs.fishc.com/thread-66471-1-1.html

```c
#include <stdio.h>

int main()
{
        int a = 520;
        char b = 'F';
        float c = 3.14;
        double d = 3.141592653;

        printf("鱼C工作室创办于2010年的%d\n", a);
        printf("I love %cishC.com!\n", b);
        printf("圆周率是：%.2f\n", c);
        printf("精确到小数点后9位的圆周率是：%11.9f\n", d); // 11代表总共11位，9代表小数点后9位

        return 0;
}
```



函数原型：

```c
#include <stdio.h>
...
int printf ( const char * format, ... );
```

参数解析：

1. format 参数

   format 参数是一个格式化字符串，由格式化占位符和普通字符组成。

   格式化占位符（以 % 开头）用于指明输出的参数值如何格式化。

   

   **格式化占位符的语法如下：**

   %{flag}{width}{.precision}{length}specifier。

   每一个格式化占位符均以 % 开始，以转换字符结束。

   **specifier（转换字符，必选）的内容及含义如下：**

   ![image-20220328112019952](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220328112019952.png)

​		注：如果 % 后边的字符不是转换字符，则其行为没有定义。



# 四、常量

## 1、C 语言中常见的常量

- 整型常量：520, 1314, 123
- 实型常量：3.14, 5.12, 8.97
- 字符常量
  - 普通字符：'L', 'o', 'v', 'e'
  - 转义字符：'\n', '\t', '\b'
- 字符串常量："FishC"
- 符号常量：使用之前必须先定义



## 2、定义符号常量（宏定义）

符号常量的定义格式是：

\#define 标识符 常量

其中这个 #define 是一条预处理命令（预处理命令都以"#"开头），我们也称为宏定义命令。它的功能就是把程序中所有出现的标识符都替换为随后的常量。

```c
#include <stdio.h>

#define URL "http://www.fishc.com"
#define NAME "鱼C工作室"
#define BOSS "小甲鱼"
#define YEAR 2010
#define MONTH 5
#define DAY 20

int main()
{
        printf("%s成立于%d年%d月%d日\n", NAME, YEAR, MONTH, DAY);
        printf("%s是%s创立的……\n", NAME, BOSS);
        printf("%s的域名是%s\n", NAME, URL);

        return 0;
}
```

上边的大写字母 URL、NAME、BOSS、YEAR、MONTH、DAY 这些都是符号常量.为了将符号常量和普通的变量名区分开，我们习惯使用全部大写字母来命名符号常量，使用小写字母来命名变量。



## 3、标识符

在 C 语言中，标识符指的就是一切的名字。比如刚刚的符号常量名是标识符，变量名也是一个标识符。以及我们即将学到的函数、数组、自定义类型这些的名字都称之为标识符。



## 4、字符串常量

C 语言用一个特殊的转义字符来表示字符串的结束位置。这样当操作系统读取到这个转义字符的时候，就知道该字符串到此为止了。

这个转义字符就是空字符：'\0'



# 五、数据类型

## 1、数据类型

在 C 语言里，所谓的数据类型就是坑的大小。我们说变量就是在内存里边挖一个坑，然后给这个坑命名。那么数据类型指的就是这个坑的尺寸。C 语言允许使用的类型如下：

![image-20220330094812683](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330094812683.png)



## 2、short 和 long

我们可以为这些基本数据类型加上一些限定符，比如表示长度的 short 和 long。比如 int 经过限定符修饰之后，可以是 short int，long int，还可以是 long long int（这个是 C99 新增加的）。

![image-20220330095051569](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330095051569.png)

这里需要说的是，这里 C 语言并没有限制 int 的坑具体要挖多大，short int 或 long int 的坑又要挖多大。标准只是要求：short int <= int <= long int <= long long int。



## 3、sizeof 运算符

sizeof 用于获得数据类型或表达式的长度，它有三种使用方式：

- sizeof(type_name); //sizeof(类型);
- sizeof(object); //sizeof(对象);
- sizeof object; //sizeof 对象;



```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    int i;
    char j;
    float k;

    i = 123;
    j = 'C';
    k = 3.14;

    printf("size of int is %d\n", sizeof(int)); // 4
    printf("size of i is %d\n", sizeof(i)); // 4
    printf("size of char is %d\n", sizeof(char)); // 1
    printf("size of j is %d\n", sizeof j); // 1
    printf("size of float is %d\n", sizeof(float)); // 4
    printf("size of k is %d\n", sizeof(k));// 4

    return 0;
}
```



## 4、signed 和 unsigned

还有一对类型限定符是 signed 和 unsigned，它们用于限定 char 类型和任何整型变量的取值范围。

signed 表示该变量是带符号位的，而 unsigned 表示该变量是不带符号位的。带符号位的变量可以表示负数，而不带符号位的变量只能表示正数，它的存储空间也就相应扩大一倍。默认所有的整型变量都是 signed 的，也就是带符号位的。



因此加上 signed 和 unsigned 限定符，四种整型就变成了八种：

- [signed] short [int]
- unsigned short [int]
- [signed] int
- unsigned int
- [signed] long [int]
- unsigned long [int]
- [signed] long long [int]
- unsigned long long [int]



> 相传国际象棋是古印度舍罕王的宰相达依尔发明的。舍罕王十分喜爱象棋，决定让宰相自己选择何种赏赐。这位聪明的宰相指着 8 * 8 共 64 格的象棋说：“陛下，请您赏给我一些麦子吧。就在棋盘的第 1 格放 1 粒，第 2 格放 2 粒，第三格放 4 粒，以后每一格都比前一格增加一倍，依此放完棋盘 64 格，我就感激不尽了。”。舍罕王听了达依尔这个“小小”的要求，想都没想就满口答应下来。
>
> 结果在给达依尔麦子时舍罕惊奇地发现要给的麦子比自己想象的要多得多，于是他进行了计算，结果令他大惊失色。请问，舍罕王要兑现他的许诺共要多少粒麦子赏赐他的宰相？如果每25000粒麦子重1kg，那么舍罕王应该给予达依尔多少公斤麦子？

```c
#include <stdio.h>
#include <math.h>

int main()
{
    unsigned long long sum = 0;
    unsigned long long temp;
    int i;
    for (i = 0; i < 64; i++) {
        // pow为C中求次方的函数，需引入math
        temp = pow(2, i);
        sum = sum + temp;
        // sum = sum + (unsigned long long)pow(2, i);
    }

    unsigned long long int weight = sum / 25000;

    printf("sum为：%llu\n", sum);
    printf("结果为：%llu\n", weight);

    return 0;
}
```



# 六、取值范围

## 1、比特位和字节

CPU 能读懂的最小单位（只能存放 0 和 1）—— 比特位，bit，b

内存机构的最小寻址单位 —— 字节，Byte，B

关系：1Byte == 8bit

因此，一个字节可以表示最大的数是：11111111



## 2、二进制、十进制和十六进制

![image-20220330103818291](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330103818291.png)

规律：

比如 11（2）== 2^2 - 1（10）；111（2）== 2^3 - 1（10）；

​		1111（2）== 2^4 - 1（10）；1111 1111（2）== 2^8 - 1（10）

**如果一个二进制所有位数都是1，转为十进制的话就是2的n次方减1，n为位数**



## 3、符号位

存放 signed 类型的存储单元中，左边第一位表示符号位。

如果该位为 0，表示该整数是一个正数；如果该位为 1，表示该整数是一个负数。

一个 32 位的整型变量，除去左边第一位符号位，剩下表示值的只有 31 个比特位。



## 4、补码

计算机是用补码的形式来存放整数的值。

正数的补码是该数的二进制形式。

负数的补码需要通过以下几步获得：

- 先取得该数的绝对值的二进制形式
- 再将第1步的值按位取反
- 最后将第2步的值加1

![image-20220330134113350](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330134113350.png)



## 5、二进制表示最大值和最小值

![image-20220330134603471](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330134603471.png)



## 6、基本数据类型的取值范围

![image-20220330144238295](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330144238295.png)

![image-20220330144841281](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220330144841281.png)

补码转十进制：-2^7+.....，即后面的正数加上-128，结果就是负的十进制。



# 七、字符和字符串

## 1、字符类型

字符类型事实上是一个特殊的整数类型，因此它也有取值范围。

signed char 的取值范围是 -128 ~ 127；unsigned char 的取值范围是 0 ~ 255。

C 标准规定普通整数类型默认使用 signed 修饰符，但没有规定 char 的默认修饰符。

因此，使用 signed 或 unsigned 修饰符，是由编译系统自行决定。



## 2、ASCII表

**存放在字符类型中的变量，都可以被解释为 ASCII 字符表中的对应字符**

标准 ASCII 字符表使用7位二进制数来表示所有的大写和小写字母，数字 0 到 9、标点符号， 以及在美式英语中使用的特殊控制字符。

其中，ASCII 字符表上的数字 0 ~ 31 以及 127（共 33 个）分配给了控制字符，用于控制像打印机等一些外围设备。这些是看不到的。数字 32 ~ 126 分配给了能在键盘上找到的字符，这些是所见即所得的。



```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char a = 70, b = 105, c = 115, d = 104, e = 67;
    printf("%c%c%c%c%c\n", a, b ,c, d, e); // FishC
    // 因为声明的是字符变量，赋值是ASCII表对应的数字
    // 输出的时候指定%c，就会找ASCII表对应的字符
    return 0;
}
```





## 3、字符串

C 语言没有专门为存储字符串设计一个单独的类型，因为没必要。我们之前已经说过，字符串事实上就是一串字符。所以只需要在内存中找一块空间，然后存放一串字符类型的变量即可。

声明字符串的语法：

`char 变量名[数量];`

对其进行赋值，事实上就是对这一块空间里边的每一个字符变量进行赋值。我们通过索引号来获得每个字符变量的空间。

`变量名[索引号] = 字符;`

```c
char name[6];

name[0] = 'F';
name[1] = 'i';
name[2] = 's';
name[3] = 'h';
name[4] = 'C',
name[5] = '\0';
```

当然，我们可以把声明和定义写在一块，语法是这样的：

```c
char name[6] = {'F', 'i', 's', 'h', 'C', '\0'};
```

其实，中括号（[]）里边的数量咱可以不写，编译器会自动帮你计算的。

```c
char a[] = {'F', 'i', 's', 'h', 'C', '\0'};
```

事实上可以直接在大括号写上字符串常量，字符串常量用双引号括起来：

```c
char a[] = {"FishC"};
```

使用字符串常量有个好处，那就是你不必亲自在末尾添加 '\0'，它会自动帮你加上。

最后，如果使用字符串常量的话，这个大括号也是可以省掉的：

```c
char a[] = "FishC";
printf("%s\n", a);
```



