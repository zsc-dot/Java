# 一、DSL查询文档

elasticsearch的查询依然是基于JSON风格的DSL来实现的。



## 1.1、DSL查询分类

Elasticsearch提供了基于JSON的DSL（[Domain Specific Language](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)）来定义查询。常见的查询类型包括：

- **查询所有**：查询出所有数据，一般测试用。例如：match_all

- **全文检索（full text）查询**：利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如：
  - match_query
  - multi_match_query
- **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。例如：
  - ids
  - range
  - term
- **地理（geo）查询**：根据经纬度查询。例如：
  - geo_distance
  - geo_bounding_box
- **复合（compound）查询**：复合查询可以将上述各种查询条件组合起来，合并查询条件。例如：
  - bool
  - function_score



查询的语法基本一致：

```json
GET /indexName/_search
{
  "query": {
    "查询类型": {
      "查询条件": "条件值"
    }
  }
}
```

我们以查询所有为例，其中：

- 查询类型为match_all
- 没有查询条件

```json
// 查询所有
GET /indexName/_search
{
  "query": {
    "match_all": {}
  }
}
```

其它查询无非就是**查询类型**、**查询条件**的变化。



## 1.2、全文检索查询



### 1.2.1、使用场景

全文检索查询的基本流程如下：

- 对用户搜索的内容做分词，得到词条
- 根据词条去倒排索引库中匹配，得到文档id
- 根据文档id找到文档，返回给用户

比较常用的场景包括：

- 商城的输入框搜索
- 百度输入框搜索

例如京东：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908210131700.png" alt="image-20220908210131700" style="zoom:80%;" />



因为是拿着词条去匹配，因此参与搜索的字段也必须是可分词的text类型的字段。



### 1.2.2、基本语法

常见的全文检索查询包括：

- match查询：单字段查询。会对用户输入内容分词，然后去倒排索引库检索
- multi_match查询：多字段查询。与match查询类似，只不过允许同时查询多个字段，任意一个字段符合条件就算符合查询条件



match查询语法如下：

```json
GET /indexName/_search
{
  "query": {
    "match": {
      "FIELD": "TEXT"
    }
  }
}
```

mulit_match语法如下：

```json
GET /indexName/_search
{
  "query": {
    "multi_match": {
      "query": "TEXT",
      "fields": ["FIELD1", " FIELD12"]
    }
  }
}
```



### 1.2.3、示例

match查询示例：

![image-20220908210608706](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908210608706.png)



multi_match查询示例：

![image-20220908210953142](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908210953142.png)



可以看到，两种查询结果是一样的，为什么？

因为我们将brand、name、business值都利用copy_to复制到了all字段中。因此你根据三个字段搜索，和根据all字段搜索效果当然一样了。

但是，搜索字段越多，对查询性能影响越大，因此建议采用copy_to，然后单字段查询的方式。



### 1.2.4、总结

match和multi_match的区别是什么？

- match：根据一个字段查询
- multi_match：根据多个字段查询，参与查询字段越多，查询性能越差



## 1.3、精准查询

精确查询一般是查找keyword、数值、日期、boolean等类型字段。所以不会对搜索条件分词。常见的有：

- term：根据词条精确值查询
- range：根据值的范围查询

城市、星级、品牌可以精确查询，价格范围查询

![image-20220908211806009](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908211806009.png)



### 1.3.1、term查询

因为精确查询的字段搜是不分词的字段，因此查询的条件也必须是不分词的词条。查询时，用户输入的内容跟自动值完全匹配时才认为符合条件。如果用户输入的内容过多，反而搜索不到数据。

语法说明：

```json
// term查询
GET /indexName/_search
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```



示例：

当我搜索的是精确词条时，能正确查询出结果：

![image-20220908212120317](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908212120317.png)



但是，当我搜索的内容不是词条，而是多个词语形成的短语时，反而搜索不到：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908212211684.png" alt="image-20220908212211684" style="zoom:80%;" />



### 1.3.2、range查询

范围查询，一般应用在对数值类型做范围过滤的时候。比如做价格范围过滤。

基本语法：

```json
// range查询
GET /indexName/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10, // 这里的gte代表大于等于，gt则代表大于
        "lte": 20 // lte代表小于等于，lt则代表小于
      }
    }
  }
}
```



示例：

![image-20220908212443276](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908212443276.png)



### 1.3.3、总结

精确查询常见的有哪些？

- term查询：根据词条精确匹配，一般搜索keyword类型、数值类型、布尔类型、日期类型字段
- range查询：根据数值范围查询，可以是数值、日期的范围



## 1.4、地理坐标查询

所谓的地理坐标查询，其实就是根据经纬度查询，官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html

常见的使用场景包括：

- 携程：搜索我附近的酒店
- 滴滴：搜索我附近的出租车
- 微信：搜索我附近的人



### 1.4.1、矩形范围查询

geo_bounding_box：查询geo_point值落在某个矩形范围的所有文档：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908212736449.png" alt="image-20220908212736449" style="zoom: 67%;" />

