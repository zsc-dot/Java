# 一、存储引擎



## 1.1、MySQL体系结构

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002110742368.png" alt="image-20221002110742368" style="zoom: 80%;" />



1. 连接层

   最上层是一些客户端和链接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于TCP/IP的通信。

   主要完成一些类似于连接处理、授权认证、及相关的安全方案。

   在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。

   同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限。

2. 服务层

   第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。

   所有跨存储引擎的功能也在这一层实现，如过程、函数等。

   在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等，最后生成相应的执行操作。

   如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能。

3. 引擎层

   存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。

   不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。

   数据库中的索引是在存储引擎层实现的。

4. 存储层

   数据存储层，主要是将数据 (如：redolog、undolog、数据、索引、二进制日志、错误日志、查询日志、慢查询日志等) 存储在文件系统之上，并完成与存储引擎的交互。



和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。

主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。

这种架构可以根据业务的需求和实际需要选择合适的存储引擎。



## 1.2、存储引擎介绍

大家可能没有听说过存储引擎，但是一定听过引擎这个词，引擎就是发动机，是一个机器的核心组件。比如，对于舰载机、直升机、火箭来说，他们都有各自的引擎，是他们最为核心的组件。在选择引擎的时候，需要在合适的场景，选择合适的存储引擎，就像在直升机上，不能选择舰载机的引擎一样。



而对于存储引擎，也是一样，他是mysql数据库的核心，我们也需要在合适的场景选择合适的存储引擎。接下来就来介绍一下存储引擎：

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。

存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。

我们可以在创建表的时候，来指定选择的存储引擎，如果没有指定将自动选择默认的存储引擎。



1. 建表时指定存储引擎

   ```sql
   CREATE TABLE 表名(
       字段1 字段1类型 [ COMMENT 字段1注释 ] ,
       ......
       字段n 字段n类型 [COMMENT 字段n注释 ]
   ) ENGINE = INNODB [ COMMENT 表注释 ] ;
   ```

2. 查询当前数据库支持的存储引擎

   ```sql
   show engines;
   ```



**示例**：

1. 查询建表语句    -- 默认存储引擎：InnoDB

   ```sql
   show create table dept;
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002112624796.png" alt="image-20221002112624796" style="zoom:80%;" />

   我们可以看到，创建表时，即使我们没有指定存储引擎，数据库也会自动选择默认的存储引擎。

2. 查询当前数据库支持的存储引擎

   ```sql
   show engines;
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002112726725.png" alt="image-20221002112726725" style="zoom:80%;" />

3. 创建表 my_myisam，并指定MyISAM存储引擎

   ```sql
   create table my_myisam(
   	id int,
   	name varchar(10)
   ) engine = MyISAM;
   ```

4. 创建表 my_memory，指定Memory存储引擎

   ```sql
   create table my_memory(
       id int,
       name varchar(10)
   ) engine = Memory;
   ```



## 1.3、存储引擎特点



### 1.3.1、InnoDB

#### 1、介绍

InnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在 MySQL 5.5 之后，InnoDB是默认的MySQL 存储引擎。



#### 2、特点

- DML操作遵循ACID模型，支持事务
- 行级锁，提高并发访问性能
- 支持外键FOREIGN KEY约束，保证数据的完整性和正确性



#### 3、文件

xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm-早期的、sdi-新版的）、数据和索引。

参数：innodb_file_per_table

```sql
show variables like 'innodb_file_per_table';
```

![image-20221002113513541](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002113513541.png)

如果该参数开启，代表对于InnoDB引擎的表，每一张表都对应一个ibd文件。

我们直接打开MySQL的数据存放目录： C:\ProgramData\MySQL\MySQL Server 8.0\Data，这个目录下有很多文件夹，不同的文件夹代表不同的数据库，我们直接打开itcast文件夹。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002113604257.png" alt="image-20221002113604257" style="zoom:67%;" />

可以看到里面有很多的ibd文件，每一个ibd文件就对应一张表，比如：我们有一张表 account，就有这样的一个account.ibd文件，而在这个ibd文件中不仅存放表结构、数据，还会存放该表对应的索引信息。

而该文件是基于二进制存储的，不能直接基于记事本打开，我们可以使用mysql提供的一个指令`ibd2sdi`，通过该指令就可以从ibd文件中提取sdi信息，而sdi数据字典信息中就包含该表的表结构。

![image-20221002113938901](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002113938901.png)



#### 4、逻辑存储结构

![image-20221002114006657](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002114006657.png)

- 表空间：InnoDB存储引擎逻辑结构的最高层，ibd文件其实就是表空间文件，在表空间中可以包含多个Segment段。
- 段：表空间是由各个段组成的，常见的段有数据段、索引段、回滚段等。InnoDB中对于段的管理，都是引擎自身完成，不需要人为对其控制，一个段中包含多个区。
- 区：区是表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中一共有64个连续的页。
- 页：页是组成区的最小单元，页也是InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区。
- 行：InnoDB 存储引擎是面向行的，也就是说数据是按行进行存放的，在每一行中除了定义表时所指定的字段以外，还包含两个隐藏字段。



### 1.3.2、MyISAM

#### 1、介绍

MyISAM是MySQL早期的默认存储引擎。



#### 2、特点

- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快



#### 3、文件

xxx.sdi：存储表结构信息

xxx.MYD：存储数据

xxx.MYI：存储索引

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221002142148517.png" alt="image-20221002142148517" style="zoom:80%;" />



### 1.3.3、Memory

#### 1、介绍

Memory引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。



#### 2、特点

- 内存存放
- hash索引（默认）



#### 3、文件

xxx.sdi：存储表结构信息



### 1.3.4、区别及特点

| 特点         | InnoDB            | MyISAM | Memory |
| ------------ | ----------------- | ------ | ------ |
| 存储限制     | 64TB              | 有     | 有     |
| **事务安全** | 支持              | -      | -      |
| **锁机制**   | 行锁、表锁        | 表锁   | 表锁   |
| B+tree索引   | 支持              | 支持   | 支持   |
| Hash索引     | -                 | -      | 支持   |
| 全文索引     | 支持(5.6版本之后) | 支持   | -      |
| 空间使用     | 高                | 低     | N/A    |
| 内存使用     | 高                | 低     | 中等   |
| 批量插入速度 | 低                | 高     | 高     |
| **支持外键** | 支持              | -      | -      |



>面试题：
>
>InnoDB引擎与MyISAM引擎的区别？
>
>- InnoDB引擎，支持事务，而MyISAM不支持
>- InnoDB引擎，支持行锁和表锁，而MyISAM仅支持表锁，不支持行锁。
>- InnoDB引擎，支持外键，而MyISAM是不支持的。



## 1.4、存储引擎选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

- InnoDB：是Mysql的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。
- MyISAM ： 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的。MongoDB可以替代该引擎。
- MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。Redis可以替代该引擎。



## 1.5、总结

1. 体系结构

   连接层、服务层、引擎层、存储层

2. 存储引擎简介

   存储引擎决定了数据库中数据的存储、获取、更新、查询的方式。不同的存储引擎，数据的存储、获取方式是有差异的。

   ```sql
   show engines;
   create table xxx(...) engine = InnoDB;
   ```

3. 存储引擎特点

   InnoDB与MyISAM：事务、外键、行级锁

