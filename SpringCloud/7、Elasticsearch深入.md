# 一、数据聚合

**[聚合](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)**可以让我们极其方便的实现对数据的统计、分析、运算。例如：

- 什么品牌的手机最受欢迎？
- 这些手机的平均价格、最高价格、最低价格？
- 这些手机每月的销售情况如何？

实现这些统计功能的比数据库的sql要方便的多，而且查询速度非常快，可以实现近实时搜索效果。



## 1.1、聚合的种类

聚合常见的有三类：

- **桶（Bucket）**聚合：用来对文档做分组
  - TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组

- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
  - Avg：求平均值
  - Max：求最大值
  - Min：求最小值
  - Stats：同时求max、min、avg、sum等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合



> **注意：**参加聚合的字段必须是keyword、日期、数值、布尔类型



## 1.2、DSL实现聚合

现在，我们要统计所有数据中的酒店品牌有几种，其实就是按照品牌对数据分组。此时可以根据酒店品牌的名称做聚合，也就是Bucket聚合。



### 1.2.1、Bucket聚合语法

语法如下：

```json
GET /hotel/_search
{
  "size": 0,  // 设置size为0，结果中不包含文档，只包含聚合结果
  "aggs": { // 定义聚合
    "brandAgg": { // 给聚合起个名字
      "terms": { // 聚合的类型，按照品牌值聚合，所以选择term
        "field": "brand", // 参与聚合的字段
        "size": 20 // 希望获取的聚合结果数量，只显示前20个。
      }
    }
  }
}
```

结果如图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210723171948228.png" alt="image-20210723171948228" style="zoom: 67%;" />



### 1.2.2、聚合结果排序

默认情况下，Bucket聚合会统计Bucket内的文档数量，记为`_count`，并且按照`_count`降序排序。

我们可以指定order属性，自定义聚合的排序方式：

```json
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "order": {
          "_count": "asc" // 按照_count升序排列
        },
        "size": 20
      }
    }
  }
}
```



### 1.2.3、限定聚合范围

默认情况下，Bucket聚合是对索引库的所有文档做聚合，但真实场景下，用户会输入搜索条件，因此聚合必须是对搜索结果聚合。那么聚合必须添加限定条件。

我们可以限定要聚合的文档范围，只要添加query条件即可：

```json
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200 // 只对200元以下的文档聚合
      }
    }
  }, 
  "size": 0, 
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20
      }
    }
  }
}
```

这次，聚合得到的品牌明显变少了：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922210939203.png" alt="image-20220922210939203" style="zoom:67%;" />



### 1.2.4、Metric聚合语法

上节课，我们对酒店按照品牌分组，形成了一个个桶。现

在我们需要对桶内的酒店做运算，获取每个品牌的用户评分的min、max、avg等值。

这就要用到Metric聚合了，例如stat聚合：就可以获取min、max、avg等结果。

语法如下：

```json
GET /hotel/_search
{
  "size": 0, 
  "aggs": {
    "brandAgg": { 
      "terms": { 
        "field": "brand", 
        "size": 20
      },
      "aggs": { // 是brands聚合的子聚合，也就是分组后对每组分别计算
        "score_stats": { // 聚合名称
          "stats": { // 聚合类型，这里stats可以计算min、max、avg等
            "field": "score" // 聚合字段，这里是score
          }
        }
      }
    }
  }
}
```

这次的score_stats聚合是在brandAgg的聚合内部嵌套的子聚合。因为我们需要在每个桶分别计算。

另外，我们还可以给聚合结果做个排序，例如按照每个桶的酒店平均分做排序：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922212038010.png" alt="image-20220922212038010" style="zoom:67%;" />



### 1.2.5、总结

aggs代表聚合，与query同级，此时query的作用是？

- 限定聚合的的文档范围

聚合必须的三要素：

- 聚合名称
- 聚合类型
- 聚合字段

聚合可配置属性有：

- size：指定聚合结果数量
- order：指定聚合结果排序方式
- field：指定聚合字段



