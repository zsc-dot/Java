# 一、视图/存储过程/触发器



## 1.1、视图



### 1.1.1、介绍

视图（View）是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图只保存了查询的SQL逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。



### 1.1.2、语法

1. 创建

   ```sql
   create [or replace] view 视图名称[(列名列表)] as select语句 [with [cascaded | local] check option];
   ```

   or replace：如果要替换视图，可以加上。

2. 查询

   ```sql
   查看创建视图语句：SHOW CREATE VIEW 视图名称;
   查看视图数据：SELECT * FROM 视图名称 ... ;
   ```

3. 修改

   ```sql
   方式一：create [or replace] view 视图名称[(列名列表)] as select语句 [with [cascaded | local] check option];
   方式二：alter view 视图名称[(列名列表)] as select语句 [with [cascaded | local] check option];
   ```

4. 删除

   ```sql
   drop vie [if exists] 视图名称 [, 视图名称...];
   ```



**示例**：

```sql
-- 创建视图
create or replace view stu_v1 as select id, name from student where id <= 10;

-- 查询视图
show create view stu_v1;
select * from stu_v1;

-- 修改视图
create or replace view stu_v1 as select id, name, no from student where  id <= 10;
alter view stu_v1 as select id, name from student where  id <= 10;

-- 删除视图
drop view if exists stu_v1;
```



上述我们演示了，视图应该如何创建、查询、修改、删除，那么我们能不能通过视图来插入、更新数据呢？ 接下来，做一个测试。

```sql
create or replace view stu_v_1 as select id, name from student where id <= 10;
select * from stu_v_1;
insert into stu_v_1 values(6, 'Tom');
insert into stu_v_1 values(17, 'Tom22');
```

执行上述的SQL会发现，id为6和17的数据都是可以成功插入的。在执行查询视图，可以看到第一条数据已经插入了，其实数据是在基表student中存储了。

但是查询出来的数据，却没有id为17的记录。

因为在创建视图的时候，指定的条件为 id<=10，id为17的数据，是不符合条件的，所以没有查询出来，但是这条数据确实是已经成功的插入到了基表中。



如果定义视图时指定了条件，然后在插入、修改、删除数据时，是否可以做到必须满足条件才能操作，否则不能够操作呢？答案是可以的，这就需要借助于视图的检查选项了。



### 1.1.3、检查选项

当使用with check option子句创建视图时，MySQL会通过视图检查正在更改的每个行，例如插入，更新，删除，以使其符合视图的定义。

MySQL允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。

为了确定检查的范围，mysql提供了两个选项：cascaded 和 local，默认值为 cascaded。



#### 1、CASCADED

**级联**：比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为 cascaded，但是v1视图创建时未指定检查选项。 则在执行检查时，不仅会检查v2，还会级联检查v2的关联视图v1。

```sql
-- stu_v_1没有检查选项，这两条插入都会成功
create or replace view stu_v_1 as select id, name from student where id <= 20;
insert into stu_v_1 values(5, 'Tom');
insert into stu_v_1 values(25, 'Tom');

-- stu_v_2添加了检查条件
create or replace view stu_v_2 as select id, name from stu_v_1 where id >= 10 with cascaded check option;

insert into stu_v_2 values(7, 'Tom');   -- 添加失败，因为不满足stu_v_2的 id >= 10 条件
insert into stu_v_2 values(26, 'Tom');  -- 添加失败，因为不满足stu_v_1的 id <= 20 条件
insert into stu_v_2 values(15, 'Tom');  -- 添加成功，满足 id >= 10 和 id <= 20的条件

create or replace view stu_v_3 as select id, name from stu_v_2 where id <= 15;
insert into stu_v_3 values(11, 'Tom');  -- 添加成功，stu_v_3的条件不做检查，同时满足 id >= 10 和 id <= 20的条件
insert into stu_v_3 values(17, 'Tom');  -- 添加成功，stu_v_3的条件不做检查，同时满足 id >= 10 和 id <= 20的条件
insert into stu_v_3 values(28, 'Tom');  -- 添加失败，stu_v_3的条件不做检查，不满足id >= 10 和 id <= 20的条件
```



<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221008140744499.png" alt="image-20221008140744499"  />

如图：

- 视图v1创建时，并没有指定with check option，此时对视图进行增删改的操作，都不会检查规则
- 视图v2创建时，是基于视图v1创建的，并且指定了with cascaded check option，相当于在v1上也加了一个检查
- 视图v3创建时，是基于视图v2创建的，没有指定检查选项，就只会检查v2和v1的条件



#### 2、 LOCAL

本地：比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为 local，但是v1视图创建时未指定检查选项。 则在执行检查时，只会检查v2，不会检查v2的关联视图v1。

此项为MySQL8.0特性。

