# 1、初识Elasticsearch



## 1.1、了解ES



### 1.1.1、Elasticsearch的作用

elasticsearch是一款非常强大的开源搜索引擎，具备非常多强大功能，可以帮助我们从海量数据中快速找到需要的内容。

例如：

- 在GitHub搜索代码
- 在电商网站搜索商品
- 在百度搜索答案
- 在打车软件搜索附近的车



### 1.1.2、ELK技术栈

elasticsearch结合kibana、Logstash、Beats，也就是elastic stack（ELK）。被广泛应用在日志数据分析、实时监控等领域。

而elasticsearch是elastic stack的核心，负责存储、搜索、分析数据。

唯有elasticsearch不可被替代。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830212442271.png" alt="image-20220830212442271" style="zoom: 67%;" />



### 1.1.3、elasticsearch和lucene

elasticsearch底层是基于**lucene**来实现的。

**Lucene**是一个Java语言的搜索引擎类库，是Apache公司的顶级项目，由DougCutting于1999年研发。官网地址：https://lucene.apache.org/ 。

**Lucene的优势：**

- 易扩展
- 高性能（基于倒排索引）

Lucene的缺点：

- 只限于Java语言开发
- 学习曲线陡峭

- 不支持水平扩展



**elasticsearch的发展：**

2004年Shay Banon基于Lucene开发了Compass

2010年Shay Banon 重写了Compass，取名为Elasticsearch。

官网地址: https://www.elastic.co/cn/

目前最新的版本是：7.12.1

相比与lucene，**elasticsearch具备下列优势：**

- 支持分布式，可水平扩展

- 提供Restful接口，可被任何语言调用



### 1.1.4、为什么不是其他搜索技术

目前比较知名的搜索引擎技术排名：

1. Elasticsearch：开源的分布式搜索引擎

2. Splunk：商业项目

3. Solr：Apache的开源搜索引擎



### 1.1.5、总结

什么是elasticsearch？

- 一个开源的分布式搜索引擎，可以用来实现搜索、日志统计、分析、系统监控等功能

什么是elastic stack（ELK）？

- 是以elasticsearch为核心的技术栈，包括beats、Logstash、kibana、elasticsearch

什么是Lucene？

- 是Apache的开源搜索引擎类库，提供了搜索引擎的核心API



## 1.2、倒排索引

倒排索引的概念是基于MySQL这样的正向索引而言的。



### 1.2.1、正向索引

传统数据库（如MySQL）采用正向索引，例如给下表（tb_goods）中的id创建索引：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830213522330.png" alt="image-20220830213522330" style="zoom: 67%;" />

如果是根据id查询，那么直接走索引，查询速度非常快。



但如果是基于title做模糊查询，只能是逐行扫描数据，流程如下：

1. 用户搜索数据，条件是title符合`"%手机%"`
2. 逐行获取数据，比如id为1的数据
3. 判断数据中的title是否符合用户搜索条件
4. 如果符合则放入结果集，不符合则丢弃。回到步骤1



逐行扫描，也就是全表扫描，随着数据量增加，其查询效率也会越来越低。当数据量达到数百万时，就是一场灾难。



### 1.2.2、倒排索引

倒排索引中有两个非常重要的概念：

- 文档（`Document`）：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息
- 词条（`Term`）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条。



**创建倒排索引**是对正向索引的一种特殊处理，流程如下：

- 将每一个文档的数据利用算法分词，得到一个个词条
- 创建表，每行数据包括词条、词条所在文档id、位置等信息
- 因为词条唯一性，可以给词条创建索引，例如hash表结构索引

如图：

![image-20220830214026179](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830214026179.png)

倒排索引的**搜索流程**如下（以搜索"华为手机"为例）：

1. 用户输入条件`华为手机`进行搜索。
2. 对用户输入内容**分词**，得到词条：`华为`、`手机`。
3. 拿着词条在倒排索引中查找，可以得到包含词条的文档id：1、2、3。
4. 拿着文档id到正向索引中查找具体文档。



虽然要先查询倒排索引，再查询正向索引，但是无论是词条、还是文档id都建立了索引，查询速度非常快！无需全表扫描。



