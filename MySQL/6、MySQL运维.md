# 一、日志



## 1.1、错误日志

错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障导致无法正常使用时，建议首先查看此日志。

该日志是默认开启的，默认存放目录 /var/log/，默认的日志文件名为 mysqld.log。查看日志位置：

```sql
show variables like '%log_error%';
```



## 1.2、二进制日志



### 1.2.1、介绍

二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但不包括数据查询（SELECT、SHOW）语句。

作用：

- 灾难时的数据恢复
- MySQL的主从复制。在MySQL8版本中，默认二进制日志是开启着的

涉及到的参数如下：

```sql
show variables like '%log_bin%';
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016213003106.png" alt="image-20221016213003106" style="zoom:80%;" />

参数说明：

- log_bin_basename：当前数据库服务器的binlog日志的基础名称(前缀)，具体的binlog文件名需要再该basename的基础上加上编号(编号从000001开始)
- log_bin_index：binlog的索引文件，里面记录了当前服务器关联的binlog文件有哪些



### 1.2.2、格式

MySQL服务器中提供了多种格式来记录二进制日志，具体格式及特点如下：

| 日志格式  | 含义                                                         |
| --------- | ------------------------------------------------------------ |
| statement | 基于SQL语句的日志记录，记录的是SQL语句，对数据进行修改的SQL都会记录在日志文件中。 |
| row       | 基于行的日志记录，记录的是每一行的数据变更。（默认）         |
| mixed     | 混合了statement和row两种格式，默认采用statement，在某些特殊情况下会自动切换为row进行记录。 |

```sql
show variables like '%binlog_format%';
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016213401142.png" alt="image-20221016213401142" style="zoom:80%;" />

如果我们需要配置二进制日志的格式，只需要在 /etc/my.cnf 中配置 binlog_format 参数即可。



### 1.2.3、查看

由于日志是以二进制方式存储的，不能直接读取，需要通过二进制日志查询工具 mysqlbinlog 来查看，具体语法：

```sh
mysqlbinlog [参数选项] 日志文件名称
参数选项：
	-d 指定数据库名称，只列出指定的数据库相关操作
	-o 忽略掉日志中的前n行命令
	-v 将行事件(数据变更)重构为SQL语句
	-vv 将行事件(数据变更)重构为SQL语句，并输出注释信息
```



### 1.2.4、删除

对于比较繁忙的业务系统，每天生成的binlog数据巨大，如果长时间不清除，将会占用大量磁盘空间。可以通过以下几种方式清理日志：

| 指令                                             | 含义                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ |
| reset master                                     | 删除全部 binlog 日志，删除之后，日志编号将从 binlog.000001重新开始 |
| purge master logs to 'binlog.*'                  | 删除 * 编号之前的所有日志                                    |
| purge master logs before 'yyyy-mm-dd hh24:mi:ss' | 删除日志为 "yyyy-mm-dd hh24:mi:ss" 之前产生的所有日志        |



也可以在mysql的配置文件中配置二进制日志的过期时间，设置之后二进制日志过期会自动删除：

```sql
show variables like '%binlog_expire_logs_seconds%';
```



## 1.3、查询日志

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。默认情况下，查询日志是未开启的。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016215055096.png" alt="image-20221016215055096" style="zoom:80%;" />

如果需要开启查询日志，可以修改MySQL的配置文件 /etc/my.cnf 文件，添加如下内容：

```sh
#该选项用来开启查询日志 ， 可选值 ： 0 或者 1 ； 0 代表关闭， 1 代表开启
general_log=1
#设置日志的文件名 ， 如果没有指定， 默认的文件名为 host_name.log
general_log_file=mysql_query.log
```



开启了查询日志之后，在MySQL的数据存放目录，也就是 /var/lib/mysql/ 目录下就会出现mysql_query.log 文件。

之后所有的客户端的增删改查操作都会记录在该日志文件之中，长时间运行后，该日志文件将会非常大。



## 1.4、慢查询日志

慢查询日志记录了所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于min_examined_row_limit 的所有的SQL语句的日志，默认未开启。

long_query_time 默认为10 秒，最小为 0，精度可以到微秒。

如果需要开启慢查询日志，需要在MySQL的配置文件 /etc/my.cnf 中配置如下参数：

```sh
#慢查询日志
slow_query_log=1
#执行时间参数
long_query_time=2
```



默认情况下，不会记录管理语句，也不会记录不使用索引进行查找的查询。

可以使用log_slow_admin_statements和 更改此行为 log_queries_not_using_indexes，如下所示：

```sh
#记录执行较慢的管理语句
log_slow_admin_statements =1
#记录执行较慢的未使用索引的语句
log_queries_not_using_indexes = 1
```



> 上述所有的参数配置完成之后，都需要重新启动MySQL服务器才可以生效。



