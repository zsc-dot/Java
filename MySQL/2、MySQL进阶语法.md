# 一、函数

**函数**是指一段可以直接被另一段程序调用的程序或代码。也就意味着，这一段程序或代码在MySQL中已经给我们提供了，我们要做的就是在合适的业务场景调用对应的函数完成对应的业务需求即可。那么，函数到底在哪儿使用呢？

我们先来看两个场景：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220928134809398.png" alt="image-20220928134809398"  />

1.  在企业的OA或其他的人力系统中，经常会提供的有这样一个功能，每一个员工登录上来之后都能够看到当前员工入职的天数。而在数据库中，存储的都是入职日期，如 2000-11-12，那如果快速计算出天数呢？
2. 在做报表这类的业务需求中，我们要展示出学员的分数等级分布。而在数据库中，存储的是学生的分数值，如98、75，如何快速判定分数的等级呢？

其实，上述的这一类的需求，我们通过MySQL中的函数都可以很方便的实现。



MySQL中的函数主要分为以下四类： 字符串函数、数值函数、日期函数、流程函数。



## 1.1、字符串函数

MySQL中内置了很多字符串函数，常用的几个如下：

| 函数                       | 功能                                                         |
| -------------------------- | ------------------------------------------------------------ |
| concat(S1, S2, ...Sn)      | 字符串拼接，将 S1，S2，... Sn 拼接成一个字符串               |
| lower(str)                 | 将字符串 str 全部转为小写                                    |
| upper(str)                 | 将字符串 str 全部转为大写                                    |
| lpad(str, n, pad)          | 左填充，用字符串 pad 对 str 的左边进行填充，直到为 n 个字符串长度 |
| rpad(str, n, pad)          | 右填充，用字符串 pad 对 str 的右边进行填充，直到为 n 个字符串长度 |
| trim(str)                  | 去掉字符串头部和尾部的空格                                   |
| substring(str, start, len) | 返回从字符串 str 从 start 位置起的 len 个长度的字符串        |



**案例**：

1. concat：字符串拼接

   ```sql
   select concat('Hello', 'MySQL');
   ```

2.  lower：全部转小写

   ```sql
   select lower('Hello');
   ```

3. upper：全部转大写

   ```sql
   select upper('Hello');
   ```

4. lpad：左填充

   ```sql
   select lpad('01', 5, '-');
   ```

5. rpad：右填充

   ```sql
   select lpad('01', 5, '-');
   ```

6. trim：去除空格

   ```sql
   select trim(' Hello MySQL ');
   ```

7. substring：截取子字符串

   ```sql
   select substring('Hello World', 1, 5); -- 结果为Hello，MySQL中第一位字符串索引为1
   ```



**案例**：

由于业务需求变更，企业员工的工号，统一为5位数，目前不足5位数的全部在前面补0。比如：1号员工的工号应该为00001。

```sql
update emp set workno = lpad(workno, 5, '0');
```



## 1.2、数值函数

常见的数值函数如下：

| 函数        | 功能                                   |
| ----------- | -------------------------------------- |
| ceil(x)     | 向上取整                               |
| floor(x)    | 向下取整                               |
| mod(x, y)   | 返回 x/y 的模                          |
| rand()      | 返回 0~1 内的随机数                    |
| round(x, y) | 求参数 x 的四舍五入的值，保留 y 位小数 |



**案例**：

1. ceil：向上取整

   ```sql
   select ceil(1.1);
   ```

2. floor：向下取整

   ```sql
   select floor(1.9);
   ```

3. mod：取模

   ```sql
   select mod(7, 4);
   ```

4. rand：获取随机数

   ```sql
   select rand();
   ```

5. round：四舍五入

   ```sql
   select round(2.344, 2);
   ```



**案例**：

通过数据库的函数，生成一个六位数的随机验证码。

思路：获取随机数可以通过rand()函数，但是获取出来的随机数是在0-1之间的，所以可以在其基础上乘以1000000，然后舍弃小数部分，如果长度不足6位，补0

```sql
select lpad(round(rand() * 1000000, 0), 6, '0');
```



## 1.3、日期函数

常见的日期函数如下：