```sql
-- stu_v_1没有检查选项，这两条插入都会成功
create or replace view stu_v_4 as select id, name from student where id <= 15;
insert into stu_v_4 values(5, 'Tom');
insert into stu_v_4 values(16, 'Tom');

create or replace view stu_v_5 as select id, name from stu_v_4 where id >= 10 with local check option;
insert into stu_v_5 values(13, 'Tom');  -- 添加成功，满足stu_v_5的 id >= 10条件，同时stu_v_4并没有指定检查选项，不做检查
insert into stu_v_5 values(17, 'Tom');  -- 添加成功，满足stu_v_5的 id >= 10条件，同时stu_v_4并没有指定检查选项，不做检查
create or replace view stu_v_6 as select id, name from stu_v_5 where id < 20;
insert into stu_v_5 values(14, 'Tom'); -- 添加成功，stu_v_6没有指定检查选项，同时满足stu_v_5的id >= 10条件，stu_v_4并没有指定检查选项
```



![image-20221008142809579](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221008142809579.png)

如图：

- 视图v1创建时，并没有指定with check option，此时对视图进行增删改的操作，都不会检查规则
- 视图v2创建时，是基于视图v1创建的，并且指定了with local check option，如果视图v1有检查选项，则会检查视图v1的条件，没有则不检查
- 视图v3创建时，是基于视图v2创建的，没有指定检查选项，就只会检查视图v2的条件



### 1.1.4、视图的更新

要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则该视图不可更新：

- 聚合函数或窗口函数（SUM()、MIN()、MAX()、COUNT()等）
- DISTINCT
- GROUP BY
-  HAVING
- UNION 或者 UNION ALL



**示例**：

```sql
create view stu_v_count as select count(*) from student;
```

上述的视图中，就只有一个单行单列的数据，如果我们对这个视图进行更新或插入，将会报错：

```sql
insert into stu_v_count values(10);
```



### 1.1.5、视图作用

- 简单

  视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件

- 安全

  数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据

- 数据独立

  视图可帮助用户屏蔽真实表结构变化带来的影响



### 1.1.6、案例

1. 为了保证数据库表的安全性，开发人员在操作tb_user表时，只能看到的用户的基本字段，屏蔽手机号和邮箱两个字段。

   ```sql
   create or replace view tb_user_view as select id, name, profession, age, gender status, createtime from tb_user;
   
   select * from tb_user_view;
   ```

2. 查询每个学生所选修的课程（三张表联查），这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。

   ```sql
   create view tb_stu_course_view as select s.name student_name, s.no student_no, c.name course_name from student s, student_course sc, course c where s.id = sc.studentid and sc.courseid = c.id;
   
   select * from tb_stu_course_view;
   ```

   

## 1.2、存储过程



### 1.2.1、介绍

存储过程是事先经过编译并存储在数据库中的一段 SQL 语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程思想上很简单，就是数据库 SQL 语言层面的代码封装与重用。

**特点**：

- 封装，复用
  - 可以把某一业务SQL封装在存储过程中，需要用到的时候直接调用即可
- 可以接收参数，也可以返回数据
  - 在存储过程中，可以传递参数，也可以接收返回值
- 减少网络交互，效率提升
  - 如果涉及到多条SQL，每执行一次都是一次网络传输。而如果封装在存储过程中，我们只需要网络交互一次可能就可以了。



### 1.2.2、基本语法

1. 创建

   ```sql
   create procedure 存储过程名称([参数列表])
   begin
   	-- SQL语句
   end;
   ```

2. 调用

   ```sql
   call 存储过程名称([参数]);
   ```

3. 查看

   ```sql
   select * from INFORMATION_SCHEMA.ROUTINES where ROUTINE_SCHEMA = 'xxx';  -- 查询指定数据库的存储过程及状态信息
   show create procedure 存储过程名称;  -- 查询某个存储过程的定义
   ```

4. 删除

   ```sql
   drop procedure [if exists] 存储过程名称;
   ```



**注意**：在命令行中，执行创建存储过程的SQL时，需要通过关键字 delimiter 指定SQL语句的结束符。



**示例**：

```sql
-- 创建
create procedure p1()
begin
	select count(*) from student;
end;

-- 调用
call p1();

-- 查看
select * from information_schema.ROUTINES where ROUTINE_SCHEMA = 'itcast';
show create procedure p1;

-- 删除
drop procedure if exists p1;


-- 命令行创建
delimiter $$ -- 指定$$取代;为结尾
create procedure p1()
begin
	select count(*) from student;
end$$
delimiter ;
```



### 1.2.3、变量

在MySQL中变量分为三种类型：系统变量、用户定义变量、局部变量。



#### 1、系统变量

**系统变量**是MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局变量（GLOBAL）、会话变量（SESSION）。

全局变量(GLOBAL)：全局变量针对于所有的会话。

会话变量(SESSION)：会话变量针对于单个会话，在另外一个会话窗口就不生效了。

1. 查看系统变量

   ```sql
   show [session | global] variables;  -- 查看所有系统变量
   show [session | global] variables like '...';  -- 可以通过like模糊匹配方式查找变量
   select @@[session | global] 系统变量名;  -- 查看指定变量的值
   ```

2. 设置系统变量

   ```sql
   set [session | global] 系统变量名 = 值;
   set @@[session | global] 系统变量名 = 值;
   ```

**注意**：

- 如果没有指定 SESSION/GLOBAL ，默认是 SESSION 会话变量
- mysql服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在 /etc/my.cnf 中配置



**示例**：