倒排索引中包含两部分内容：

- 词条词典（Term Dictionary）：记录所有词条，以及词条与倒排列表（Posting List）之间的关系，会给词条创建索引，提高查询和插入效率

- 倒排列表（Posting List）：记录词条所在的文档id、词条出现频率 、词条在文档中的位置等信息
  - 文档id：用于快速获取文档
  - 词条频率（TF）：文档在词条出现的次数，用于评分

![image-20220830214822574](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830214822574.png)



### 1.2.3、正向和倒排

那么为什么一个叫做正向索引，一个叫做倒排索引呢？

- **正向索引**是最传统的，根据id索引的方式。但根据词条查询时，必须先逐条获取每个文档，然后判断文档中是否包含所需要的词条，是**根据文档找词条的过程**。
- 而**倒排索引**则相反，是先找到用户要搜索的词条，根据词条得到保护词条的文档的id，然后根据id获取文档。是**根据词条找文档的过程**。

是不是恰好反过来了？



那么两者方式的优缺点是什么呢？

**正向索引**：

- 优点：
  - 可以给多个字段创建索引
  - 根据索引字段搜索、排序速度非常快
- 缺点：
  - 根据非索引字段，或者索引字段中的部分词条查找时，只能全表扫描。

**倒排索引**：

- 优点：
  - 根据词条搜索、模糊搜索时，速度非常快
- 缺点：
  - 只能给词条创建索引，而不是字段
  - 无法根据字段做排序



### 1.2.4、总结

什么是文档和词条？

- 每一条数据就是一个文档
- 对文档中的内容分词，得到的词语就是词条

什么是正向索引？

- 基于文档id创建索引。查询词条时必须先找到文档，而后判断是否包含词条

什么是倒排索引？

- 对文档内容分词，对词条创建索引，并记录词条所在文档的信息。查询时先根据词条查询到文档id，而后获取到文档



## 1.3、es的一些概念

elasticsearch中有很多独有的概念，与mysql中略有差别，但也有相似之处。



### 1.3.1、文档和字段

elasticsearch是面向**文档（Document）**存储的，可以是数据库中的一条商品数据，一个订单信息。

文档数据会被序列化为json格式后存储在elasticsearch中。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830215220232.png" alt="image-20220830215220232" style="zoom: 50%;" />

而Json文档中往往包含很多的**字段（Field）**，类似于数据库中的列。



### 1.3.2、索引和映射

**索引（Index）**，就是相同类型的文档的集合。

**映射（mapping）**，索引中文档的字段约束信息，类似表的结构约束

例如：

- 所有用户文档，就可以组织在一起，称为用户的索引；
- 所有商品的文档，可以组织在一起，称为商品的索引；
- 所有订单的文档，可以组织在一起，称为订单的索引；

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830215346964.png" alt="image-20220830215346964" style="zoom: 67%;" />



因此，我们可以把索引当做是数据库中的表。

数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。



### 1.3.3、mysql与elasticsearch

我们统一的把mysql与elasticsearch的概念做一下对比：

| **MySQL** | **Elasticsearch** | **说明**                                                     |
| --------- | ----------------- | ------------------------------------------------------------ |
| Table     | Index             | 索引(index)，就是文档的集合，类似数据库的表(table)           |
| Row       | Document          | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是JSON格式 |
| Column    | Field             | 字段（Field），就是JSON文档中的字段，类似数据库中的列（Column） |
| Schema    | Mapping           | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
| SQL       | DSL               | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

两者各自有自己的擅长支出：

- Mysql：擅长事务类型操作，可以确保数据的安全和一致性
- Elasticsearch：擅长海量数据的搜索、分析、计算



因此在企业中，往往是两者结合使用：

- 对安全性要求较高的写操作，使用mysql实现
- 对查询性能要求较高的搜索需求，使用elasticsearch实现
- 两者再基于某种方式，实现数据的同步，保证一致性

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220830215942151.png" alt="image-20220830215942151" style="zoom: 67%;" />



### 1.3.4、总结

文档：一条数据就是一个文档，es中是Json格式

字段：Json文档中的字段

索引：同类型文档的集合