| 函数                               | 功能                                              |
| ---------------------------------- | ------------------------------------------------- |
| curdate()                          | 返回当前日期                                      |
| curtime()                          | 返回当前时间                                      |
| now()                              | 返回当前日期和时间                                |
| year(date)                         | 获取指定date的年份                                |
| month(date)                        | 获取指定date的月份                                |
| day(date)                          | 获取指定date的日期                                |
| DATE_ADD(date, interval expr type) | 返回一个日期/时间值加上一个时间间隔expr后的时间值 |
| DATEDIFF(date1, date2)             | 返回起始时间date1 和 结束时间date2之间的天数      |



**案例**：

1.  curdate：当前日期

   ```sql
   select curdate();
   ```

2. curtime：当前时间

   ```sql
   select curtime();
   ```

3.  now：当前日期和时间

   ```sql
   select now();
   ```

4. YEAR，MONTH，DAY：当前年、月、日

   ```sql
   select year(now());
   select month(now());
   select day(now());
   ```

5.  date_add：增加指定的时间间隔

   ```sql
   select date_add(now(), interval 70 YEAR);
   ```

6.  datediff：获取两个日期相差的天数

   ```sql
   select datediff('2021-10-01', '2021-12-01');  -- -61
   ```



**案例**：

查询所有员工的入职天数，并根据入职天数倒序排序。

思路：入职天数，就是通过当前日期 - 入职日期，所以需要使用datediff函数来完成。

```sql
select name, datediff(now(), entrydate) entrydays from emp order by entrydays desc;
```



## 3.4、流程函数

流程函数也是很常用的一类函数，可以在SQL语句中实现条件筛选，从而提高语句的效率。

| 函数                                                         | 功能                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| if(value, t, f)                                              | 如果 value 为 true ，则返回 t ，否则返回 f                   |
| ifnull(value1, value2)                                       | 如果 value1 不为空，返回 value1 ，否则返回 value2            |
| case when [val1] then [res1] ... else [default] end          | 如果 val1 为 true ，返回 res1 ，... 否则返回 default 默认值  |
| case [expr] when [ val1 ] then [res1] ...  else [default] end | 如果expr的值等于val1，返回res1 ，... 否则返回 default 默认值 |



**案例**：

1. if

   ```sql
   select if(false, 'OK', 'Error');
   ```

2. ifnull

   ```sql
   select ifnull('Ok', 'Default');
   select ifnull('', 'Default');
   select ifnull(null, 'Default');
   ```

3. case when then else end

   需求：查询emp表的员工姓名和工作地址 (北京/上海 ----> 一线城市，其他 ----> 二线城市)

   ```sql
   select
   	name,
   	(case workaddress when '北京' then '一线城市' when'上海' then '一线城市' else '二线城市' end) '工作地址'
   from emp;
   ```



**案例**：

统计班级各个学员的成绩，展示的规则如下：

- \>=85，展示优秀
- \>=60，展示及格
- 否则，展示不及格

```sql
create table score(
    id int comment 'ID',
    name varchar(20) comment '姓名',
    math int comment '数学',
	english int comment '英语',
	chinese int comment '语文'
) comment '学员成绩表';

insert into score (id, name, math, english, chinese) values
	(1, 'Tom', 67, 88, 95),
	(2, 'Rose', 23, 66, 90),
	(3, 'Jack', 56, 98, 76);
```



```sql
select 
	id, 
	name, 
	(case when math >= 85 then '优秀' when math >= 60 then '及格' else '不及格' end) '数学',
	(case when english >= 85 then '优秀' when english >= 60 then '及格' else '不及格' end) '英语',
	(case when chinese >= 85 then '优秀' when chinese >= 60 then '及格' else '不及格' end) '语文'
from score;
```



## 3.5、总结

1. 字符串函数

   ```sql
   concat、lower、upper、lpad、rpad、trim、substring
   ```

2. 数值函数

   ```sql
   ceil、floor、mod、rand、round
   ```

3. 日期函数

   ```sql
   curdate、curtime、now、year、month、day、date_add、datediff
   ```

4. 流程函数

   ```sql
   if、ifnull、case [...] when ... then ... else ... end
   ```