查询时，需要指定矩形的**左上**、**右下**两个点的坐标，然后画出一个矩形，落在该矩形内的都是符合条件的点。

```json
// geo_bounding_box查询
GET /indexName/_search
{
  "query": {
    "geo_bounding_box": {
      "FIELD": {
        "top_left": { // 左上点
          "lat": 31.1,
          "lon": 121.5
        },
        "bottom_right": { // 右下点
          "lat": 30.9,
          "lon": 121.7
        }
      }
    }
  }
}
```



这种并不符合“附近的人”这样的需求，所以我们就不做了。



### 1.4.2、附近查询

geo_distance：查询到指定中心点小于某个距离值的所有文档：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908213120204.png" alt="image-20220908213120204" style="zoom: 67%;" />

语法：

```json
// geo_distance 查询
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance": "15km", // 半径
      "FIELD": "31.21,121.5" // 圆心
    }
  }
}
```



示例：

我们先搜索陆家嘴附近15km的酒店：

![image-20220908213342839](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908213342839.png)



## 1.5、复合查询

复合（compound）查询：复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑。常见的有两种：

- fuction score：算分函数查询，可以控制文档相关性算分，控制文档排名。例如百度广告
- bool query：布尔查询，利用逻辑关系组合多个其它的查询，实现复杂搜索



### 1.5.1、相关性算分

当我们利用match查询时，文档结果会根据与搜索词条的关联度打分（_score），返回结果时按照分值降序排列。

例如，我们搜索 "虹桥如家"，结果如下：

```json
[
  {
    "_score" : 17.850193,
    "_source" : {
      "name" : "虹桥如家酒店真不错",
    }
  },
  {
    "_score" : 12.259849,
    "_source" : {
      "name" : "外滩如家酒店真不错",
    }
  },
  {
    "_score" : 11.91091,
    "_source" : {
      "name" : "迪士尼如家酒店真不错",
    }
  }
]
```



在elasticsearch中，早期使用的打分算法是**TF-IDF算法**，公式如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908213846806.png" alt="image-20220908213846806" style="zoom: 80%;" />



在后来的5.1版本升级中，elasticsearch将算法改进为BM25算法，公式如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908214010585.png" alt="image-20220908214010585" style="zoom: 80%;" />



TF-IDF算法有一各缺陷，就是词条频率越高，文档得分也会越高，单个词条对文档影响较大。而BM25则会让单个词条的算分有一个上限，曲线更加平滑：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908214139942.png" alt="image-20220908214139942" style="zoom:67%;" />



**小结：**elasticsearch会根据词条和文档的相关度做打分，算法由两种：

- TF-IDF算法
- BM25算法，elasticsearch5.1版本后采用的算法



### 1.5.2、算分函数查询

根据相关度打分是比较合理的需求，但**合理的不一定是产品经理需要**的。

以百度为例，你搜索的结果中，并不是相关度越高排名越靠前，而是谁掏的钱多排名就越靠前。

要想认为控制相关性算分，就需要利用elasticsearch中的function score 查询了。



#### 1、语法说明

![image-20220908215631647](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220908215631647.png)



function score 查询中包含四部分内容：

- **原始查询**条件：query部分，基于这个条件搜索文档，并且基于BM25算法给文档打分，**原始算分**（query score)
- **过滤条件**：filter部分，符合该条件的文档才会重新算分
- **算分函数**：符合filter条件的文档要根据这个函数做运算，得到的**函数算分**（function score），有四种函数
  - weight：函数结果是常量
  - field_value_factor：以文档中的某个字段值作为函数结果
  - random_score：以随机数作为函数结果
  - script_score：自定义算分函数算法
- **运算模式**：算分函数的结果、原始查询的相关性算分，两者之间的运算方式，包括：
  - multiply：相乘
  - replace：用function score替换query score
  - 其它，例如：sum、avg、max、min



因此，其中的关键点是：

- 过滤条件：决定哪些文档的算分被修改
- 算分函数：决定函数算分的算法
- 运算模式：决定最终算分结果



#### 2、示例

需求：给“如家”这个品牌的酒店排名靠前一些

翻译一下这个需求，转换为之前说的四个要点：

- 原始条件：不确定，可以任意变化
- 过滤条件：brand = "如家"
- 算分函数：可以简单粗暴，直接给固定的算分结果，weight
- 运算模式：比如求和

因此最终的DSL语句如下：

```json
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {  .... }, // 原始查询，可以是任意条件
      "functions": [ // 算分函数
        {
          "filter": { // 满足的条件，品牌必须是如家
            "term": {
              "brand": "如家"
            }
          },
          "weight": 2 // 算分权重为2
        }
      ],
      "boost_mode": "sum" // 加权模式，求和
    }
  }
}
```



#### 3、小结

function score query定义的三要素是什么？

- 过滤条件：哪些文档要加分
- 算分函数：如何计算function score
- 加权方式：function score 与 query score如何运算



### 1.5.3、布尔查询

布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must_not：必须不匹配，**不参与算分**，类似“非”
- filter：必须匹配，**不参与算分**