```sql
-- 查看系统变量
show session variables;
show session variables like 'auto%';
show global variables like 'auto%';
select @@session.autocommit;
select @@global.autocommit;

-- 设置系统变量
set session autocommit = 0;
insert into course(id, name) VALUES (6, 'ES');
commit;
```



#### 2、用户自定义变量

**用户定义变量**是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用 "@变量名" 使用就可以。其作用域为当前连接。

1. 赋值

   方式一：

   ```sql
   set @var_name = expr [, @var_name = expr] ...;
   set @var_name := expr [, @var_name := expr] ...;
   ```

   赋值时，可以使用=，也可以使用:=

   方式二：

   ```sql
   select @var_name := expr [, @var_name := expr] ...;
   select 字段名 into @var_name from 表名;
   ```

2. 使用

   ```sql
   select @var_name;
   ```



**注意**：

- 用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。



**示例**：

```sql
-- set 赋值
set @myname = 'itcast';
set @myage := 10;
set @mygender := '男', @myhobby := 'java';


-- 使用
select @myname, @myage, @mygender, @myhobby;


-- select 赋值
select @mycolor := 'red';
select count(*) into @mycount from tb_user;

select @mycolor, @mycount;

select @abc; -- 没有定义，获取到的值为null
```



#### 3、局部变量

**局部变量**是根据需要定义的在局部生效的变量，访问之前，需要DECLARE声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的BEGIN ... END块。



1. 声明

   ```sql
   DECLARE 变量名 变量类型 [DEFAULT ... ];
   ```

   变量类型就是数据库字段类型：INT、BIGINT、CHAR、VARCHAR、DATE、TIME等。

2. 赋值

   ```sql
   SET 变量名 = 值 ;
   SET 变量名 := 值 ;
   SELECT 字段名 INTO 变量名 FROM 表名 ... ;
   ```



**示例**：

```sql
-- 声明、赋值
create procedure p2()
begin
	declare stu_count int default 0;
	-- set stu_count := 100;
	select count(*) into stu_count from student;
	select stu_count;
end;

call p2();
```



### 1.2.4、if

if 用于做条件判断，具体的语法结构为：

```sql
if 条件1 then
	......
elseif 条件2 then  -- 可选
	......
else  -- 可选
	......
end if;
```

在if条件判断的结构中，ELSE IF 结构可以有多个，也可以没有。ELSE结构可以有，也可以没有。



**案例**：

根据定义的分数score变量，判定当前分数对应的分数等级。

- score >= 85分，等级为优秀。
- score >= 60分 且 score < 85分，等级为及格。
- score < 60分，等级为不及格。

```sql
create procedure p3()
begin
	declare score int default 58;
	declare result varchar(10);
	
	if score >= 85 then
		set result := '优秀';
	elseif score >= 60 then
		set result := '及格';
	else
		set result := '不及格';
	end if;
	select result;
end;

call p3();
```



上述的需求我们虽然已经实现了，但是也存在一些问题，比如：score分数我们是在存储过程中定义死的，而且最终计算出来的分数等级，我们也仅仅是最终查询展示出来而已。

那么我们能不能，把score分数动态的传递进来，计算出来的分数等级是否可以作为返回值返回呢？

答案是肯定的，我们可以通过接下来所讲解的**参数**来解决上述的问题。



### 1.2.5、参数

参数的类型，主要分为以下三种：IN、OUT、INOUT。 具体的含义如下：

| 类型  | 含义                                         | 备注 |
| ----- | -------------------------------------------- | ---- |
| in    | 该类参数作为输入，也就是需要调用时传入值     | 默认 |
| out   | 该类参数作为输出，也就是该参数可以作为返回值 |      |
| inout | 既可以作为输入参数，也可以作为输出参数       |      |



用法为：

```sql
create procedure 存储过程名称([in/out/inout 参数名 参数类型])
begin
	-- SQL语句
end;
```



**案例一**：

根据传入参数score，判定当前分数对应的分数等级，并返回。

- score >= 85分，等级为优秀。
- score >= 60分 且 score < 85分，等级为及格。
- score < 60分，等级为不及格。

```sql
create procedure p4(in score int, out result varchar(10))
begin
	if score >= 85 then
		set result := '优秀';
	elseif score >= 60 then
		set result := '及格';
	else
		set result := '不及格';
	end if;
end;

-- 定义用户变量 @result来接收返回的数据, 用户变量可以不用声明
call p4(68, @result);

select @result;
```



**案例二**：

将传入的200分制的分数，进行换算，换算成百分制，然后返回。

```sql
create procedure p5(inout score double)
begin
	set score := score * 0.5;
end;

set @score := 78;
call p5(@score);
select @score;
```



### 1.2.6、case

case结构及作用，和我们在基础篇中所讲解的流程控制函数很类似。有两种语法格式：

语法1：

```sql
-- 含义： 当case_value的值为 when_value1时，执行statement_list1，当值为 when_value2时，执行statement_list2，否则就执行 statement_list
case case_value
	when when_value1 then statement_list1
	[ WHEN when_value2 THEN statement_list2]
	......
	[else statement_list]
end case;
```

语法2：