MySQL的常见函数我们学习完了，那接下来，我们就来分析一下，在前面讲到的两个函数的案例场景，思考一下需要用到什么样的函数来实现？

1. 数据库中，存储的是入职日期，如 2000-01-01，如何快速计算出入职天数呢？

   答案：datediff

2. 数据库中，存储的是学生的分数值，如98、75，如何快速判定分数的等级呢？

   答案：case ... when ...



# 2、约束



## 2.1、概述

概念：约束是作用于表中字段上的规则，用于限制存储在表中的数据。

目的：保证数据库中数据的正确、有效性和完整性。

分类：

| 约束                      | 描述                                                     | 关键字          |
| ------------------------- | -------------------------------------------------------- | --------------- |
| 非空约束                  | 限制该字段的数据不能为null                               | not null        |
| 唯一约束                  | 保证该字段的所有数据都是唯一、不重复的                   | unique          |
| 主键约束                  | 主键是一行数据的唯一标识，要求非空且唯一                 | primary key     |
| 默认约束                  | 保存数据时，如果未指定该字段的值，则采用默认值           | default         |
| 检查约束 (8.0.16版本之后) | 保证字段值满足某一个条件                                 | check           |
| 外键约束                  | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | **foreign key** |

**注意**：约束是作用于表中字段上的，可以在创建/修改表的时候添加约束。



## 2.2、约束演示

上面我们介绍了数据库中常见的约束，以及约束涉及到的关键字，那这些约束我们到底如何在创建表、修改表的时候来指定呢，接下来我们就通过一个案例，来演示一下。



**案例**：

根据如下需求，完成表结构的创建：

| 字段名 | 字段含义   | 字段类型    | 约束条件                  | 约束关键字                  |
| ------ | ---------- | ----------- | ------------------------- | --------------------------- |
| id     | ID唯一标识 | int         | 主键，并且自动增长        | primary key，auto_increment |
| name   | 姓名       | varchar(10) | 不为空，并且唯一          | not null，unique            |
| age    | 年龄       | int         | 大于0，并且小于等于120    | check                       |
| status | 状态       | char(1)     | 如果没有指定该值，默认为1 | default                     |
| gender | 性别       | char(1)     | 无                        |                             |



对应的建表语句为：

```sql
create table tb_user(
    id int auto_increment primary key comment 'ID唯一标识',
    name varchar(10) not null unique comment '姓名',
    age int comment '年龄',
    status char(1) default 1 comment '状态',
    gender char(1) comment '性别'
);
```

在为字段添加约束时，我们只需要在字段之后加上约束的关键字即可，需要关注其语法。



我们执行上面的SQL把表结构创建完成，然后接下来，就可以通过一组数据进行测试，从而验证一下，约束是否可以生效。

```sql
insert into tb_user(name,age,status,gender) values ('Tom1',19,'1','男'), ('Tom2',25,'0','男');
insert into tb_user(name,age,status,gender) values ('Tom3',19,'1','男');
insert into tb_user(name,age,status,gender) values (null,19,'1','男');
insert into tb_user(name,age,status,gender) values ('Tom3',19,'1','男');
insert into tb_user(name,age,status,gender) values ('Tom4',80,'1','男');
insert into tb_user(name,age,status,gender) values ('Tom5',-1,'1','男');
insert into tb_user(name,age,status,gender) values ('Tom5',121,'1','男');
insert into tb_user(name,age,gender) values ('Tom5',120,'男');
```



## 2.3、外键约束



### 2.3.1、介绍

外键：用来让两张表的数据之间建立连接，从而保证数据的一致性和完整性。

我们来看一个例子：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220928171312144.png" alt="image-20220928171312144"  />



左侧的emp表是员工表，里面存储员工的基本信息，包含员工的ID、姓名、年龄、职位、薪资、入职日期、上级主管ID、部门ID，这个部门的ID是关联的部门表dept的主键id，那emp表的dept_id就是外键，关联的是另一张表的主键。



**注意**：目前上述两张表，只是在逻辑上存在这样一层关系；在数据库层面，并未建立外键关联，所以是无法保证数据的一致性和完整性的。

没有数据库外键关联的情况下，能够保证一致性和完整性呢？我们来测试一下。