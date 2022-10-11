# 一、InnoDB引擎



## 1.1、逻辑存储结构

InnoDB的逻辑存储结构如下图所示：

![image-20221011212533348](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011212533348.png)

1. 表空间

   表空间是InnoDB存储引擎逻辑结构的最高层，如果用户启用了参数`innodb_file_per_table`(在8.0版本中默认开启)，则每张表都会有一个表空间 (xxx.ibd)，一个mysql实例可以对应多个表空间，用于存储记录、索引等数据。

2. 段

   段，分为数据段 (Leaf node segment)、索引段 (Non-leaf node segment)、回滚段 (Rollback segment)，InnoDB是索引组织表，数据段就是B+树的叶子节点， 索引段即为B+树的非叶子节点。段用来管理多个Extent (区)。

3. 区

   区，表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中一共有64个连续的页。

4. 页

   页，是InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区。

5. 行

   行，InnoDB 存储引擎数据是按行进行存放的。

   在行中，默认有两个隐藏字段：

   - Trx_id：每次对某条记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列。
   - Roll_pointer：每次对某条引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。



## 1.2、架构



### 1.2.1、概述

MySQL5.5版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是InnoDB架构图，左侧为内存结构，右侧为磁盘结构。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20221011215146050.png" alt="image-20221011215146050" style="zoom:80%;" />



### 1.2.2、内存结构

在左侧的内存结构中，主要分为四大块：Buffer Pool、Change Buffer、Adaptive Hash Index、Log Buffer。 接下来介绍一下这四个部分。

#### 1、Buffer Pool

InnoDB存储引擎基于磁盘文件存储，访问物理硬盘和在内存中进行访问，速度相差很大，为了尽可能弥补这两者之间的I/O效率的差值，就需要把经常使用的数据加载到缓冲池中，避免每次访问都进行磁盘I/O。

在InnoDB的缓冲池中不仅缓存了索引页和数据页，还包含了undo页、插入缓存、自适应哈希索引以及InnoDB的锁信息等等。

缓冲池 Buffer Pool，是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度。

缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型：

- free page：空闲page，未被使用
- clean page：被使用page，数据没有被修改过
- dirty page：脏页，被使用page，数据被修改过，页中数据与磁盘的数据产生了不一致。类似于这个页在缓冲池中存在，并且被占用了，业务在增删改之后把缓冲页中的数据改，但数据并没有刷新到磁盘中

在专用服务器上，通常将多达80％的物理内存分配给缓冲池 。参数设置：

```sql
show variables like 'innodb_buffer_pool_size';
```