## 1.3、RestAPI实现聚合



### 1.3.1、API语法

聚合条件与query条件同级别，因此需要使用request.source()来指定聚合条件。

聚合条件的语法：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922212238753.png" alt="image-20220922212238753" style="zoom:67%;" />

聚合的结果也与查询结果不同，API也比较特殊。不过同样是JSON逐层解析：

![image-20220922213212063](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922213212063.png)



示例代码：

```java
@Test
void testAggregation() throws IOException {
    // 1. 准备request
    SearchRequest request = new SearchRequest("hotel");
    // 2. 准备DSL
    // 2.1. 设置size
    request.source().size(0);
    request.source().aggregation(AggregationBuilders
                                 .terms("brandAgg")
                                 .field("brand")
                                 .size(10)
                                );
    // 3. 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4. 解析结果
    Aggregations aggregations = response.getAggregations();
    // 4.1. 根据聚合名称获取聚合结果
    Terms brandTerms = aggregations.get("brandAgg");
    // 4.2. 获取buckets
    List<? extends Terms.Bucket> buckets = brandTerms.getBuckets();
    // 4.3. 遍历
    for (Terms.Bucket bucket : buckets) {
        // 4.4. 获取key
        String key = bucket.getKeyAsString();
        System.out.println(key);
    }
}
```



### 1.3.2、业务需求

需求：搜索页面的品牌、城市等信息不应该是在页面写死，而是通过聚合索引库中的酒店数据得来的：

![image-20220922214031274](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922214031274.png)



分析：

目前，页面的城市列表、星级列表、品牌列表都是写死的，并不会随着搜索结果的变化而变化。但是用户搜索条件改变时，搜索结果会跟着变化。

例如：用户搜索“东方明珠”，那搜索的酒店肯定是在上海东方明珠附近，因此，城市只能是上海，此时城市列表中就不应该显示北京、深圳、杭州这些信息了。



也就是说，搜索结果中包含哪些城市，页面就应该列出哪些城市；搜索结果中包含哪些品牌，页面就应该列出哪些品牌。

如何得知搜索结果中包含哪些品牌？如何得知搜索结果中包含哪些城市？



使用聚合功能，利用Bucket聚合，对搜索结果中的文档基于品牌分组、基于城市分组，就能得知包含哪些品牌、哪些城市了。

因为是对搜索结果聚合，因此聚合是**限定范围的聚合**，也就是说聚合的限定条件跟搜索文档的条件一致。



查看浏览器可以发现，前端其实已经发出了这样的一个请求：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922214550009.png" alt="image-20220922214550009" style="zoom:67%;" />

**请求参数与搜索文档的参数完全一致**。



返回值类型就是页面要展示的最终结果：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20210723203915982.png" alt="image-20210723203915982" style="zoom: 80%;" />

结果是一个Map结构：

- key是字符串，城市、星级、品牌、价格
- value是集合，例如多个城市的名称



### 1.3.3、业务实现

在`cn.itcast.hotel.web`包的`HotelController`中添加一个方法，遵循下面的要求：

- 请求方式：`POST`
- 请求路径：`/hotel/filters`
- 请求参数：`RequestParams`，与搜索文档的参数一致
- 返回值类型：`Map<String, List<String>>`

代码：

1. controller层

   ```java
   @PostMapping("filters")
   public Map<String, List<String>> getFilters(@RequestBody RequestParams params){
       return hotelService.getFilters(params);
   }
   ```

2. 这里调用了IHotelService中的getFilters方法，尚未实现。

   在`cn.itcast.hotel.service.IHotelService`中定义新方法：

   ```java
   Map<String, List<String>> filters(RequestParams params);
   ```