```sql
-- 含义：当条件search_condition1成立时，执行statement_list1，当条件search_condition2成立时，执行statement_list2，否则就执行 statement_list
case
	when search_condition1 then statement_list1
	[WHEN search_condition2 THEN statement_list2]
	......
	[else statement_list]
end case;
```



**案例**：

根据传入的月份，判定月份所属的季节（要求采用case结构）。

- 1-3月份，为第一季度
- 4-6月份，为第二季度
- 7-9月份，为第三季度
- 10-12月份，为第四季度

```sql
create procedure p6(in month int)
begin
	declare result varchar(10);
	case 
		when month >= 1 and month <= 3 then
			set result := '第一季度';
		when month >= 4 and month <= 6 then
			set result := '第二季度';
		when month >= 7 and month <= 9 then
			set result := '第三季度';
		when month >= 10 and month <= 12 then
			set result := '第四季度';
		else
			set result := '非法参数';
	end case;
	select concat('输入的月份为：', month, '所属的季度为：', result);
end;

call p6(4);
```



### 1.2.7、while

while 循环是有条件的循环控制语句。满足条件后，再执行循环体中的SQL语句。具体语法为：

```sql
-- 先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
while 条件 do
	SQL逻辑
end while;
```



**案例**：

计算从1累加到n的值，n为传入的参数值。

```sql
create procedure p7(in n int)
begin
	declare total int default 0;
	while n > 0 do
		set total := total + n;
		set n := n -1;
	end while;
	select total;
end;

call p7(100);
```



### 1.2.8、repeat

repeat是有条件的循环控制语句，当满足until声明的条件的时候，则退出循环。具体语法为：

```sql
-- 先执行一次逻辑，然后判定until条件是否满足，如果满足，则退出。如果不满足，则继续下一次循环
repeat
	SQL逻辑
	until 条件
end repeat;
```



**案例**：

计算从1累加到n的值，n为传入的参数值。(使用repeat实现)

```sql
create procedure p8(in n int)
begin
	declare total int default 0;
	repeat
		set total := total + n;
		set n := n-1;
	until n <= 0
	end repeat;
	select total;
end;

call p8(100);
```



### 1.2.9、loop

LOOP 实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。

LOOP可以配合一下两个语句使用：

- LEAVE：配合循环使用，退出循环。
- ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

```sql
[begin_label:] loop
	SQL逻辑
end loop [end_label];

leave label; -- 退出指定标记的循环体
iterate label; -- 直接进入下一次循环
```

上述语法中出现的 begin_label，end_label，label 指的都是我们所自定义的标记。



**案例一**：

计算从1累加到n的值，n为传入的参数值。

```sql
create procedure p9(in n int)
begin
	declare total int default 0;
	sum:loop
		if n <= 0 then
			leave sum;
		end if;
		set total := total + n;
		set n := n - 1;
	end loop sum;
	select total;
end;

call p9(10);
```



**案例二**：

计算从1到n之间的偶数累加的值，n为传入的参数值。

```sql
create procedure p10(in n int)
begin
	declare total int default 0;
	sum:loop
		if n <= 0 then
			leave sum;
		end if;
		if n%2 != 0 then
			set n := n - 1;
			iterate sum;
		end if;
		set total := total + n;
		set n := n - 1;
	end loop sum;
	select total;
end;

call p10(10);
```



### 1.2.10、游标

游标（CURSOR）是用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下：

```sql
-- 声明游标
declare 游标名称 cursor for 查询语句;
-- 打开游标
open 游标名称;
-- 获取游标记录
fetch 游标名称 into 变量[, 变量];
-- 关闭游标
close 游标名称;
```



**案例**：

根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名（name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表 (id, name, profession) 中。

```sql
-- 逻辑:
-- A. 声明游标, 存储查询结果集
-- B. 准备: 创建表结构
-- C. 开启游标
-- D. 获取游标中的记录
-- E. 插入数据到新表中
-- F. 关闭游标
create procedure p11(in uage int)
begin
	-- 定义变量要在定义游标之前
	declare uname varchar(100);
	declare upro varchar(100);
	declare u_cursor cursor for select name, profession from tb_user where age <= uage;
	
	drop table if exists tb_user_pro;
	create table if not exists tb_user_pro(
		id int primary key auto_increment,
		name varchar(100),
		profession varchar(100)
	);
	
	open u_cursor;
	while true do
		fetch u_cursor into uname, upro;
		insert into tb_user_pro values (null, uname, upro);
	end while;
	close u_cursor;
end;

call p11(40);
```



上述存储过程在调用的过程中会报错，因为上面的while循环中并没有退出条件。当游标的数据集获取完毕之后，再次获取数据就会报错，从而终止了程序的执行。

但是此时，tb_user_pro表结构及其数据都已经插入成功了，我们可以直接刷新表结构，检查表结构中的数据。

功能虽然实现了，但是逻辑并不完善，而且程序执行完毕，获取不到数据，数据库还报错。接下来，我们就需要来完成这个存储过程，并且解决这个问题。

要想解决这个问题，就需要通过MySQL中提供的**条件处理程序 Handler**来解决。



### 1.2.11、条件处理程序