4. 存储引擎应用

   - InnoDB：存储业务系统中对于事务、数据完整性要求比较高的核心数据。
   - MyISAM：存储业务中的非核心事务



# 二、索引



## 2.1、索引概述



### 2.1.1、介绍

索引（index）是帮助MySQL高效获取数据的数据结构 (有序)。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。



### 2.1.2、演示

表结构及其数据如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003084701959.png" alt="image-20221003084701959" style="zoom: 67%;" />

假如我们要执行的SQL语句为：select * from user where age = 45;



#### 1、无索引情况

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003084859584.png" alt="image-20221003084859584" style="zoom: 67%;" />

在无索引情况下，就需要从第一行开始扫描，一直扫描到最后一行，我们称之为 全表扫描，性能很低。



#### 2、有索引情况

如果我们针对于这张表建立了索引，假设索引结构就是二叉树，那么也就意味着，会对age这个字段建立一个二叉树的索引结构。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003085216151.png" alt="image-20221003085216151" style="zoom:80%;" />

此时我们在进行查询时，只需要扫描三次就可以找到数据了，极大的提高的查询的效率。



**注意**：这里我们只是假设索引的结构是二叉树，介绍一下索引的大概原理，只是一个示意图，并不是索引的真实结构，索引的真实结构，后面会详细介绍。



### 2.1.3、特点

| 优势                                                         | 劣势                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 提高数据检索的效率，降低数据库的IO成本                       | 索引列也是要占用空间的                                       |
| 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗。 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。 |



## 2.2、索引结构



### 2.2.1、概述

MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的索引结构，主要包含以下几种：