映射：索引中文档的约束，比如字段名称、类型

elasticsearch与数据库的关系：

- 数据库负责事务类型操作
- elasticsearch负责海量数据的搜索、分析、计算



## 1.4、安装单点ES



### 1.4.1、创建网络

因为我们还需要部署kibana容器，因此需要让es和kibana容器互联。这里先创建一个网络：

```sh
docker network create es-net # es-net为自定义名称
```



### 1.4.2、加载镜像

这里我们采用elasticsearch的7.12.1版本的镜像。可以自己下载然后在上传到虚拟机中。

```sh
# 导入数据
docker load -i es.tar
```

同理还有`kibana`的tar包也需要这样做。



### 1.4.3、运行

运行docker命令，部署单点es：

```sh
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

命令解释：

- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9200:9200`：端口映射配置
- `-p 9300:9300`：ES各个节点之间互联的端口



在浏览器中输入服务器IP和ES端口， 即可看到elasticsearch的响应结果：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220831204912429.png" alt="image-20220831204912429" style="zoom: 80%;" />



## 1.5、安装kibana

kibana可以给我们提供一个elasticsearch的可视化界面，便于我们学习。



### 1.5.1、部署

运行docker命令，部署kibana

```sh
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

- `--network es-net` ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
- `-p 5601:5601`：端口映射配置

kibana启动一般比较慢，需要多等待一会，可以通过命令：

```sh
docker logs -f kibana
```

查看运行日志，看到下面的日志时，说明运行成功：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220831210139297.png" alt="image-20220831210139297" style="zoom: 80%;" />

此时，在浏览器输入地址访问：http://虚拟机IP:5601，即可看到结果。



### 1.5.2、DevTools

kibana中提供了一个DevTools界面：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905203807221.png" alt="image-20220905203807221" style="zoom:67%;" />

这个界面中可以编写DSL来操作elasticsearch。并且对DSL语句有自动补全功能。

在发送请求时，无需指定es的ip端口。因为在创建kibana容器时，已经指定了es的ip端口。



## 1.6、IK分词器

es在创建倒排索引时需要对文档分词。在搜索时，需要对用户输入内容分词。但默认的分词规则对中文处理并不友好。

我们在kibana的DevTools中测试：

```json
POST /_analyze
{
  "analyzer": "standard",
  "text": "学习java太棒了！"
}
```

语法说明：

- POST：请求方式

- /_analyze：请求路径，这里省略了[http://192.168.150.101:9200](http://192.168.150.101:9200/)，有kibana帮我们补充

- 请求参数，json风格：
  - analyzer：分词器类型，这里是默认的standard分词器
  - text：要分词的内容

处理中文分词，一般会使用IK分词器。https://github.com/medcl/elasticsearch-analysis-ik



### 1.6.1、在线安装ik插件（较慢）

```sh
# 进入容器内部
docker exec -it elasticsearch /bin/bash

# 在线下载并安装
./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