条件处理程序（Handler）可以用来定义在流程控制结构执行过程中遇到问题时相应的处理步骤。具体语法为：

```sql
declare handler_action handler for condition_value [, condition_value]... statement;

handler_action的取值：
CONTINUE: 继续执行当前程序
EXIT: 终止执行当前程序

condition_value 的取值：
SQLSTATE sqlstate_value: 状态码，如 02000
SQLWARNING: 所有以01开头的SQLSTATE代码的简写
NOT FOUND: 所有以02开头的SQLSTATE代码的简写
SQLEXCEPTION: 所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
```



**案例**：

我们继续来完成在上一小节提出的需求，并解决其中的问题。

根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名（name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表(id, name, profession)中。

1.  通过SQLSTATE指定具体的状态码

   ```sql
   create procedure p11(in uage int)
   begin
   	declare uname varchar(100);
   	declare upro varchar(100);
   	declare u_cursor cursor for select name, profession from tb_user where age <= uage;
   	-- 声明条件处理程序：当SQL语句执行抛出的状态码为02000时，将关闭游标u_cursor，并退出
   	declare exit handler for SQLSTATE '02000' close u_cursor;
   	
   	drop table if exists tb_user_pro;
   	create table if not exists tb_user_pro(
   		id int primary key auto_increment,
   		name varchar(100),
   		profession varchar(100)
   	);
   	
   	open u_cursor;
   	while true do
   		fetch u_cursor into uname, upro;
   		insert into tb_user_pro values (null, uname, upro);
   	end while;
   	close u_cursor;
   end;
   
   call p11(30);
   ```

2. 通过SQLSTATE的代码简写方式 NOT FOUND

   02 开头的状态码，代码简写为 NOT FOUND

   ```sql
   create procedure p11(in uage int)
   begin
   	declare uname varchar(100);
   	declare upro varchar(100);
   	declare u_cursor cursor for select name, profession from tb_user where age <= uage;
   	declare exit handler for not found close u_cursor;
   	
   	drop table if exists tb_user_pro;
   	create table if not exists tb_user_pro(
   		id int primary key auto_increment,
   		name varchar(100),
   		profession varchar(100)
   	);
   	
   	open u_cursor;
   	while true do
   		fetch u_cursor into uname, upro;
   		insert into tb_user_pro values (null, uname, upro);
   	end while;
   	close u_cursor;
   end;
   
   call p11(30);
   ```

   具体的错误状态码，可以参考官方文档：

   https://dev.mysql.com/doc/refman/8.0/en/declare-handler.html 

   https://dev.mysql.com/doc/mysql-errors/8.0/en/server-error-reference.html



## 1.3、存储函数

存储函数是有返回值的存储过程，存储函数的参数只能是IN类型的。具体语法如下：

```sql
create function 存储函数名称 ([参数列表])
returns type [characteristic ...]
begin
	SQL语句
	return ...;
end;
```

characteristic说明：

- DETERMINISTIC：相同的输入参数总是产生相同的结果
- NO SQL：不包含 SQL 语句
- READS SQL DATA：包含读取数据的语句，但不包含写入数据的语句



**案例**：

计算从1累加到n的值，n为传入的参数值。

```sql
create function fun1(n int)
returns int DETERMINISTIC
begin
	declare total int default 0;
	while n > 0 do
		set total := total + n;
		set n := n - 1;
	end while;
	return total;
end;

select fun1(100);
```



存储函数一般用的比较少，因为存储函数能做的事，存储过程也可以做



## 1.4、触发器



### 1.4.1、介绍

触发器是与表有关的数据库对象，指在insert/update/delete之前(BEFORE)或之后(AFTER)，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。
使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

| 触发器类型      | NEW 和 OLD                                             |
| --------------- | ------------------------------------------------------ |
| INSERT 型触发器 | NEW 表示将要或者已经新增的数据                         |
| UPDATE 型触发器 | OLD 表示修改之前的数据，NEW 表示将要或已经修改后的数据 |
| DELETE 型触发器 | OLD 表示将要或者已经删除的数据                         |



### 1.4.2、语法

1. 创建

   ```sql
   create trigger 触发器名称
   before/after insert/update/delete
   on 表名 for each row -- 行级触发器
   begin
   	SQL语句;
   end;
   ```

2. 查看

   ```sql
   show triggers;
   ```

3. 删除

   ```sql
   drop trigger 数据库名.触发器名称;  -- 如果没有指定数据库名，默认为当前数据库
   ```



### 1.4.3、案例

通过触发器记录 tb_user 表的数据变更日志，将变更日志插入到日志表user_logs中，包含增加，修改，删除

表结构准备：

```sql
-- 准备工作 : 日志表 user_logs
create table user_logs(
    id int(11) not null auto_increment,
    operation varchar(20) not null comment '操作类型, insert/update/delete',
    operate_time datetime not null comment '操作时间',
    operate_id int(11) not null comment '操作的ID',
    operate_params varchar(500) comment '操作参数',
    primary key(`id`)
)engine=innodb default charset=utf8;
```