## 1.5、总结

1. 错误日志
2. 二进制日志
3. 查询日志
4. 慢查询日志



# 二、主从复制



## 2.1、概述

主从复制是指将主数据库的 DDL 和 DML 操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步。

MySQL支持一台主库同时向多台从库进行复制，从库同时也可以作为其他从服务器的主库，实现链状复制。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016220814699.png" alt="image-20221016220814699" style="zoom:67%;" />

MySQL 复制的优点主要包含以下三个方面：

- 主库出现问题，可以快速切换到从库提供服务
- 实现读写分离，降低主库的访问压力
- 可以在从库中执行备份，以避免备份期间影响主库服务



## 2.2、原理

MySQL主从复制的核心就是 二进制日志，具体的过程如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016221227448.png" alt="image-20221016221227448" style="zoom: 80%;" />

从上图来看，复制分成三步：

- Master 主库在事务提交时，会把数据变更记录在二进制日志文件 Binlog 中
- 从库读取主库的二进制日志文件 Binlog，写入到从库的中继日志 Relay Log 
- slave重做中继日志中的事件，将改变反映它自己的数据



## 2.3、搭建



### 2.3.1、搭建准备

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016221612603.png" alt="image-20221016221612603" style="zoom:80%;" />

准备好两台服务器之后，在上述的两台服务器中分别安装好MySQL，并完成基础的初始化准备(安装、密码配置等操作)工作。 其中：

- 192.168.200.200 作为主服务器master
- 192.168.200.201 作为从服务器slave



### 2.3.2、主库配置

1. 修改配置文件 /etc/my.cnf

   ```sh
   #mysql 服务ID，保证整个集群环境中唯一，取值范围：1 – 232-1，默认为1
   server-id=1
   #是否只读,1 代表只读, 0 代表读写
   read-only=0
   #忽略的数据, 指不需要同步的数据库
   #binlog-ignore-db=mysql
   #指定同步的数据库
   #binlog-do-db=db01
   ```

2. 重启MySQL服务器

   ```sh
   systemctl restart mysqld
   ```

3. 登录mysql，创建远程连接的账号，并授予主从复制权限

   ```sql
   -- 创建itcast用户，并设置密码，该用户可在任意主机连接该MySQL服务
   CREATE USER 'itcast'@'%' IDENTIFIED WITH mysql_native_password BY 'Root@123456';
   
   -- 为 'itcast'@'%' 用户分配主从复制权限
   GRANT REPLICATION SLAVE ON *.* TO 'itcast'@'%';
   ```

4. 通过指令，查看二进制日志坐标

   ```sql
   show master status;
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221017114943075.png" alt="image-20221017114943075"  />

   字段含义说明：

   - file：从哪个日志文件开始推送日志文件
   - position：从哪个位置开始推送日志
   - binlog_ignore_db：指定不需要同步的数据库



### 2.3.3、从库配置

1. 修改配置文件 /etc/my.cnf

   对于从库，只需要查询即可，不需要写入数据

   ```sh
   #mysql 服务ID，保证整个集群环境中唯一，取值范围：1 – 2^32-1，和主库不一样即可
   server-id=2
   #是否只读,1 代表只读, 0 代表读写
   read-only=1
   # read-only设置为1后，只针对普通用户，超级管理员还是可以读写，需要设置super-read-only=1才能禁用
   super-read-only=1
   ```

2. 重新启动MySQL服务

   ```sh
   systemctl restart mysqld
   ```

3.  登录mysql，设置主库配置

   ```sql
   CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.200.200', SOURCE_USER='itcast', SOURCE_PASSWORD='Root@123456', SOURCE_LOG_FILE='binlog.000004', SOURCE_LOG_POS=663;
   ```

   上述是8.0.23中的语法。如果mysql是 8.0.23 之前的版本，执行如下SQL：

   ```sql
   CHANGE MASTER TO MASTER_HOST='192.168.200.200', MASTER_USER='itcast', MASTER_PASSWORD='Root@123456', MASTER_LOG_FILE='binlog.000004', MASTER_LOG_POS=663;
   ```

   | 参数名          | 含义               | 8.0.23之前      |
   | --------------- | ------------------ | --------------- |
   | SOURCE_HOST     | 主库IP地址         | MASTER_HOST     |
   | SOURCE_USER     | 连接主库的用户名   | MASTER_USER     |
   | SOURCE_PASSWORD | 连接主库的密码     | MASTER_PASSWORD |
   | SOURCE_LOG_FILE | binlog日志文件名   | MASTER_LOG_FILE |
   | SOURCE_LOG_POS  | binlog日志文件位置 | MASTER_LOG_POS  |

4. 开启同步操作

   ```sql
   start replica; # 8.0.22之后
   start slave; # 8.0.22之前
   ```

5. 查看主从同步状态

   ```sql
   show replica status; # 8.0.22之后
   show slave status; # 8.0.22之前
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221017135710773.png" alt="image-20221017135710773"  />

   这两个参数为yes，就说明主从状态是正常的。