3. 在`cn.itcast.hotel.service.impl.HotelService`中实现该方法：

   ```java
   @Override
   public Map<String, List<String>> filters(RequestParams params) {
       try {
           // 1.准备Request
           SearchRequest request = new SearchRequest("hotel");
           // 2.准备DSL
           // 2.1.query
           buildBasicQuery(params, request);
           // 2.2.设置size
           request.source().size(0);
           // 2.3.聚合
           buildAggregation(request);
           // 3.发出请求
           SearchResponse response = client.search(request, RequestOptions.DEFAULT);
           // 4.解析结果
           Map<String, List<String>> result = new HashMap<>();
           Aggregations aggregations = response.getAggregations();
           // 4.1.根据品牌名称，获取品牌结果
           List<String> brandList = getAggByName(aggregations, "brandAgg");
           result.put("品牌", brandList);
           // 4.2.根据品牌名称，获取品牌结果
           List<String> cityList = getAggByName(aggregations, "cityAgg");
           result.put("城市", cityList);
           // 4.3.根据品牌名称，获取品牌结果
           List<String> starList = getAggByName(aggregations, "starAgg");
           result.put("星级", starList);
   
           return result;
       } catch (IOException e) {
           throw new RuntimeException(e);
       }
   }
   
   private void buildAggregation(SearchRequest request) {
       request.source().aggregation(AggregationBuilders
                                    .terms("brandAgg")
                                    .field("brand")
                                    .size(10)
                                   );
       request.source().aggregation(AggregationBuilders
                                    .terms("cityAgg")
                                    .field("city")
                                    .size(10)
                                   );
       request.source().aggregation(AggregationBuilders
                                    .terms("starAgg")
                                    .field("starName")
                                    .size(10)
                                   );
   }
   
   private List<String> getAggByName(Aggregations aggregations, String aggName) {
       // 4.1.根据聚合名称获取聚合结果
       Terms brandTerms = aggregations.get(aggName);
       // 4.2.获取buckets
       List<? extends Terms.Bucket> buckets = brandTerms.getBuckets();
       // 4.3.遍历
       List<String> brandList = new ArrayList<>();
       for (Terms.Bucket bucket : buckets) {
           // 4.4.获取key
           String key = bucket.getKeyAsString();
           brandList.add(key);
       }
       return brandList;
   }
   ```

   

# 二、自动补全

当用户在搜索框输入字符时，我们应该提示出与该字符有关的搜索项，如图：

![image-20220922221322180](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220922221322180.png)

这种根据用户输入的字母，提示完整词条的功能，就是自动补全了。

因为需要根据拼音字母来推断，因此要用到拼音分词功能。



## 2.1、拼音分词器

要实现根据字母做补全，就必须对文档按照拼音分词。在GitHub上恰好有elasticsearch的拼音分词插件。地址：https://github.com/medcl/elasticsearch-analysis-pinyin



安装方式与IK分词器一样，分三步：

1. 解压
2. 上传到虚拟机中，elasticsearch的plugin目录
3. 重启elasticsearch
4. 测试

详细安装步骤可以参考IK分词器的安装过程。



测试用法如下：

```json
POST /_analyze
{
  "text": "如家酒店还不错",
  "analyzer": "pinyin"
}
```

结果：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925105405136.png" alt="image-20220925105405136" style="zoom:80%;" />



## 2.2、自定义分词器

默认的拼音分词器会将每个汉字单独分为拼音，而我们希望的是每个词条形成一组拼音，需要对拼音分词器做个性化定制，形成自定义分词器。



elasticsearch中分词器（analyzer）的组成包含三部分：

- character filters：在tokenizer之前对文本进行处理。例如删除字符、替换字符
- tokenizer：将文本按照一定的规则切割成词条（term）。例如keyword，就是不分词；还有ik_smart
- tokenizer filter：将tokenizer输出的词条做进一步处理。例如大小写转换、同义词处理、拼音处理等

文档分词时会依次由这三部分来处理文档：

![image-20220925211351412](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925211351412.png)

我们可以在创建索引库时，通过settings来配置自定义的analyzer（分词器）。

