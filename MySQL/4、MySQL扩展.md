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