1. 插入数据触发器

   ```sql
   create trigger tb_user_insert_trigger
   	after insert on tb_user for each row
   begin
   	insert into user_logs (id, operation, operate_time, operate_id, operate_params) values
   	(null, 'insert', now(), new.id, concat('插入的数据内容为：id=', new.id, 'name=', new.name, 'phone=', new.phone, 'email=',new.email, 'profession=', new.profession));
   end;
   ```

   测试：

   ```sql
   -- 查看
   show triggers;
   -- 插入数据到tb_user
   insert into tb_user(id, name, phone, email, profession, age, gender, status,createtime) VALUES (26,'三子','18809091212','erhuangzi@163.com','软件工程',23,'1','1',now());
   ```

2. 修改触发器

   ```sql
   create trigger tb_user_updaye_trigger
   	after update on tb_user for each row
   begin
   	insert into user_logs (id, operation, operate_time, operate_id, operate_params) values
   	(null, 'update', now(), new.id, concat(
   	'更新之前的数据：id=', old.id, 'name=', old.name, 'phone=', old.phone, 'email=', old.email, 'profession=', old.profession,
   	' | 更新之后的数据：id=', new.id, 'name=', new.name, 'phone=', new.phone, 'email=', new.email, 'profession=', new.profession
   	));
   end;
   ```

   测试：

   ```sql
   update tb_user set profession = '会计' where id = 23;
   update tb_user set profession = '会计' where id <= 5;
   ```

3. 删除数据触发器

   ```sql
   create trigger tb_user_delete_trigger
   	after delete on tb_user for each row
   begin
   	insert into user_logs (id, operation, operate_time, operate_id, operate_params) values
   	(null, 'delete', now(), old.id, concat('删除的数据内容为：id=', old.id, 'name=', old.name, 'phone=', old.phone, 'email=', old.email, 'profession=', old.profession));
   end;
   ```

   测试：

   ```sql
   delete from tb_user where id = 26;
   ```



## 1.5、总结

1. 视图 (VIEW)
   - 虚拟存在的表，不保存查询结果，只保存查询的SQL逻辑
   - 简单、安全、数据独立
2. 存储过程 (PROCEDURE)
   - 事先定义并存储在数据库中的一段SQL语句的集合
   - 减少网络交互，提高性能，封装重用
   - 变量、if、case、参数(in/out/inout)、while、repeat、loop、cursor、handler
3. 存储函数 (FUNCTION)
   - 存储函数是有返回值的存储过程，参数类型只能为in类型
   - 存储函数可以被存储过程替代
4. 触发器 (TRIGGER)
   - 可以在表数据进行insert、update、delete之前或之后触发
   - 保证数据完整性、日志记录、数据校验



# 二、锁



## 2.1、概述

锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。

如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

MySQL中的锁，按照锁的粒度分，分为以下三类：

- 全局锁：锁定数据库中的所有表
- 表级锁：每次操作锁住整张表
- 行级锁：每次操作锁住对应的行数据



## 2.2、全局锁



### 2.2.1、介绍

全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都将被阻塞。

其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。



为什么全库逻辑备份，就需要加全就锁呢？

1. 我们一起先来分析一下不加全局锁，可能存在的问题。

   假设在数据库中存在这样三张表：tb_stock 库存表，tb_order 订单表，tb_orderlog 订单日志表。

   ![image-20221010093520999](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010093520999.png)

   - 在进行数据备份时，先备份了tb_stock库存表
   - 然后接下来，在业务系统中，执行了下单操作，扣减库存，生成订单（更新tb_stock表，插入tb_order表）
   - 然后再执行备份 tb_order表的逻辑
   - 业务中执行插入订单日志操作
   - 最后，又备份了tb_orderlog表

   此时备份出来的数据，是存在问题的。因为备份出来的数据，tb_stock表与tb_order表的数据不一致 (有最新操作的订单信息,但是库存数没减)。

   那如何来规避这种问题呢? 此时就可以借助于MySQL的全局锁来解决。

2. 再来分析一下加了全局锁后的情况

   ![image-20221010093701668](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010093701668.png)

   对数据库进行进行逻辑备份之前，先对整个数据库加上全局锁，一旦加了全局锁之后，其他的DDL、DML全部都处于阻塞状态，但是可以执行DQL语句，也就是处于只读状态，而数据备份就是查询操作。那么数据在进行逻辑备份的过程中，数据库中的数据就是不会发生变化的，这样就保证了数据的一致性和完整性。



### 2.2.2、语法

1. 加全局锁

   ```sql
   flush tables with read lock;
   ```

2. 数据备份

   不需要在数据库命令行中执行，在终端命令行中执行即可

   ```sh
   mysqldump -uroot -p1234 itcast > itcast.sql
   ```

   数据备份的相关指令, 在后面MySQL管理章节, 还会详细讲解。

3. 释放锁

   ```sql
   unlock tables;
   ```



### 2.2.3、特点

数据库中加全局锁，是一个比较重的操作，存在以下问题：

- 如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆
- 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志（binlog），会导致主从延迟。



在InnoDB引擎中，我们可以在备份时加上参数 --single-transaction 参数来完成不加锁的一致性数据备份。

```sh
mysqldump --single-transaction -uroot -p1234 itcast > itcast.sql
```



## 2.3、表级锁