#退出
exit
#重启容器
docker restart elasticsearch
```



### 1.6.2、离线安装ik插件（推荐）

#### 1、查看数据卷目录

安装插件需要知道elasticsearch的plugins目录位置，而我们用了数据卷挂载，因此需要查看elasticsearch的数据卷目录，通过下面命令查看：

```sh
docker volume inspect es-plugins
```

显示结果：

```json
[
    {
        "CreatedAt": "2022-05-06T10:06:34+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]
```

说明plugins目录被挂载到了：`/var/lib/docker/volumes/es-plugins/_data `这个目录中。



#### 2、上传到es容器的插件数据卷中

我们需要把ik分词器解压缩，重命名为ik。

然后把解压后的文件夹上传到`/var/lib/docker/volumes/es-plugins/_data `下。



#### 3、重启es容器

```sh
# 4、重启容器
docker restart es
# 查看es日志
docker logs -f es
```



#### 4、测试

IK分词器包含两种模式：

* `ik_smart`：最少切分

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905205800400.png" alt="image-20220905205800400" style="zoom: 67%;" />

* `ik_max_word`：最细切分

  <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905205905995.png" alt="image-20220905205905995" style="zoom:67%;" />



### 1.6.3、扩展词词典

随着互联网的发展，“造词运动”也越发的频繁。出现了很多新的词语，在原有的词汇列表中并不存在。比如：“奥力给”等。

所以我们的词汇也需要不断的更新，IK分词器提供了扩展词汇的功能。

1. 打开IK分词器config目录

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905210228772.png" alt="image-20220905210228772" style="zoom:67%;" />

2. 在IKAnalyzer.cfg.xml配置文件内容添加

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
   <properties>
           <comment>IK Analyzer 扩展配置</comment>
           <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
           <entry key="ext_dict">ext.dic</entry>
   </properties>
   ```

3. 新建一个 ext.dic，可以参考config目录下复制一个配置文件进行修改

   ```properties
   奥力给
   白嫖
   ```

4. 重启elasticsearch

   ```sh
   docker restart es
   # 查看 日志
   docker logs -f elasticsearch
   ```

5. 测试效果

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905211258405.png" alt="image-20220905211258405" style="zoom:67%;" />



> 注意当前文件的编码必须是 UTF-8 格式，严禁使用Windows记事本编辑



### 1.6.4、停用词词典

在互联网项目中，在网络间传输的速度很快，所以很多语言是不允许在网络上传递的，如：关于宗教、政治等敏感词语，那么我们在搜索时也应该忽略当前词汇。

IK分词器也提供了强大的停用词功能，让我们在索引时就直接忽略当前的停用词汇表中的内容。

1. IKAnalyzer.cfg.xml配置文件内容添加

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
   <properties>
           <comment>IK Analyzer 扩展配置</comment>
           <!--用户可以在这里配置自己的扩展字典-->
           <entry key="ext_dict">ext.dic</entry>
            <!--用户可以在这里配置自己的扩展停止词字典  *** 添加停用词词典-->
           <entry key="ext_stopwords">stopword.dic</entry>
   </properties>
   ```

2. 在 stopword.dic 添加停用词

   ```properties
   的
   了
   ```

3. 重启elasticsearch

   ```sh
   # 重启服务
   docker restart elasticsearch
   docker restart kibana
   
   # 查看 日志
   docker logs -f elasticsearch
   ```

4. 测试效果

   ```json
   POST /_analyze
   {
     "analyzer": "ik_max_word",
     "text": "冲啊，奥利给，白嫖了！"
   }
   ```



### 1.6.5、总结

分词器的作用是什么？

- 创建倒排索引时对文档分词
- 用户搜索时，对输入的内容分词

IK分词器有几种模式？

- ik_smart：智能切分，粗粒度
- ik_max_word：最细切分，细粒度

IK分词器如何拓展词条？如何停用词条？

- 利用config目录的IkAnalyzer.cfg.xml文件添加拓展词典和停用词典
- 在词典中添加拓展词条或者停用词条



# 2、索引库操作

索引库就类似数据库表，mapping映射就类似表的结构。

我们要向es中存储数据，必须先创建“库”和“表”。



## 2.1、mapping映射属性

mapping是对索引库中文档的约束，常见的mapping属性包括：

- type：字段数据类型，常见的简单类型有：
  - 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  - 数值：long、integer、short、byte、double、float、
  - 布尔：boolean
  - 日期：date
  - 对象：object
- index：是否创建索引，默认为true
- analyzer：使用哪种分词器
- properties：该字段的子字段



例如下面的json文档：

```json
{
    "age": 21,
    "weight": 52.1,
    "isMarried": false,
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "score": [99.1, 99.5, 98.9],
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

对应的每个字段映射（mapping）：

- age：类型为 integer；参与搜索，因此需要index为true；无需分词器
- weight：类型为float；参与搜索，因此需要index为true；无需分词器
- isMarried：类型为boolean；参与搜索，因此需要index为true；无需分词器
- info：类型为字符串，需要分词，因此是text；参与搜索，因此需要index为true；分词器可以用ik_smart
- email：类型为字符串，但是不需要分词，因此是keyword；不参与搜索，因此需要index为false；无需分词器
- score：虽然是数组，但是我们只看元素的类型，类型为float；参与搜索，因此需要index为true；无需分词器
- name：类型为object，需要定义多个子属性
  - name.firstName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器
  - name.lastName；类型为字符串，但是不需要分词，因此是keyword；参与搜索，因此需要index为true；无需分词器



## 2.2、索引库的CRUD

这里我们统一使用Kibana编写DSL的方式来演示。



### 2.2.1、创建索引库和映射

ES中通过Restful请求操作索引库、文档。请求内容用DSL语句来表示。

**基本语法：**

- 请求方式：PUT
- 请求路径：/索引库名，可以自定义
- 请求参数：mapping映射

创建索引库和mapping的DSL语法如下：

```json
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```



**示例：**

```json
PUT /heima
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object", 
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```



属性不写index时，默认为true。



### 2.2.2、查询索引库

**基本语法**：

- 请求方式：GET
- 请求路径：/索引库名
- 请求参数：无

**格式**：

```json
GET /索引库名
```



**查询：**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905215543343.png" alt="image-20220905215543343" style="zoom: 67%;" />



### 2.2.3、修改索引库

倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改mapping**。

虽然无法修改mapping中已有的字段，但是却允许添加新的字段到mapping中，因为不会对倒排索引产生影响。



**语法说明**：

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```



**示例**：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905220216944.png" alt="image-20220905220216944" style="zoom:67%;" />



### 2.2.4、删除索引库

**语法：**

- 请求方式：DELETE
- 请求路径：/索引库名
- 请求参数：无

**格式：**

```json
DELETE /索引库名
```



**示例：**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905220330560.png" alt="image-20220905220330560" style="zoom:67%;" />



### 2.2.5、总结

索引库操作有哪些？

- 创建索引库：PUT /索引库名
- 查询索引库：GET /索引库名
- 删除索引库：DELETE /索引库名
- 添加字段：PUT /索引库名/_mapping



# 3、文档操作



## 3.1、新增文档

**语法：**

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // ...
}
```

**示例：**

```json
POST /heima/_doc/1
{
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

一定要加上文档id，不然系统会生成一个默认的id。

**响应：**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905221003691.png" alt="image-20220905221003691" style="zoom:67%;" />



## 3.2、查询文档

根据restful风格，新增是post，查询应该是get，不过查询一般都需要条件，这里我们把文档id带上。

**语法：**

```json
GET /{索引库名称}/_doc/{id}
```

**通过kibana查看数据：**

```json
GET /heima/_doc/1
```

**查看结果：**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905221232882.png" alt="image-20220905221232882" style="zoom:67%;" />



## 3.3、删除文档

删除使用DELETE请求，同样，需要根据id进行删除：

**语法：**

```json
DELETE /{索引库名}/_doc/id值
```

**示例：**

```json
# 根据id删除数据
DELETE /heima/_doc/1
```

**结果：**

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220905221348848.png" alt="image-20220905221348848" style="zoom:67%;" />



## 3.4、修改文档

修改有两种方式：

- 全量修改：直接覆盖原来的文档
- 增量修改：修改文档中的部分字段



### 3.4.1、全量修改

全量修改是覆盖原来的文档，其本质是：

- 根据指定的id删除文档
- 新增一个相同id的文档

**注意**：如果根据id删除时，id不存在，第二步的新增也会执行，也就从修改变成了新增操作了。



**语法：**

```json
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```



**示例：**

```json
PUT /heima/_doc/1
{
    "info": "高级Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```



### 3.4.2、增量修改

增量修改是只修改指定id匹配的文档中的部分字段。

**语法：**

```json
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```



**示例：**

```json
POST /heima/_update/1
{
  "doc": {
    "email": "ZhaoYun@itcast.cn"
  }
}
```



## 3.5、总结

文档操作有哪些？

- 创建文档：POST /{索引库名}/_doc/文档id   { json文档 }
- 查询文档：GET /{索引库名}/_doc/文档id
- 删除文档：DELETE /{索引库名}/_doc/文档id
- 修改文档：
  - 全量修改：PUT /{索引库名}/_doc/文档id { json文档 }
  - 增量修改：POST /{索引库名}/_update/文档id { "doc": {字段}}