声明自定义分词器的语法如下：

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": {  // 自定义分词器
        "my_analyzer": { // 分词器名称
          "tokenizer": "ik_max_word", // 先用ik_max_word分词
          "filter": "py" // 再把分好的词交给py处理
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```



测试：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925212328487.png" alt="image-20220925212328487" style="zoom:80%;" />



拼音分词器适合在创建倒排索引的时候使用，但不能在搜索的时候使用。

创建倒排索引时：

![image-20220925212639534](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925212639534.png)



因此字段在创建倒排索引时应该用my_analyzer分词器；字段在搜索时应该使用ik_smart分词器：

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { 
        "my_analyzer": { 
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": {
        "py": { 
          "type": "pinyin",
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```



总结：

如何使用拼音分词器？

- 下载pinyin分词器

- 解压并放到elasticsearch的plugin目录

- 重启即可

如何自定义分词器？

- 创建索引库时，在settings中配置，可以包含三部分

- character filter

- tokenizer

- filter

拼音分词器注意事项？

- 为了避免搜索到同音字，搜索时不要使用拼音分词器



## 2.3、自动补全查询

elasticsearch提供了[Completion Suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-suggesters.html)查询来实现自动补全功能。这个查询会匹配以用户输入内容开头的词条并返回。为了提高补全查询的效率，对于文档中字段的类型有一些约束：

- 参与补全查询的字段必须是completion类型。
- 字段的内容一般是用来补全的多个词条形成的数组。

比如，一个这样的索引库：

```json
// 创建索引库
PUT test
{
  "mappings": {
    "properties": {
      "title":{
        "type": "completion"
      }
    }
  }
}
```



然后插入下面的数据：

```json
// 示例数据
POST test/_doc
{
  "title": ["Sony", "WH-1000XM3"]
}
POST test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST test/_doc
{
  "title": ["Nintendo", "switch"]
}
```



查询的DSL语句如下：

```json
// 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 补全查询的字段
        "skip_duplicates": true, // 跳过重复的
        "size": 10 // 获取前10条结果
      }
    }
  }
}
```



自动补全对字段的要求：

- 类型是completion类型
- 字段值是多词条的数组



## 2.4、实现酒店搜索框自动补全

现在，我们的hotel索引库还没有设置拼音分词器，需要修改索引库中的配置。但是我们知道索引库是无法修改的，只能删除然后重新创建。

另外，我们需要添加一个字段，用来做自动补全，将brand、suggestion、city等都放进去，作为自动补全的提示。



因此，总结一下，我们需要做的事情包括：

1. 修改hotel索引库结构，设置自定义拼音分词器

2. 修改索引库的name、all字段，使用自定义分词器

3. 索引库添加一个新字段suggestion，类型为completion类型，使用自定义的分词器

4. 给HotelDoc类添加suggestion字段，内容包含brand、business

5. 重新导入数据到hotel库



### 2.4.1、修改酒店映射结构

代码如下：

```json
// 酒店数据索引库
PUT /hotel
{
  "settings": {
    "analysis": {
      "analyzer": {
        "text_anlyzer": { // 全文检索分词器
          "tokenizer": "ik_max_word",
          "filter": "py"
        },
        "completion_analyzer": { // 自动补全分词器
          "tokenizer": "keyword", // 自动补全的数组里面本来就是词条，所以用keyword就行
          "filter": "py"
        }
      },
      "filter": {
        "py": {
          "type": "pinyin",
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id":{
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "text_anlyzer",
        "search_analyzer": "ik_smart",
        "copy_to": "all"
      },
      "address":{
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "integer"
      },
      "score":{
        "type": "integer"
      },
      "brand":{
        "type": "keyword",
        "copy_to": "all"
      },
      "city":{
        "type": "keyword"
      },
      "starName":{
        "type": "keyword"
      },
      "business":{
        "type": "keyword",
        "copy_to": "all"
      },
      "location":{
        "type": "geo_point"
      },
      "pic":{
        "type": "keyword",
        "index": false
      },
      "all":{
        "type": "text",
        "analyzer": "text_anlyzer",
        "search_analyzer": "ik_smart"
      },
      "suggestion":{
          "type": "completion",
          "analyzer": "completion_analyzer"
      }
    }
  }
}
```



### 2.4.2、修改HotelDoc实体

```java
@Data
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
    private String pic;
    private Object distance;
    private Boolean isAD;
    private List<String> suggestion;

    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location = hotel.getLatitude() + ", " + hotel.getLongitude();
        this.pic = hotel.getPic();
        // 组装suggestion
        if (this.business.contains("/")) {
            // business有多个值，需要切割
            String[] arr = business.split("/");
            // 添加元素
            this.suggestion = new ArrayList<>();
            this.suggestion.add(this.brand);
            Collections.addAll(this.suggestion, arr);
        }else {
            this.suggestion = Arrays.asList(this.brand, this.business);
        }
    }
}
```



### 2.4.3、重新导入

重新执行之前编写的导入数据功能，可以看到新的酒店数据中包含了suggestion：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925220528707.png" alt="image-20220925220528707" style="zoom: 80%;" />



### 2.4.4、自动补全查询的JavaAPI

之前我们学习了自动补全查询的DSL，而没有学习对应的JavaAPI，这里给出一个示例：

![image-20220925220636303](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925220636303.png)



而自动补全的结果也比较特殊，解析的代码如下：

![image-20220925220749416](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220925220749416.png)

测试代码如下：

```java
@Test
void testSuggest() throws IOException {
    // 1. 准备request
    SearchRequest request = new SearchRequest("hotel");
    // 2. 准备DSL
    request.source().suggest(new SuggestBuilder().addSuggestion(
        "suggestions",
        SuggestBuilders.completionSuggestion("suggestion")
        .prefix("h")
        .skipDuplicates(true)
        .size(10)
    ));
    // 3. 发起请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    // 4. 处理结果
    Suggest suggest = response.getSuggest();
    // 4.1. 根据名称获取补全结果
    CompletionSuggestion suggestions = suggest.getSuggestion("suggestions");
    // 4.2. 获取options
    List<CompletionSuggestion.Entry.Option> options = suggestions.getOptions();
    // 4.3. 遍历
    for (CompletionSuggestion.Entry.Option option : options) {
        String text = option.getText().toString();
        System.out.println(text);
    }
}
```



### 2.4.5、实现搜索框自动补全

查看前端页面，可以发现当我们在输入框键入时，前端会发起ajax请求。

返回值是补全词条的集合，类型为`List<String>`



1. 在`cn.itcast.hotel.web`包下的`HotelController`中添加新接口，接收新的请求：

   ```java
   @GetMapping("suggestion")
   public List<String> getSuggestions(@RequestParam("key") String prefix) {
       return hotelService.getSuggestions(prefix);
   }
   ```

2. 在`cn.itcast.hotel.service`包下的`IhotelService`中添加方法：

   ```java
   List<String> getSuggestions(String prefix);
   ```

3. 在`cn.itcast.hotel.service.impl.HotelService`中实现该方法：

   ```java
   @Override
   public List<String> getSuggestions(String prefix) {
       try {
           // 1.准备Request
           SearchRequest request = new SearchRequest("hotel");
           // 2.准备DSL
           request.source().suggest(new SuggestBuilder().addSuggestion(
               "suggestions",
               SuggestBuilders.completionSuggestion("suggestion")
               .prefix(prefix)
               .skipDuplicates(true)
               .size(10)
           ));
           // 3.发起请求
           SearchResponse response = client.search(request, RequestOptions.DEFAULT);
           // 4.解析结果
           Suggest suggest = response.getSuggest();
           // 4.1.根据补全查询名称，获取补全结果
           CompletionSuggestion suggestions = suggest.getSuggestion("suggestions");
           // 4.2.获取options
           List<CompletionSuggestion.Entry.Option> options = suggestions.getOptions();
           // 4.3.遍历
           List<String> list = new ArrayList<>(options.size());
           for (CompletionSuggestion.Entry.Option option : options) {
               String text = option.getText().toString();
               list.add(text);
           }
           return list;
       } catch (IOException e) {
           throw new RuntimeException(e);
       }
   }
   ```



# 三、数据同步