### 2.3.1、介绍

表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。

对于表级锁，主要分为以下三类：

- 表锁
- 元数据锁 (meta data lock，MDL)
- 意向锁



### 2.3.2、表锁

对于表锁，分为两类：

- 表共享读锁（read lock）
- 表独占写锁（write lock）



**语法**：

- 加锁

  ```sql
  lock tables 表名1[表名2...] read/write;
  ```

- 释放锁

  ```sql
  unlock tables;
  客户端断开连接也会释放锁
  ```



**特点**：

1. 共享读锁

   ![image-20221010102003251](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010102003251.png)

   左侧为客户端一，对指定表加了读锁，不会影响右侧客户端二的读，但是会阻塞右侧客户端的写。

2. 独占写锁

   ![image-20221010102035725](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010102035725.png)

   左侧为客户端一，对指定表加了写锁，会阻塞右侧客户端的读和写。



> 结论：读锁不会阻塞其他客户端的读，但是会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。



### 2.3.3、元数据锁

meta data lock，元数据锁，简写MDL。

MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。

MDL锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。**为了避免DML与DDL冲突，保证读写的正确性。**

这里的元数据，大家可以简单理解为就是一张表的表结构。也就是说，某一张表涉及到未提交的事务时，是不能够修改这张表的表结构的。

在MySQL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁(共享)；当对表结构进行变更操作的时候，加MDL写锁(排他)。

常见的SQL操作时，所添加的元数据锁：

| 对应SQL                                        | 锁类型                                  | 说明                                             |
| ---------------------------------------------- | --------------------------------------- | ------------------------------------------------ |
| lock tables xxx read / write                   | SHARED_READ_ONLY / SHARED_NO_READ_WRITE |                                                  |
| select 、select ... lock in share mode         | SHARED_READ (MDL共享锁)                 | 与SHARED_READ、SHARED_WRITE兼容，与EXCLUSIVE互斥 |
| insert 、update、delete、select ... for update | SHARED_WRITE (MDL共享锁)                | 与SHARED_READ、SHARED_WRITE兼容，与EXCLUSIVE互斥 |
| alter table ...                                | EXCLUSIVE (排他锁)                      | 与其他的MDL都互斥                                |



**演示**：

当执行select、insert、update、delete等语句时，添加的是元数据共享锁（SHARED_READ / SHARED_WRITE），之间是兼容的。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010105044413.png" alt="image-20221010105044413"  />



当执行SELECT语句时，添加的是元数据共享锁（SHARED_READ），会阻塞元数据排他锁（EXCLUSIVE），之间是互斥的。

![image-20221010105643119](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010105643119.png)



我们可以通过下面的SQL，来查看数据库中的元数据锁的情况：

```sql
select object_type, object_schema, object_name, lock_type, lock_duration from performance_schema.metadata_locks;
```



### 2.3.4、意向锁

#### 1、介绍

为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

假如没有意向锁，客户端一对表加了行锁后，客户端二如何给表加表锁呢，来通过示意图简单分析一下：

1. 首先客户端一，开启一个事务，然后执行DML操作，在执行DML语句时，会对涉及到的行加行锁

   ![image-20221010110501515](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010110501515.png)

2. 当客户端二，想对这张表加表锁时，会检查当前表是否有对应的行锁，如果没有，则添加表锁，此时就会从第一行数据，检查到最后一行数据，效率较低

   ![image-20221010110545545](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010110545545.png)



有了意向锁之后：

1. 客户端一，在执行DML操作时，会对涉及的行加行锁，同时也会对该表加上意向锁

   ![image-20221010110632671](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010110632671.png)

2. 而其他客户端，在对这张表加表锁的时候，会根据该表上所加的意向锁来判定是否可以成功加表锁，而不用逐行判断行锁情况了

   ![image-20221010110709273](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010110709273.png)

#### 2、分类

- 意向共享锁(IS)：由语句select ... lock in share mode添加 。与 表锁共享锁(read)兼容，与表锁排他锁(write)互斥。
- 意向排他锁(IX)：由insert、update、delete、select...for update添加 。与表锁共享锁(read)及排他锁(write)都互斥，意向锁之间不会互斥。



> 一旦事务提交了，意向共享锁、意向排他锁，都会自动释放。

可以通过以下SQL，查看意向锁及行锁的加锁情况：

```sql
select object_schema, object_name, index_name, lock_type, lock_mode, lock_data from performance_schema.data_locks;
```



**演示**：

1. 意向共享锁与表读锁是兼容的

   ![image-20221010112727187](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010112727187.png)

2. 意向排他锁和表读锁、写锁都互斥

   ![image-20221010112929338](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010112929338.png)



## 2.4、行级锁



### 2.4.1、介绍

行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在InnoDB存储引擎中。

InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类：

- 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete。在RC、RR隔离级别下都支持。

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010153538091.png" alt="image-20221010153538091" style="zoom:80%;" />

- 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持。

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010153625543.png" alt="image-20221010153625543" style="zoom:80%;" />

- 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持。

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010153651557.png" alt="image-20221010153651557" style="zoom:80%;" />



### 2.4.2、行锁

#### 1、介绍

