# 1、视图/存储过程/触发器



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