| 索引结构            | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| B+Tree索引          | 最常见的索引类型，大部分引擎都支持 B+ 树索引                 |
| Hash索引            | 底层数据结构是用哈希表实现的, 只有精确匹配索引列的查询才有效，不支持范围查询 |
| R-tree(空间索引）   | 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-text(全文索引) | 是一种通过建立倒排索引，快速匹配文档的方式。类似于Lucene，Solr，ES |



上述是MySQL中所支持的所有的索引结构，接下来，我们再来看看不同的存储引擎对于索引结构的支持情况。

| 索引        | InnoDB          | MyISAM | Memory |
| ----------- | --------------- | ------ | ------ |
| B+tree索引  | 支持            | 支持   | 支持   |
| Hash 索引   | 不支持          | 不支持 | 支持   |
| R-tree 索引 | 不支持          | 支持   | 不支持 |
| Full-text   | 5.6版本之后支持 | 支持   | 不支持 |



**注意**：我们平常所说的索引，如果没有特别指明，都是指B+树结构组织的索引。



### 2.2.2、二叉树

假如说MySQL的索引结构采用二叉树的数据结构，比较理想的结构如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003091445861.png" alt="image-20221003091445861" style="zoom: 67%;" />

如果主键是顺序插入的，则会形成一个单向链表，结构如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003091524602.png" alt="image-20221003091524602" style="zoom: 67%;" />

所以，如果选择二叉树作为索引结构，会存在以下缺点：

- 顺序插入时，会形成一个链表，查询性能大大降低。
- 大数据量情况下，层级较深，检索速度慢。



此时大家可能会想到，我们可以选择红黑树，红黑树是一颗自平衡二叉树，那这样即使是顺序插入数据，最终形成的数据结构也是一颗平衡的二叉树，结构如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003091648053.png" alt="image-20221003091648053" style="zoom:67%;" />

但是，即使如此，由于红黑树也是一颗二叉树，所以也会存在一个缺点：

- 大数据量情况下，层级较深，检索速度慢。



所以，在MySQL的索引结构中，并没有选择二叉树或者红黑树，而选择的是B+Tree，那么什么是B+Tree呢？在详解B+Tree之前，先来介绍一个B-Tree。



### 2.2.3、B-Tree

B-Tree，B树是一种多路平衡查找树，相对于二叉树，B树每个节点可以有多个分支，即多叉。

以一颗最大度数（max-degree）为5(5阶)的b-tree为例，那这个B树每个节点最多存储4个key，5个指针：

![image-20221003091928213](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003091928213.png)



**注意**：树的度数指的是一个节点的子节点个数。



我们可以通过一个数据结构可视化的网站来简单演示一下：https://www.cs.usfca.edu/~galles/visualization/BTree.html

![image-20221003092236054](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003092236054.png)



插入一组数据：100 65 169 368 900 556 780 35 215 1200 234 888 158 90 1000 88 120 268 250 。

然后观察一些数据插入过程中，节点的变化情况。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003092724182.png" alt="image-20221003092724182" style="zoom: 80%;" />



**特点**：

- 5阶的B树，每一个节点最多存储4个key，对应5个指针。
- 一旦节点存储的key数量到达5，就会裂变，中间元素向上分裂。
- 在B树中，非叶子节点和叶子节点都会存放数据。



### 2.2.4、B+Tree

B+Tree是B-Tree的变种，我们以一颗最大度数（max-degree）为4（4阶）的b+tree为例，来看一下其结构示意图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003092912429.png" alt="image-20221003092912429" style="zoom:80%;" />

我们可以看到，两部分：

- 绿色框起来的部分，是索引部分，仅仅起到索引数据的作用，不存储数据。
- 红色框起来的部分，是数据存储部分，在其叶子节点中要存储具体的数据。



我们可以通过一个数据结构可视化的网站来简单演示一下。 https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

插入一组数据： 100 65 169 368 900 556 780 35 215 1200 234 888 158 90 1000 88 120 268 250 。然后观察一些数据插入过程中，节点的变化情况。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003093351002.png" alt="image-20221003093351002" style="zoom:67%;" />

最终我们看到，B+Tree 与 B-Tree相比，主要有以下三点区别：

- 所有的数据都会出现在叶子节点。
- 叶子节点形成一个单向链表。
- 非叶子节点仅仅起到索引数据作用，具体的数据都是在叶子节点存放的。



上述我们所看到的结构是标准的B+Tree的数据结构，接下来，我们再来看看MySQL中优化之后的B+Tree。



MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能，利于排序。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003093547258.png" alt="image-20221003093547258" style="zoom:80%;" />



### 2.2.5、Hash

MySQL中除了支持B+Tree索引，还支持一种索引类型---Hash索引。



#### 1、结构

哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中。

比如：金庸算出的哈希值为005，数据就放在005槽位上，58dda就是金庸这一行数据的哈希值。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003093756540.png" alt="image-20221003093756540" style="zoom:80%;" />

如果两个 (或多个) 键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可以通过链表来解决。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003094032142.png" alt="image-20221003094032142" style="zoom:67%;" />



#### 2、特点

1. Hash索引只能用于对等比较 (=，in)，不支持范围查询（between，>，< ，...）
2. 无法利用索引完成排序操作
3. 查询效率高，通常 (不存在hash冲突的情况) 只需要一次检索就可以了，效率通常要高于B+tree索引



#### 3、存储引擎支持

在MySQL中，支持hash索引的是Memory存储引擎。

 而InnoDB中具有自适应hash功能，hash索引是InnoDB存储引擎根据B+Tree索引在指定条件下自动构建的。



### 2.2.6、思考

为什么InnoDB存储引擎选择使用B+tree索引结构？

- 相对于二叉树，层级更少，搜索效率高
- 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低
- 相对Hash索引，B+tree支持范围匹配及排序操作



## 2.3、索引分类



### 2.3.1、索引分类

在MySQL数据库，将索引的具体类型主要分为以下几类：主键索引、唯一索引、常规索引、全文索引。

| 分类     | 含义                                                 | 特点                     | 关键字   |
| -------- | ---------------------------------------------------- | ------------------------ | -------- |
| 主键索引 | 针对于表中主键创建的索引                             | 默认自动创建, 只能有一个 | primary  |
| 唯一索引 | 避免同一个表中某数据列中的值重复                     | 可以有多个               | unique   |
| 常规索引 | 快速定位特定数据                                     | 可以有多个               |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比较索引中的值 | 可以有多个               | fulltext |



### 2.3.2、聚集索引&二级索引

而在在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：

| 分类                       | 含义                                                       | 特点                 |
| -------------------------- | ---------------------------------------------------------- | -------------------- |
| 聚集索引 (Clustered Index) | 将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据 | 必须有，而且只有一个 |
| 二级索引 (Secondary Index) | 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键 | 可以存在多个         |



聚集索引选取规则：

- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。



聚集索引和二级索引的具体结构如下：

![image-20221003095409518](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003095409518.png)



- 聚集索引的叶子节点下挂的是这一行的数据。
- 二级索引的叶子节点下挂的是该字段值对应的主键值。



接下来，我们来分析一下，当我们执行如下的SQL语句时，具体的查找过程是什么样子的：

![image-20221003095745884](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003095745884.png)

具体过程如下：

1. 由于是根据 name 字段进行查询，所以先根据 name='Arm' 到 name 字段的二级索引中进行匹配查找。但是在二级索引中只能查找到 Arm 对应的主键值10。
2. 由于查询返回的数据是*，所以此时，还需要根据主键值10，到聚集索引中查找10对应的记录，最终找到10对应的行row。
3. 最终拿到这一行的数据，直接返回即可。



> 回表查询：这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取数据的方式，就称之为回表查询。



### 2.3.3、思考

>  以下两条SQL语句，那个执行效率高？为什么？
>
> ​	A. select * from user where id = 10
>
> ​	B. select * from user where name = 'Arm'
>
> ​	备注：id为主键，name字段创建的有索引
>
> 解答：
> 	A 语句的执行性能要高于B 语句。
>
> ​	因为A语句直接走聚集索引，直接返回数据。 
>
> ​	而B语句需要先查询name字段的二级索引，然后再查询聚集索引，也就是需要进行回表查询。



> InnoDB主键索引的B+tree高度为多高呢？
>
> <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003100309413.png" alt="image-20221003100309413" style="zoom:67%;" />
>
>
> 假设：
>
> ​	一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6个字节的空间，主键即使为bigint，占用字节数为8。
>
> ​	高度为2：
>
> ​		n * 8 + (n + 1) * 6 = 16*1024 , 算出n约为 1170
>
> ​		1171* 16 = 18736
>
> ​		也就是说，如果树的高度为2，则可以存储 18000 多条记录。
>
> ​	高度为3：
>
> ​		1171 * 1171 * 16 = 21939856
>
> ​		也就是说，如果树的高度为3，则可以存储 2200w 左右的记录。



## 2.4、索引语法

1. 创建索引

   ```sql
   create [ unique | fulltext ] index 索引名称 on 表名 (字段名1, ... ) ;
   ```

2. 查看索引

   ```sql
   show index from 表名;
   ```

3. 删除索引

   ```sql
   drop index 索引名称 on 表名;
   ```



**案例**：

先来创建一张表 tb_user，并且查询测试数据。

```sql
create table tb_user(
    id int primary key auto_increment comment '主键',
    name varchar(50) not null comment '用户名',
    phone varchar(11) not null comment '手机号',
    email varchar(100) comment '邮箱',
    profession varchar(11) comment '专业',
    age tinyint unsigned comment '年龄',
    gender char(1) comment '性别 , 1: 男, 2: 女',
    status char(1) comment '状态',
    createtime datetime comment '创建时间'
) comment '系统用户表';
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('吕布', '17799990000', 'lvbu666@163.com', '软件工程', 23, '1', '6', '2001-02-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('曹操', '17799990001', 'caocao666@qq.com', '通讯工程', 33, '1', '0', '2001-03-05 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('赵云', '17799990002', '17799990@139.com', '英语', 34, '1', '2', '2002-03-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('孙悟空', '17799990003', '17799990@sina.com', '工程造价', 54, '1', '0', '2001-07-02 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('花木兰', '17799990004', '19980729@sina.com', '软件工程', 23, '2', '1', '2001-04-22 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('大乔', '17799990005', 'daqiao666@sina.com', '舞蹈', 22, '2', '0', '2001-02-07 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('露娜', '17799990006', 'luna_love@sina.com', '应用数学', 24, '2', '0', '2001-02-08 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('程咬金', '17799990007', 'chengyaojin@163.com', '化工', 38, '1', '5', '2001-05-23 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('项羽', '17799990008', 'xiaoyu666@qq.com', '金属材料', 43, '1', '0', '2001-09-18 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('白起', '17799990009', 'baiqi666@sina.com', '机械工程及其自动化', 27, '1', '2', '2001-08-16 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('韩信', '17799990010', 'hanxin520@163.com', '无机非金属材料工程', 27, '1', '0', '2001-06-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('荆轲', '17799990011', 'jingke123@163.com', '会计', 29, '1', '0', '2001-05-11 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('兰陵王', '17799990012', 'lanlinwang666@126.com', '工程造价', 44, '1', '1', '2001-04-09 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('狂铁', '17799990013', 'kuangtie@sina.com', '应用数学', 43, '1', '2', '2001-04-10 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('貂蝉', '17799990014', '84958948374@qq.com', '软件工程', 40, '2', '3', '2001-02-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('妲己', '17799990015', '2783238293@qq.com', '软件工程', 31, '2', '0', '2001-01-30 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('芈月', '17799990016', 'xiaomin2001@sina.com', '工业经济', 35, '2', '0', '2000-05-03 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('嬴政', '17799990017', '8839434342@qq.com', '化工', 38, '1', '1', '2001-08-08 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('狄仁杰', '17799990018', 'jujiamlm8166@163.com', '国际贸易', 30, '1', '0', '2007-03-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('安琪拉', '17799990019', 'jdodm1h@126.com', '城市规划', 51, '2', '0', '2001-08-15 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('典韦', '17799990020', 'ycaunanjian@163.com', '城市规划', 52, '1', '2', '2000-04-12 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('廉颇', '17799990021', 'lianpo321@126.com', '土木工程', 19, '1', '3', '2002-07-18 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('后羿', '17799990022', 'altycj2000@139.com', '城市园林', 20, '1', '0', '2002-03-10 00:00:00');
INSERT INTO tb_user (name, phone, email, profession, age, gender, status, createtime) VALUES ('姜子牙', '17799990023', '37483844@qq.com', '工程造价', 29, '1', '4', '2003-05-26 00:00:00');
```



数据准备好了之后，接下来，我们就来完成如下需求：

1. name字段为姓名字段，该字段的值可能会重复，为该字段创建索引。

   ```sql
   create index idx_user_name on table_user(name);
   ```

2. phone手机号字段的值，是非空，且唯一的，为该字段创建唯一索引。

   ```sql
   create unique index idx_user_phone on tb_user(phone);
   ```

3. 为profession、age、status创建联合索引。

   ```sql
   create index idx_user_pro_age_sta on tb_user(profession, age, status);
   ```

4. 为email建立合适的索引来提升查询效率。

   ```sql
   create index idx_user_email on tb_user(email);
   ```

5. 删除email的索引

   ```sql
   drop index idx_user_email on tb_user;
   ```



完成上述的需求之后，我们再查看tb_user表的所有的索引数据。

```sql
show index from tb_user;
```

![image-20221003103526990](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221003103526990.png)



## 2.5、SQL性能分析



### 2.5.1、SQL执行频率

MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。

通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次：

```sql
-- session 是查看当前会话 ;
-- global 是查询全局数据 ;
show global status like 'Com_______';
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004084206155.png" alt="image-20221004084206155" style="zoom:80%;" />

Com_delete：删除次数

Com_insert：插入次数

Com_select：查询次数

Com_update：更新次数

我们可以在当前数据库再执行几次查询操作，然后再次查看执行频次，看看 Com_select 参数会不会变化。



> 通过上述指令，我们可以查看到当前数据库到底是以查询为主，还是以增删改为主，从而为数据库优化提供参考依据。 
>
> 如果是以增删改为主，我们可以考虑不对其进行索引的优化。 如果是以查询为主，那么就要考虑对数据库的索引进行优化了。



那么通过查询SQL的执行频次，我们就能够知道当前数据库到底是增删改为主，还是查询为主。 

那假如说是以查询为主，我们又该如何定位针对于那些查询语句进行优化呢？ 次数我们可以借助于慢查询日志。



### 2.5.2、慢查询日志

慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志。

MySQL的慢查询日志默认没有开启，我们可以查看一下系统变量 slow_query_log。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004084448057.png" alt="image-20221004084448057" style="zoom: 80%;" />

如果要开启慢查询日志，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：

```
# 开启MySQL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
```

配置完毕之后，通过以下指令重新启动MySQL服务器进行测试，查看慢日志文件中记录的信息：/var/lib/mysql/localhost-slow.log

```sh
systemctl restart mysqld
```

然后，再次查看开关情况，慢查询日志就已经打开了。



**测试**：

1. 执行如下SQL语句 

   ```sql
   select * from tb_user; -- 这条SQL执行效率比较高，执行耗时 0.00sec
   select count(*) from tb_sku; -- 由于tb_sku表中，预先存入了1000w的记录，count一次，耗时13.35sec
   ```

2. 检查慢查询日志

   最终我们发现，在慢查询日志中，只会记录执行时间超多我们预设时间（2s）的SQL，执行较快的SQL 是不会记录的。

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004085429581.png" alt="image-20221004085429581" style="zoom: 80%;" />

   那这样，通过慢查询日志，就可以定位出执行效率比较低的SQL，从而有针对性的进行优化。



### 2.5.3、profile详情

show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。通过 have_profiling 参数，能够看到当前MySQL是否支持profile操作。

```sql
SELECT @@have_profiling;
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004085834390.png" alt="image-20221004085834390" style="zoom:80%;" />

可以看到，当前MySQL是支持 profile操作的，但是开关是关闭的。

可以通过set语句在session/global级别开启profiling：

```sql
set profiling = 1;
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004085930097.png" alt="image-20221004085930097" style="zoom:80%;" />

开关已经打开了，接下来，我们所执行的SQL语句，都会被MySQL记录，并记录执行时间消耗到哪儿去了。

我们直接执行如下的SQL语句：

```sql
select * from tb_user;
select * from tb_user where id = 1;
select * from tb_user where name = '白起';
select count(*) from tb_sku;
```

执行一系列的业务SQL的操作，然后通过如下指令查看指令的执行耗时：

```sql
-- 查看每一条SQL的耗时基本情况
show profiles;

-- 查看指定Query_ID的SQL语句各个阶段的耗时情况
show profile for query Query_ID;

-- 查看指定Query_ID的SQL语句CPU的使用情况
show profile cpu for query Query_ID;
```

查看每一条SQL的耗时情况：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004090444760.png" alt="image-20221004090444760" style="zoom:80%;" />

查看指定SQL各个阶段的耗时情况：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004090509751.png" alt="image-20221004090509751" style="zoom:80%;" />



### 2.5.4、explain

explain或者 desc 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。

语法：

```sql
-- 直接在select语句之前加上关键字 explain / desc
EXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件;
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004090740676.png" alt="image-20221004090740676" style="zoom:80%;" />



Explain 执行计划中各个字段的含义：

| 字段          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | select 查询的序列号，表示查询中执行 select 子句或者是操作表的顺序 (id相同，执行顺序从上到下；id不同，值越大，越先执行) |
| select_type   | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（SELECT/WHERE之后包含了子查询）等 |
| type          | 表示连接类型，性能由好到差的连接类型为NULL、system、const、eq_ref、ref、range、index、all |
| possible_keys | 显示可能应用在这张表上的索引，一个或多个                     |
| key           | 实际使用的索引，如果为NULL，则没有使用索引                   |
| key_len       | 表示索引中使用的字节数， 该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好 |
| rows          | MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，可能并不总是准确的 |
| filtered      | 表示返回结果的行数占需读取行数的百分比，filtered 的值越大越好 |

**注意**：

- 使用主键索引或唯一性索引时，type会出现const；使用非唯一性索引时，会出现ref
- type为all时，代表全盘扫描；type为index时，代表用了索引，但是会对索引进行扫描，遍历整个索引树



## 2.6、索引使用



### 2.6.1、验证索引效率

在讲解索引的使用原则之前，先通过一个简单的案例，来验证一下索引，看看是否能够通过索引来提升数据查询性能。在演示的时候，我们还是使用之前准备的一张表 tb_sku , 在这张表中准备了1000w的记录。

这张表中id为主键，有主键索引，而其他字段是没有建立索引的。我们先来查询其中的一条记录，看看里面的字段情况，执行如下SQL：

```sql
select * from tb_sku where id = 1\G;
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004093413058.png" alt="image-20221004093413058" style="zoom:80%;" />

可以看到即使有1000w的数据，根据id进行数据查询,性能依然很快，因为主键id是有索引的。

那么接 下来，我们再来根据 sn 字段进行查询，执行如下SQL：

```sql
SELECT * FROM tb_sku WHERE sn = '100000003145001';
```

![image-20221004093522915](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004093522915.png)

我们可以看到根据sn字段进行查询，查询返回了一条数据，结果耗时 20.78sec ，就是因为sn没有索引，而造成查询效率很低。

那么我们可以针对于sn字段，建立一个索引，建立了索引之后，我们再次根据sn进行查询，再来看一下查询耗时情况。

创建索引：

```sql
create index idx_sku_sn on tb_sku(sn);
```

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004093648679.png" alt="image-20221004093648679" style="zoom:80%;" />

创建索引时，会构建一个B+Tree数据结构，耗时也会比较长。

然后再次执行根据 sn 字段进行查询的SQL语句，查看SQL耗时。

![image-20221004093740171](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004093740171.png)

我们明显会看到，sn字段建立了索引之后，查询性能大大提升。建立索引前后，查询耗时都不是一个数量级的。



### 2.6.2、最左前缀法则

如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效 (后面的字段索引失效) 。

以 tb_user 表为例，我们先来查看一下之前 tb_user 表所创建的索引。

![image-20221004094907628](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004094907628.png)

在 tb_user 表中，有一个联合索引，这个联合索引涉及到三个字段，顺序分别为：profession，age，status。



对于最左前缀法则指的是，查询时，最左边的列，也就是profession必须存在，否则索引全部失效。

而且中间不能跳过某一列，否则该列后面的字段索引将失效。 



接下来，我们来演示几组案例，看一下具体的执行计划：

1. profession、age、status

   ```sql
   explain select * from tb_user where profession = '软件工程' and age = 31 and status = '0';
   ```

   ![image-20221004095457223](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004095457223.png)

2. profession、age

   ```sql
   explain select * from tb_user where profession = '软件工程' and age = 31;
   ```

   ![image-20221004095635442](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004095635442.png)

3. profession

   ```sql
   explain select * from tb_user where profession = '软件工程';
   ```

   ![image-20221004095711045](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004095711045.png)

以上的这三组测试中，我们发现只要联合索引最左边的字段 profession 存在，索引就会生效，只不过索引的长度不同。而且由以上三组测试，我们也可以推测出profession字段索引长度为36、age字段索引长度为2、status字段索引长度为4。



1. age、status

   ```sql
   explain select * from tb_user where age = 31 and status = '0';
   ```

   ![image-20221004095836392](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004095836392.png)

2. status

   ```sql
   explain select * from tb_user where status = '0';
   ```

   ![image-20221004095912733](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004095912733.png)

而通过上面的这两组测试，我们也可以看到索引并未生效，原因是因为不满足最左前缀法则，联合索引最左边的列profession不存在。



1. profession、status

   ```sql
   explain select * from tb_user where profession = '软件工程' and status = '0';
   ```

   ![image-20221004100033353](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004100033353.png)

   上述的SQL查询时，存在profession字段，最左边的列是存在的，索引满足最左前缀法则的基本条件。

   但是查询时，跳过了age这个列，所以后面的列索引是不会使用的，也就是索引部分生效，所以索引的长度就是36。



> 思考：
>
> ​	当执行SQL语句：explain select * from tb_user where age = 31 and status = '0' and profession = '软件工程'; 时，是否满足最左前缀法则，走不走上述的联合索引，索引长度？
>
> ![image-20221004100316891](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004100316891.png)
>
> 可以看到，是完全满足最左前缀法则的，索引长度42，联合索引是生效的。
>
> 注意：最左前缀法则中指的最左边的列，是指在查询时，联合索引的最左边的字段 (即是第一个字段) 必须存在，与我们编写SQL时，条件编写的先后顺序无关。



### 2.6.3、范围查询

联合索引中，出现范围查询 (>，<) ，范围查询右侧的列索引失效。

1. profession、age、status

   ```sql
   explain select * from tb_user where profession = '软件工程' and age > 30 and status = '0';
   ```

   ![image-20221004100742263](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004100742263.png)

   当范围查询使用 > 或 < 时，走联合索引了，但是索引的长度为38，就说明范围查询右边的 status字段是没有走索引的。

2. profession、age、status

   ```sql
   explain select * from tb_user where profession = '软件工程' and age >= 30 and status = '0';
   ```

   ![image-20221004100903078](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004100903078.png)

   当范围查询使用 >= 或 <= 时，走联合索引了，但是索引的长度为42，就说明所有的字段都是走索引的。

所以，在业务允许的情况下，尽可能的使用类似于 >= 或 <= 这类的范围查询，而避免使用 > 或 <。



### 2.6.4、索引失效情况

#### 1、 索引列运算

不要在索引列上进行运算操作，索引将失效。

在tb_user表中，除了前面介绍的联合索引之外，还有一个索引，是phone字段的单列索引。

![image-20221004102559431](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004102559431.png)



1. 当根据phone字段进行等值匹配查询时，索引生效。

   ```sql
   explain select * from tb_user where phone = '17799990015';
   ```

   ![image-20221004102655009](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004102655009.png)

2. 当根据phone字段进行函数运算操作之后，索引失效。

   ```sql
   explain select * from tb_user where substring(phone,10,2) = '15';
   ```

   ![image-20221004102739717](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004102739717.png)



#### 2、字符串不加引号

字符串类型字段使用时，不加引号，索引将失效。

接下来，我们通过两组示例，来看看对于字符串类型的字段，加单引号与不加单引号的区别：

1. profession、age、status

   ```sql
   explain select * from tb_user where profession = '软件工程' and age = 31 and status = '0';
   explain select * from tb_user where profession = '软件工程' and age = 31 and status = 0;
   ```

   ![image-20221004103146645](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004103146645.png)

2. phone

   ```sql
   explain select * from tb_user where phone = '17799990015';
   explain select * from tb_user where phone = 17799990015;
   ```

   ![image-20221004103235536](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004103235536.png)



经过上面两组示例，我们会明显的发现，如果字符串不加单引号，对于查询结果，没什么影响，但是数据库存在隐式类型转换，索引将失效。



#### 3、模糊查询

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

接下来，我们来看一下这三条SQL语句的执行效果，查看一下其执行计划：

由于下面查询语句中，都是根据profession字段查询，符合最左前缀法则，联合索引是可以生效的，我们主要看一下，模糊查询时，%加在关键字之前，和加在关键字之后的影响。

```sql
explain select * from tb_user where profession like '软件%';
explain select * from tb_user where profession like '%工程';
explain select * from tb_user where profession like '%工%';
```

![image-20221004103652356](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004103652356.png)

经过上述的测试，我们发现，在like模糊查询中，在关键字后面加%，索引可以生效。而如果在关键字前面加了%，索引将会失效。



#### 4、or连接条件

用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

```sql
explain select * from tb_user where id = 10 or age = 23;
explain select * from tb_user where phone = '17799990017' or age = 23;
```

 <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004201920039.png" alt="image-20221004201920039" style="zoom:80%;" />

由于age没有索引，所以即使id、phone有索引，索引也会失效。所以需要针对于age也要建立索引。



然后，我们可以对age字段建立索引。

```sql
create index idx_user_age on tb_user(age);
```

建立了索引之后，我们再次执行上述的SQL语句，看看前后执行计划的变化。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004202146383.png" alt="image-20221004202146383" style="zoom:80%;" />



最终，我们发现，当or连接的条件，左右两侧字段都有索引时，索引才会生效。



#### 5、数据分布影响

如果MySQL评估使用索引比全表更慢，则不使用索引。

```sql
select * from tb_user where phone >= '17799990005';
select * from tb_user where phone >= '17799990015';
```

![image-20221004202821181](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004202821181.png)

经过测试我们发现，相同的SQL语句，只是传入的字段值不同，最终的执行计划也完全不一样，这是为什么呢？

就是因为MySQL在查询时，会评估使用索引的效率与走全表扫描的效率，如果走全表扫描更快，则放弃索引，走全表扫描。 

因为索引是用来索引少量数据的，如果通过索引查询返回大批量的数据，则还不如走全表扫描来的快，此时索引就会失效。



接下来，我们再来看看 is null 与 is not null 操作是否走索引。

执行如下两条语句：

```sql
explain select * from tb_user where profession is null;
explain select * from tb_user where profession is not null;
```

![image-20221004203400593](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004203400593.png)

接下来，我们做一个操作将profession字段值全部更新为null。

然后再执行上面的两条sql，查看sql的执行计划。

![image-20221004203525226](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004203525226.png)

最终我们看到，一模一样的SQL语句，先后执行了两次，结果查询计划是不一样的，为什么会出现这种现象，这是和数据库的数据分布有关系。

查询时MySQL会评估，走索引快，还是全表扫描快，如果全表扫描更快，则放弃索引走全表扫描。因此，is null、is not null是否走索引，得具体情况具体分析，并不是固定的。



### 2.6.5、SQL提示

恢复tb_user表的数据。

把上述的 idx_user_age，idx_email 这两个之前测试使用过的索引直接删除。

1. 执行SQL

   ```sql
   explain select * from tb_user where profession = '软件工程';
   ```

   ![image-20221004204553452](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004204553452.png)

   查询走了联合索引。

2. 执行SQL，创建profession的单列索引

   ```sql
   create index idx_user_pro on tb_user(profession);
   ```

3. 创建单列索引后，再次执行上面SQL语句，查看执行计划，看看到底走哪个索引。

   ![image-20221004204723257](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004204723257.png)

   测试结果，我们可以看到，possible_keys中 idx_user_pro_age_sta、idx_user_pro 这两个索引都可能用到，最终MySQL选择了idx_user_pro_age_sta索引。这是MySQL自动选择的结果。

那么，我们能不能在查询的时候，自己来指定使用哪个索引呢？

答案是肯定的，此时就可以借助于MySQL的SQL提示来完成。 



SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

1. use index：建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进行评估）

   ```sql
   explain select * from tb_user use index(idx_user_pro) where profession = '软件工程';
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004205205294.png" alt="image-20221004205205294" style="zoom:80%;" />

2. ignore index：忽略指定的索引

   ```sql
   explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程';
   ```

   ![image-20221004205324245](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004205324245.png)

3.  force index：强制使用索引

   ```sql
   explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
   ```

   ![image-20221004205343606](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004205343606.png)



### 2.6.6、覆盖索引

尽量使用覆盖索引，减少select *。 那么什么是覆盖索引呢？

覆盖索引是指查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到。



接下来，我们来看一组SQL的执行计划，看看执行计划的差别，然后再来具体做一个解析。

```sql
explain select id, profession from tb_user where profession = '软件工程' and age = 31 and status = '0';
explain select id, profession, age, status from tb_user where profession = '软件工程' and age = 31 and status = '0';
explain select id, profession, age, status, name from tb_user where profession = '软件工程' and age = 31 and status = '0';
explain select * from tb_user where profession = '软件工程' and age = 31 and status = '0';
```

上述这几条SQL的执行结果为：

![image-20221004212844428](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004212844428.png)

从上述的执行计划我们可以看到，这四条SQL语句的执行计划前面所有的指标都是一样的，看不出来差异。但是此时，我们主要关注的是后面的Extra，前面两条SQL的结果为 Using Index，而后面两条SQL的结果为：NULL。

| Extra       | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| Using Index | 查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据 |
| NULL        | 查找使用了索引，但是需要回表查询数据                         |

因为，在tb_user表中有一个联合索引 idx_user_pro_age_sta，该索引关联了三个字段profession、age、status，而这个索引也是一个二级索引，所以叶子节点下面挂的是这一行的主键id。 

所以当我们查询返回的数据在 id、profession、age、status 之中，则直接走二级索引直接返回数据了。

如果超出这个范围，就需要拿到主键id，再去扫描聚集索引，再获取额外的数据了，这个过程就是回表。

而我们如果一直使用select * 查询返回所有字段值，很容易就会造成回表查询（除非是根据主键查询，此时只会扫描聚集索引）。



为了大家更清楚的理解，什么是覆盖索引，什么是回表查询，我们一起再来看下面的这组SQL的执行过程。

1. 表结构及索引示意图：

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004214435731.png" alt="image-20221004214435731" style="zoom:80%;" />

   id是主键，是一个聚集索引。 name字段建立了普通索引，是一个二级索引（辅助索引）。

2. 执行SQL

   ```sql
   select * from tb_user where id = 2;
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004214306232.png" alt="image-20221004214306232" style="zoom:80%;" />

   根据id查询，直接走聚集索引查询，一次索引扫描，直接返回数据，性能高。

3. 执行SQL

   ```sql
   selet id, name from tb_user where name = 'Arm';
   ```

   ![image-20221004214534368](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004214534368.png)

   虽然是根据name字段查询，查询二级索引，但是由于查询返回在字段为 id，name，在name的二级索引中，这两个值都是可以直接获取到的，因为覆盖索引，所以不需要回表查询，性能高。

4. 执行SQL

   ```sql
   selet id, name, gender from tb_user where name = 'Arm';
   ```

   ![image-20221004214652214](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221004214652214.png)
   
   由于在name的二级索引中，不包含gender，所以需要两次索引扫描，也就是需要回表查询，性能相对较差一点。



> 思考：一张表，有四个字段 (id、username、password、status)，由于数据量大，需要对以下SQL语句进行优化，该如何进行才是最优方案：
>
> select id, username, password from tb_user where username = 'itcast';
>
> 答案：针对于 username，password建立联合索引，sql为：
>
> create index idx_user_name_pass on tb_user(username, password);
>
> 这样可以避免上述的SQL语句，在查询的过程中，出现回表查询。



### 2.6.7、前缀索引

当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

1. 语法：

   ```sql
   create index 索引名 on 表名(字段名(长度)) ;
   ```

   示例：

   为tb_user表的email字段，建立长度为5的前缀索引。

   ```sql
   create index idx_email_5 on tb_user(email(5));
   ```

2. 前缀长度

   可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

   ```sql
   select count(distinct email) / count(*) from tb_user;
   select count(distinct substring(email,1,5)) / count(*) from tb_user;
   ```

3. 前缀索引的查询流程

   ![image-20221005094605206](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005094605206.png)

   回表查询后，会再去二级节点继续查询，把查询到的行数据和查询条件相比，一样的话才返回。



### 2.6.8、单列索引与联合索引

单列索引：即一个索引只包含单个列。

联合索引：即一个索引包含了多个列。

我们先来看看 tb_user 表中目前的索引情况：

![image-20221005100917956](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005100917956.png)

在查询出来的索引中，既有单列索引，又有联合索引。

接下来，我们来执行一条SQL语句，看看其执行计划：

![image-20221005101058597](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005101058597.png)

通过上述执行计划我们可以看出来，在and连接的两个字段 phone、name上都是有单列索引的，但是最终mysql只会选择一个索引，也就是说，只能走一个字段的索引，此时是会回表查询的。



我们再来创建一个phone和name字段的联合索引来查询一下执行计划：

```sql
create unique index idx_user_phone_name on tb_user(phone,name);
```

此时，查询时，就走了联合索引，而在联合索引中包含 phone、name的信息，在叶子节点下挂的是对应的主键id，所以查询是无需回表查询的。

![image-20221005101945994](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005101945994.png)



> 在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

如果查询使用的是联合索引，具体的结构示意图如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005102333634.png" alt="image-20221005102333634" style="zoom:80%;" />



## 2.7、索引设计原则

1. 针对于数据量较大，且查询比较频繁的表建立索引。
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。



## 2.8、总结

1. 索引概述

   索引是高效获取数据的数据结构

2. 索引结构

   B+Tree、Hash

3. 索引分类

   主键索引、唯一索引、常规索引、全文索引、聚集索引、二级索引

4. 索引语法

   ```sql
   create [unique] index 索引名 on 表名(字段名);
   show index from 表名;
   drop index 索引名 on 表名;
   ```

5. SQL性能分析

   执行频次、慢查询日志、profile、explain

6. 索引使用

   联合索引

   索引失效

   SQL提示

   覆盖索引

   前缀索引

   单列/联合索引

7. 索引设计原则

   表

   字段

   索引



# 三、SQL优化



## 3.1、插入数据



### 3.1.1、insert

如果我们需要一次性往数据库表中插入多条记录，可以从以下三个方面进行优化。

```sql
insert into tb_test values(1, 'tom');
insert into tb_test values(2, 'cat');
insert into tb_test values(3, 'jerry');
.....
```



#### 1、优化方案一

批量插入数据：

```sql
Insert into tb_test values(1, 'Tom'), (2, 'Cat'), (3, 'Jerry');
```

**注意**：

- 通过 insert 批量插入时，一次性插入的数据建议不超过1000条
- 如果一次性插入数据超过1000条，可以将其分割为多条 insert 插入



#### 2、优化方案二

手动控制事务：

```sql
start transaction;
insert into tb_test values(1, 'Tom'), (2, 'Cat'), (3, 'Jerry');
insert into tb_test values(4, 'Tom'), (5, 'Cat'), (6, 'Jerry');
insert into tb_test values(7, 'Tom'), (8, 'Cat'), (9, 'Jerry');
commit;
```

**注意**：

- 如果单条执行，每次执行完一条 sql ，都会自动提交一次事务，导致频繁提交事务



#### 3、优化方案三

主键顺序插入，性能要高于乱序插入。

```
主键乱序插入：8 1 9 21 88 2 4 15 89 5 7 3
主键顺序插入：1 2 3 4 5 7 8 9 15 21 88 89
```

**注意**：

- 取决于MySQL的数据组织结构



### 3.1.2、大批量插入数据

如果一次性需要插入大批量数据 (比如：几百万的记录) ，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。

可以执行如下指令，将数据脚本文件中的数据加载到表结构中：

```sql
-- 客户端连接服务端时，加上参数 -–local-infile
mysql –-local-infile -u root -p

-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;

-- 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table 表名 fields terminated by ',' lines terminated by '\n';
-- ','：每个字段之间用','分隔
-- '\n'：每行数据之间用'\n'分隔
```



**示例**：

1. 设置参数

   ```sql
   -- 客户端连接服务端时，加上参数 -–local-infile
   mysql –-local-infile -u root -p
   
   -- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
   set global local_infile = 1;
   ```

2. 新建数据库，并创建表结构

   ```sql
   CREATE TABLE `tb_user` (
       `id` INT(11) NOT NULL AUTO_INCREMENT,
       `username` VARCHAR(50) NOT NULL,
       `password` VARCHAR(50) NOT NULL,
       `name` VARCHAR(20) NOT NULL,
       `birthday` DATE DEFAULT NULL,
       `sex` CHAR(1) DEFAULT NULL,
       PRIMARY KEY (`id`),
       UNIQUE KEY `unique_user_username` (`username`)
   ) ENGINE=INNODB DEFAULT CHARSET=utf8;
   ```

3. load加载数据

   ```sql
   load data local infile '/root/load_user_100w_sort.sql' into table tb_user fields terminated by ',' lines terminated by '\n';
   ```

   可以看到，插入100w的记录，十几秒就完成了，性能很好。

**注意**：在load时，主键顺序插入性能高于乱序插入



## 3.2、主键优化

在上一小节，我们提到，主键顺序插入的性能是要高于乱序插入的。这一小节就来介绍一下具体的原因，然后再分析一下主键又该如何设计。



### 3.2.1、数据组织方式

在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表 (index organized table IOT) 。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110000210.png" alt="image-20221005110000210" style="zoom: 67%;" />

行数据都是存储在聚集索引的叶子节点上的。而我们之前也讲解过InnoDB的逻辑结构图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110030216.png" alt="image-20221005110030216" style="zoom: 67%;" />

在InnoDB引擎中，数据行是记录在逻辑结构 page 页中的，而每一个页的大小是固定的，默认16K。那也就意味着，一个页中所存储的行也是有限的，如果插入的数据行row在该页存储不小，将会存储到下一个页中，页与页之间会通过指针连接。



### 3.2.2、页分裂

页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据 (如果一行数据过大，会行溢出) ，根据主键排列。

#### 1、主键顺序插入效果

1. 从磁盘中申请页， 主键顺序插入

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110250822.png" alt="image-20221005110250822" style="zoom:50%;" />

2. 第一个页没有满，继续往第一页插入

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110314660.png" alt="image-20221005110314660" style="zoom: 50%;" />

3. 当第一个也写满之后，再写入第二个页，页与页之间会通过指针连接

   ![image-20221005110341346](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110341346.png)

4. 当第二页写满了，再往第三页写入

   ![image-20221005110404412](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110404412.png)



#### 2、主键乱序插入效果

1. 加入1#，2#页都已经写满了，存放了如图所示的数据

   ![image-20221005110447388](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110447388.png)

2. 此时再插入id为50的记录，我们来看看会发生什么现象，会再次开启一个页，写入新的页中吗？

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110515901.png" alt="image-20221005110515901" style="zoom: 80%;" />

   不会。因为，索引结构的叶子节点是有顺序的。按照顺序，应该存储在47之后。

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110616398.png" alt="image-20221005110616398" style="zoom:67%;" />

   但是47所在的 1#页，已经写满了，存储不了50对应的数据了。那么此时会开辟一个新的页 3#。

   但是并不会直接将50存入3#页，而是会将1#页后一半的数据，移动到3#页，然后在3#页，插入50。

   移动数据，并插入id为50的数据之后，那么此时，这三个页之间的数据顺序是有问题的。

   1#的下一个 页，应该是3#，3#的下一个页是2#。所以，此时，需要重新设置链表指针。

   ![image-20221005110835065](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221005110835065.png)

   上述的这种现象，称之为 "页分裂"，是比较耗费性能的操作。



### 3.2.3、页合并

目前表中已有数据的索引结构 (叶子节点) 如下：

![image-20221006103753478](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006103753478.png)

当我们对已有数据进行删除时，具体的效果如下：

当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。

![image-20221006103857205](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006103857205.png)

当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

![image-20221006103939334](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006103939334.png)

删除数据，并将页合并之后，再次插入新的数据20，则直接插入3#页

![image-20221006104006401](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006104006401.png)

这个里面所发生的合并页的这个现象，就称之为 "页合并"。



> 注意：
>
> MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。



### 3.2.4、索引设计原则

- 满足业务需求的情况下，尽量降低主键的长度。
- 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键。
- 尽量不要使用UUID做主键或者是其他自然主键，如身份证号。
- 业务操作时，避免对主键的修改。



## 3.3、order by优化

MySQL的排序，有两种方式：

- Using filesort
  - 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。
- Using index
  - 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

对于以上的两种排序方式，Using index的性能高，而Using filesort的性能低，我们在优化排序操作时，尽量要优化为 Using index。

接下来，我们来做一个测试：

1. 数据准备

   把之前测试时，为tb_user表所建立的部分索引直接删除掉，只保留以下索引。

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006105641274.png" alt="image-20221006105641274" style="zoom:80%;" />

2. 执行排序SQL

   ```sql
   explain select id, age, phone from tb_user order by age;
   ```

   ![image-20221006105802895](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006105802895.png)

   ```sql
   explain select id, age, phone from tb_user order by age,phone;
   ```

   ![image-20221006105842346](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006105842346.png)

   由于 age，phone 都没有索引，所以此时再排序时，出现Using filesort，排序性能较低。

3. 创建索引

   ```sql
   create index idx_user_age_phone_aa on tb_user(age, phone);
   ```

4. 创建索引后，根据age，phone进行升序排序

   ```sql
   explain select id, age, phone from tb_user order by age;
   ```

   ![image-20221006110141890](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006110141890.png)

   ```sql
   explain select id, age, phone from tb_user order by age, phone;
   ```

   ![image-20221006110204461](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006110204461.png)

   建立索引之后，再次进行排序查询，就由原来的Using filesort，变为了 Using index，性能就是比较高的了。

5. 创建索引后，根据age,  phone进行降序排序

   ```sql
   explain select id, age, phone from tb_user order by age desc, phone desc;
   ```

   ![image-20221006110346834](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006110346834.png)

   也出现 Using index，但是此时Extra中出现了 Backward index scan，这个代表反向扫描索引，因为在MySQL中我们创建的索引，默认索引的叶子节点是从小到大排序的，而此时我们查询排序时，是从大到小，在扫描时，就是反向扫描，就会出现 Backward index scan。 在MySQL8版本中，支持降序索引，我们也可以创建降序索引。

6. 根据phone，age进行升序排序，phone在前，age在后

   ```sql
   explain select id, age, phone from tb_user order by phone, age;
   ```

   ![image-20221006110636764](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006110636764.png)

   排序时也需要满足最左前缀法则，否则也会出现 filesort。因为在创建索引的时候，age是第一个字段，phone是第二个字段，所以排序时，也该按照这个顺序来，否则就会出现 Using filesort。

7. 根据age，phone进行降序一个升序，一个降序

   ```sql
   explain select id, age, phone from tb_user order by age asc, phone desc;
   ```

   ![image-20221006110814054](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006110814054.png)

   创建索引时，如果未指定顺序，默认都是按照升序排序的，而查询时，一个升序，一个降序，此时就会出现Using filesort。
   
   ![image-20221006111156880](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006111156880.png)
   
   为了解决上述的问题，我们可以创建一个索引，这个联合索引中 age 升序排序，phone 倒序排序。
   
8. 创建联合索引 (age 升序排序，phone 倒序排序)

   ```sql
   create index idx_user_age_phone_ad on tb_user(age asc, phone desc);
   ```

9. 然后再次执行如下SQL

   ```sql
   explain select id, age, phone from tb_user order by age asc, phone desc;
   ```



升序/降序联合索引结构图示：

![image-20221006111434576](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006111434576.png)

![image-20221006111448370](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006111448370.png)

由上述的测试，我们得出order by优化原则：

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
- 尽量使用覆盖索引，否则还会出现Using filesort。
- 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。
- 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小：sort_buffer_size(默认256k)。



## 3.4、group by优化

分组操作，我们主要来看看索引对于分组操作的影响。

首先我们先将 tb_user 表的索引全部删除掉。

再在没有索引的情况下，执行如下SQL，查询执行计划：

```sql
explain select profession, count(*) from tb_user group by profession;
```

![image-20221006112658013](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006112658013.png)

然后，我们在针对于 profession，age，status 创建一个联合索引。

```sql
create index idx_user_pro_age_sta on tb_user(profession, age, status);
```

然后再次执行上面的查询SQL：

![image-20221006113018092](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006113018092.png)

再执行如下的分组查询SQL，查看执行计划：

```sql
explain select age, count(*) from tb_user group by age;
explain select profession,age, count(*) from tb_user group by profession, age;
```

![image-20221006113750480](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006113750480.png)

我们发现，如果仅仅根据age分组，就会出现 Using temporary；而如果是根据profession，age两个字段同时分组，则不会出现 Using temporary。原因是因为对于分组操作，在联合索引中，也是符合最左前缀法则的。



所以，在分组操作中，我们需要通过以下两点进行优化，以提升性能：

- 在分组操作时，可以通过索引来提高效率。
- 分组操作时，索引的使用也是满足最左前缀法则的。 



## 3.5、limit优化

在数据量比较大时，如果进行limit分页查询，在查询时，越往后，分页查询效率越低。

我们一起来看看执行limit分页查询耗时对比：

![image-20221006154030881](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221006154030881.png)

通过测试我们会看到，越往后，分页查询效率越低，这就是分页查询的问题所在。

因为，当在进行分页查询时，如果执行`limit 2000000,10`，此时需要MySQL排序前2000010 记录，仅仅返回 2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 。



优化思路：一般分页查询时，通过创建**覆盖索引**能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

```sql
explain select * from tb_sku t, (select id from tb_sku order by id limit 2000000, 10) a where t.id = a.id;
```



## 3.6、count优化



### 3.6.1、概述

```sql
select count(*) from tb_user;
```

在之前的测试中，我们发现，如果数据量很大，在执行count操作时，是非常耗时的。

- MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(*)  的时候会直接返回这个数，效率很高；但是如果是带条件的count，MyISAM也慢。
- InnoDB 引擎就麻烦了，它执行 count(*) 的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

如果说要大幅度提升InnoDB表的count效率，主要的优化思路：自己计数 (可以借助于redis这样的数据库进行，但是如果是带条件的count又比较麻烦了)。



### 3.6.2、count用法

count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是NULL，累计值就加 1，否则不加，最后返回累计值。

用法：count(*)、count(主键)、count(字段)、count(数字)

| count用法   | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| count(主键) | InnoDB 引擎会遍历整张表，把每一行的主键id值都取出来，返回给服务层。<br />服务层拿到主键后，直接按行进行累加(主键不可能为null) |
| count(字段) | 没有not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加。<br />有not null 约束：引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加。 |
| count(数字) | InnoDB 引擎遍历整张表，但不取值。服务层对于返回的每一行，放一个数字“1”进去，直接按行进行累加。 |
| count(*)    | InnoDB 引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加。 |



按照效率排序的话：

count(字段) < count(主键) < count(数字) ≈ count(*)；

所以尽量使用 count(*)。



## 3.7、update优化

我们主要需要注意一下update语句执行时的注意事项。

```sql
update course set name = 'javaEE' where id = 1;
```

当我们在执行删除的SQL语句时，会锁定id为1这一行的数据，然后事务提交之后，行锁释放。

但是当我们执行如下SQL：

```sql
update course set name = 'SpringBoot' where name = 'PHP';
```

当我们开启多个事务，在执行上述的SQL时，我们发现行锁升级为了表锁。导致该update语句的性能大大降低。



**注意**：

- InnoDB的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁



## 3.8、总结

1. 插入数据
   - 通过insert语句批量插入数据，手动控制事务，主键顺序插入
   - 大批量插入：load data local infile
2. 主键优化
   - 主键长度尽量短、顺序插入、auto_increment
3. order by优化
   - using index：直接通过索引返回数据，性能高
   - using filesort：需要将返回的结果在排序缓冲区排序
4. group by优化
   - 索引，多字段分组满足最左前缀法则
5. limit优化
   - 覆盖索引 + 子查询
6. count优化
   - 性能：count(字段) < count(主键) < count(数字) ≈ count(*)
7. update优化
   - 根据主键/索引字段去更新，避免行锁升级为表锁