InnoDB实现了以下两种类型的行锁：

- 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
- 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。



两种行锁的兼容情况如下：

| 当前锁类型 \ 请求锁类型 | S（共享锁） | X（排他锁） |
| ----------------------- | ----------- | ----------- |
| S（共享锁）             | 兼容        | 冲突        |
| X（排他锁）             | 冲突        | 冲突        |



常见的SQL语句，在执行时，所加的行锁如下：

| SQL                           | 行锁类型   | 说明                                     |
| ----------------------------- | ---------- | ---------------------------------------- |
| insert ...                    | 排他锁     | 自动加锁                                 |
| update ...                    | 排他锁     | 自动加锁                                 |
| delete ...                    | 排他锁     | 自动加锁                                 |
| select ...                    | 不加任何锁 |                                          |
| select ... lock in share mode | 共享锁     | 需要手动在SELECT之后加LOCK IN SHARE MODE |
| select ... for update         | 排他锁     | 需要手动在SELECT之后加FOR UPDATE         |



#### 2、演示

默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读。

- 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
- InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据时，InnoDB将对表中的所有记录加锁，此时就会升级为表锁。



可以通过以下SQL，查看意向锁及行锁的加锁情况：

```sql
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```



数据准备：

```sql
CREATE TABLE `stu` (
    `id` int NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `name` varchar(255) DEFAULT NULL,
    `age` int NOT NULL
) ENGINE = InnoDB CHARACTER SET = utf8mb4;
INSERT INTO `stu` VALUES (1, 'tom', 1);
INSERT INTO `stu` VALUES (3, 'cat', 3);
INSERT INTO `stu` VALUES (8, 'rose', 8);
INSERT INTO `stu` VALUES (11, 'jetty', 11);
INSERT INTO `stu` VALUES (19, 'lily', 19);
INSERT INTO `stu` VALUES (25, 'luci', 25);
```



1. 普通的select语句，执行时，不会加锁。

   ![image-20221011201957788](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011201957788.png)
   
2. select...lock in share mode，加共享锁，共享锁与共享锁之间兼容。

   ![image-20221011202703383](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011202703383.png)

3. 共享锁与排他锁之间互斥

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011202943314.png" alt="image-20221011202943314" style="zoom:80%;" />

   客户端一获取的是id为1这行的共享锁，客户端二是可以获取id为3这行的排它锁的，因为不是同一行数据。 而如果客户端二想获取id为1这行的排他锁，会处于阻塞状态，因为共享锁与排他锁之间互斥。

4. 排它锁与排他锁之间互斥

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011203402148.png" alt="image-20221011203402148" style="zoom:80%;" />

   当客户端一，执行update语句，会为id为1的记录加排他锁； 客户端二，如果也执行update语句更新id为1的数据，也要为id为1的数据加排他锁，但是客户端二会处于阻塞状态，因为排他锁之间是互斥的。 直到客户端一，把事务提交了，才会把这一行的行锁释放，此时客户端二，解除阻塞。

5. 无索引行锁升级为表锁

   我们在两个客户端中执行如下操作：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011203954229.png" alt="image-20221011203954229" style="zoom:80%;" />

   在客户端一中，开启事务，并执行update语句，更新name为lily的数据，也就是id为19的记录。

   然后在客户端二中更新id为3的记录，却不能直接执行，会处于阻塞状态，为什么呢？

   原因就是因为此时，客户端一，根据name字段进行更新时，name字段是没有索引的，如果没有索引，此时行锁会升级为表锁 (因为行锁是对索引项加的锁，而name没有索引)。

   再针对name字段建立索引，索引建立之后，再次做一次测试会发现，客户端二并没有进入阻塞状态。这样就说明，根据索引字段进行更新操作，就可以避免行锁升级为表锁的情况。



### 2.4.3、间隙锁&临键锁

默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用临键锁 (next-key) 进行搜索和索引扫描，以防止幻读。

- 索引上的等值查询 (唯一索引)，给不存在的记录加锁时, 优化为间隙锁。
- 索引上的等值查询 (非唯一普通索引)，向右遍历时，最后一个值不满足查询需求时，临键锁 (next-key) 退化为间隙锁。
- 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止。

> 注意：间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。



**示例**：

1. 索引上的等值查询 (唯一索引)，给不存在的记录加锁时，优化为间隙锁

   ![image-20221011205111209](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011205111209.png)
   
2. 索引上的等值查询(非唯一普通索引)，向右遍历时最后一个值不满足查询需求时，临键锁 (next-key) 退化为间隙锁

   我们知道InnoDB的B+树索引，叶子节点是有序的双向链表。假如，我们要根据这个二级索引查询值为18的数据，并加上共享锁，我们是只锁定18这一行就可以了吗？

   并不是，因为是非唯一索引，这个结构中可能有多个18的存在，所以，在加锁时会继续往后找，找到一个不满足条件的值（当前案例中也就是29）。此时会对18加临键锁，并对29之前的间隙加锁。

   ![image-20221010221811830](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221010221811830.png)

   对字段age建立普通索引：
   
   ![image-20221011210534594](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011210534594.png)
   
3. 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止

   ![image-20221011211454876](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011211454876.png)