### 2.3.4、测试

1. 在主库上创建数据库、表，并插入数据

   ```sql
   create database db01;
   use db01;
   create table tb_user(
       id int(11) primary key not null auto_increment,
       name varchar(50) not null,
       sex varchar(1)
   )engine=innodb default charset=utf8mb4;
   insert into tb_user(id,name,sex) values(null,'Tom', '1'), (null,'Trigger','0'), (null,'Dawn','1');
   ```

2. 在从库中查询数据，验证主从是否同步



如果要同步二进制日志主从复制位置之前的数据，从主库中导出sql文件，在从库中执行即可



## 2.4、总结

1. 概述

   将主库的数据变更同步到从库，从而保证主库和从库数据一致

   数据备份、失败迁移，读写分离，降低单库读写压力

2. 原理

   1. 主库会把数据变更记录在二进制日志文件binlog中
   2. 从库连接主库，读取binlog日志，并写入自身中继日志relaylog
   3. 从库重做中继日志，将改变反应为自己的数据

3. 搭建

   1. 准备服务器
   2. 配置主库
   3. 配置从库
   4. 测试主从复制



# 三、分库分表



## 3.1、介绍



### 3.1.1、问题分析

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016222556817.png" alt="image-20221016222556817" style="zoom: 67%;" />

随着互联网及移动互联网的发展，应用系统的数据量也是成指数式增长，若采用单数据库进行数据存储，存在以下性能瓶颈：

1. IO瓶颈：热点数据太多，数据库缓存不足，产生大量磁盘IO，效率较低。请求数据太多，带宽不够，网络IO瓶颈
2. CPU瓶颈：排序、分组、连接查询、聚合统计等SQL会耗费大量的CPU资源，请求数太多，CPU出现瓶颈



为了解决上述问题，我们需要对数据库进行分库分表处理：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016222711060.png" alt="image-20221016222711060" style="zoom: 80%;" />

分库分表的中心思想都是将数据分散存储，使得单一数据库/表的数据量变小来缓解单一数据库的性能问题，从而达到提升数据库性能的目的。



### 3.1.2、拆分策略

分库分表的形式，主要是两种：垂直拆分和水平拆分。而拆分的粒度，一般又分为分库和分表，所以组成的拆分策略最终如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016223548703.png" alt="image-20221016223548703" style="zoom: 67%;" />



### 3.1.3、垂直拆分

#### 1、垂直分库

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016223644263.png" alt="image-20221016223644263" style="zoom: 80%;" />

垂直分库：以表为依据，根据业务将不同表拆分到不同库中。

特点：

- 每个库的表结构都不一样
- 每个库的数据也不一样
- 所有库的并集是全量数据



#### 2、垂直分表

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016223753084.png" alt="image-20221016223753084" style="zoom:67%;" />

垂直分表：以字段为依据，根据字段属性将不同字段拆分到不同表中。

特点：

- 每个表的结构都不一样
- 每个表的数据也不一样，一般通过一列（主键/外键）关联
- 所有表的并集是全量数据



### 3.1.4、水平拆分

#### 1、水平分库

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016224211559.png" alt="image-20221016224211559" style="zoom: 80%;" />

水平分库：以字段为依据，按照一定策略，将一个库的数据拆分到多个库中。

特点：

- 每个库的表结构都一样
- 每个库的数据都不一样
- 所有库的并集是全量数据



#### 2、水平分表

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016224309085.png" alt="image-20221016224309085" style="zoom:80%;" />

水平分表：以字段为依据，按照一定策略，将一个表的数据拆分到多个表中。

特点：

- 每个表的表结构都一样
- 每个表的数据都不一样
- 所有表的并集是全量数据



> 在业务系统中，为了缓解磁盘IO及CPU的性能瓶颈，到底是垂直拆分，还是水平拆分；具体是分库，还是分表，都需要根据具体的业务需求具体分析。



### 3.1.5、实现技术

- shardingJDBC：基于AOP原理，在应用程序中对本地执行的SQL进行拦截，解析、改写、路由处理。需要自行编码配置实现，只支持java语言，性能较高
- MyCat：数据库分库分表中间件，不用调整代码即可实现分库分表，支持多种语言，性能不及前者

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221016224616377.png" alt="image-20221016224616377" style="zoom:80%;" />

本次课程，我们选择了是MyCat数据库中间件，通过MyCat中间件来完成分库分表操作。



## 3.2、MyCat概述



### 3.2.1、介绍

