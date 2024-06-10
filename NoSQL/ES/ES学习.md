

## 初识elasticsearch

**正向索引和倒排索引**

elasticsearch采用倒排索引:

- · 文档(document):每条数据就是一个文档
- 词条(term):文档按照语义分成的词语

<img src="/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/1.png" alt="image-20240608225433122" style="zoom:40%;" /><img src="/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/2.png" alt="image-20240608225610680" style="zoom: 25%;" />



### **文档（Document）** 

elasticsearch是面向文档存储的，可以是数据库中的一条商品数据，一个订单信息文档数据会被序列化为json格式后存储在elasticsearch中。

![image-20240608225737108](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/3.png)

### 索引（Index）

- 索引(index):相同类型的文档的集合

  > 类似mysql对表结构

![image-20240608225835582](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/4.png)

### 概念对比

| MySQL  | Elasticsearch | 说明                                                         |
| ------ | ------------- | ------------------------------------------------------------ |
| Table  | Index         | 索引(index)，就是文档的集合，类似数据库的表(table)           |
| Row    | Document      | 文档(Document)，就是一条条的数据，类似数据库中的行(Row)，文档都是JSON格式 |
| Column | Field         | 字段(Field)，就是JSON文档中的字段，类似数据库中的列(Column） |
| Schema | Mapping       | Mapping(映射)是索引中文档的约束，例如字段类型约束。类似数据库的表结构(Schema) |
| SQL    | DSL           | DSL是elasticsearch提供的JSON风格的请求语句，用来操作elasticsearch，实现CRUD |

### 架构

- Mysql:擅长事务类型操作，可以确保数据的安全和一致性
- Elasticsearch:擅长海量数据的搜索、分析、计算

![image-20240609132233638](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/6.png)



### 分词器

es在创建倒排索引时需要对文档分词;在搜索时，需要对用户输入内容分词。但默认的分词规则对中文处理并不友好。

![image-20240609133626742](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/7.png)

> analyzer 是分词器类型
>
> text是要分词的文档

处理中文分词器一般使用IK分词器。https://github.com/infinilabs/analysis-ik

**使用IK分词器**

![image-20240609141146147](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/11.png)

#### 安装IK分词器

> 见GitHub的README文件，注意统一版本

![image-20240604222356583](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/12.png)

![image-20240604222411174](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/13.png)

分词器的作用是什么?

- 创建倒排索引时对文档分词
- 用户搜索时，对输入的内容分词

IK分词器有几种模式?

- ik_smart：智能切分，粗粒度
- ·ik_max_word：最细切分，细粒度

IK分词器如何拓展词条?如何停用词条?

- 利用config目录的IkAnalyzer.cfg.xml文件添加拓展词典和停用词典
- 在词典中添加拓展词条或者停用词条

## 索引库操作

### mapping属性

mapping是对索引库中文档的约束，常见的mapping属性包括：

- type：字段数据类型，常见的简单类型有:

  - 对象：object

  - **文本（Text）**：用于全文搜索的非分析字段，**Elasticsearch会使用分词器对其进行分词**。

  - **关键词（Keyword）**：用于精确匹配的字段，不会进行分词处理。

  - **整数（Integer）**：用于存储整数值，可以是32位的。

  - **长整型（Long）**：用于存储较大范围的整数值，是64位的。

  - **浮点型（Float）**：用于存储单精度浮点数。

  - **双精度型（Double）**：用于存储双精度浮点数。

  - **日期（Date）**：用于存储日期和时间。

  - **布尔型（Boolean）**：用于存储真或假的值。

  - **二进制（Binary）**：用于存储二进制数据。

  - **范围类型（Range Types）**：
    - 日期范围（Date Range）
    - 长整型范围（Long Range）
    - 双精度型范围（Double Range）
    
  - **Nested** 嵌套对象

  - **多字段（Multi-field）**：一个字段可以有多个类型，例如一个字段可以同时是Text和Keyword。

  - **IP地址（IP）**：用于存储IP地址。

  - **完成型（Completion）**：用于自动完成功能的字段。

  - **令牌计数（Token Count）**：用于控制字段中可以包含的术语数量。

  - **地理点（Geo-point）**：用于存储地理坐标。

  - **地理形状（Geo-shape）**：用于存储地理区域。

    > **ES中支持两种地理坐标数据类型:**
    > geo_point：由纬度(latitude)和经度(longitude)确定的一个点。例如:"32.8752345,120.2981576"
    > geo_shape：有多个geo_point组成的复杂几何图形。例如一条直线，"LINESTRING(-77.03653 38.897676,-77.009051 38.889939)"

  - **数组（Array）**：Elasticsearch可以索引数组类型的字段，数组中的每个元素都将被单独索引。

  - **常量关键字（Constant Keyword）**：用于存储不会改变的关键词。

  - **搜索时分析的文本（Text with Search As You Type Analyzer）**：一种特殊的文本字段，使用`search_as_you_type`分析器，适用于实现自动完成和建议功能。

- index:是否创建索引，默认为true

- analyzer：使用哪种分词器

- properties：该字段的子字段

![image-20240609191415355](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/10.png)

```json
{
  "_index": "index",
  "_type": "_doc",
  "_id": "UQK1_I8BUVoIbccrWOVY",
  "_version": 1,
  "_seq_no": 15,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "className": "分类名称",
    "email": "qqq@qq.com",
    "indexName": "indexaaza Name 中国",
    "info": "这个是一个信息滴滴滴滴",
    "ip": "192.101.191.1"
  }
}
```



### 创建索引库

ES中通过Restful请求操作索引库、文档。请求内容用DSL语句来表示。创建索引库和mapping的DSL语法如下：

```
PUT /indexName
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type":"keyword"
      },
      "date":{
        "type":"date"
      },
      "location":{
      "type":"geo_point"
      },
      "ip":{
        "type":"ip"
      }
    }
  }
}
```

### 查看索引库

```
GET /indexName
```

### 删除索引库

```
DELETE /indexName
```

### 修改索引库

> 索引库和mapping一旦创建无法修改，但是可以添加新的字段，语法如下

添加新字段：

```json
PUT /index/_mapping
{
  "properties": {
    "indexName": {
      "type": "text"
    }
  }
}
```

**修改现有字段的映射**（注意：Elasticsearch通常不允许直接修改字段的类型，但可以更新字段的分析器等属性）：

```

```



## 文档操作

### 新增文档

```
# id自己不写 就会自动生成
POST /index/_doc/id

{
  "className":"分类名称",
  "email":"qqq@qq.com",
  "indexName":"indexaaza Name 中国",
  "info":"这个是一个信息滴滴滴滴",
  "ip":"192.101.191.1"
}
```

![image-20240609173753479](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/14.png)

### **查询文档**

```
GET /index/_doc/文档ID
```

![image-20240609173925625](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/15.png)

### 删除文档

```
DELETE /index/_doc/文档ID
```

### 修改文档

方式一：全量修改，会删除旧文档，添加新文档

```
PUT /index/_doc/_id
```

![image-20240609180800892](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/16.png)

方式二:增量修改，修改指定字段值

![image-20240609180849050](/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/17.png)



## RestClient操作索引库

ES官方提供了各种不同语言的客户端，用来操作ES。这些客户端的本质就是组装DSL语句，通过http请求发送给ES。文档地址：https://www.elastic.co/guide/en/elasticsearch/client/java-api-client/current/getting-started-java.html

### 创建索引库

引入dependence

```xml
<dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.17.21</version>
        </dependency>
```

```java
private static void createIndex() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        // 索引库名字
        CreateIndexRequest request = new CreateIndexRequest("hotel1");
        // DSL和对应的报文形式
        request.source(INDEX_DSL, XContentType.JSON);
        IndicesClient indices = restHighLevelClient.indices();
        indices.create(request, RequestOptions.DEFAULT);
    }
```

### 删除索引库

```java
private static void deleteIndex() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        // 索引库名字
        DeleteIndexRequest request = new DeleteIndexRequest("hotel1");
        IndicesClient indices = restHighLevelClient.indices();
        AcknowledgedResponse delete = indices.delete(request, RequestOptions.DEFAULT);
        System.out.println(delete.toString());
    }
```

### 判断索引库是否存在

```java
private static void existIndex() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        // 索引库名字
        GetIndexRequest request = new GetIndexRequest("hotel1");
        IndicesClient indices = restHighLevelClient.indices();
        boolean exists = indices.exists(request, RequestOptions.DEFAULT);
        System.out.println(exists);
    }
```



## RestClient操作文档

### 新增文档

```java
private void createDocument() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        // 索引库名字
        IndexRequest request = new IndexRequest("hotel1");
        User user = new User();
        user.setUserInfo("库吗今天的天气");
        user.setIp("120.0.0.0");
        user.setUserName("jack");
        user.setEmail("11@qq.com");
        request.source(new Gson().toJson(user), XContentType.JSON);
        IndexResponse index = restHighLevelClient.index(request, RequestOptions.DEFAULT);
        System.out.println(index);
    }
```

### 查询文档

```java
private void queryDocument() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        // 索引库名字 _id
        GetRequest request = new GetRequest("hotel1", "UgLd_I8BUVoIbccrgOXM");
        GetResponse documentFields = restHighLevelClient.get(request, RequestOptions.DEFAULT);
        String sourceAsString = documentFields.getSourceAsString();
        Gson gson = new Gson();
        User user = gson.fromJson(sourceAsString, User.class);
        System.out.println(user);
        System.out.println(documentFields);
    }
```

### 删除文档

```java
 private void deleteDocument() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        DeleteRequest request = new DeleteRequest("hotel1", "UgLd_I8BUVoIbccrgOXM");
        DeleteResponse delete = restHighLevelClient.delete(request, RequestOptions.DEFAULT);
        System.out.println(delete);
    }
```

### 修改文档

```java
// 方式一:全量更新。再次写入id一样的文档，就会删除旧文档，添加新文档
// 方式二:局部更新。只更新部分字段，我们演示方式二

private void updateDocument() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        UpdateRequest request = new UpdateRequest("hotel1", "UwLk_I8BUVoIbccrO-Vg");
        request.doc("email", "88@gmail.com",
                "userName", "李四"
        );
        UpdateResponse update = restHighLevelClient.update(request, RequestOptions.DEFAULT);
        System.out.println(update);
    }
```

### 批量导入文档

```java
private void saveBatchDocument() throws IOException {
        RestHighLevelClient restHighLevelClient = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://127.0.0.1:9200")));
        BulkRequest bulkRequest = new BulkRequest();
        for (int i = 0; i < 100000; i++) {
            User user = new User();
            user.setEmail(UUID.randomUUID().toString());
            user.setIp(UUID.randomUUID().toString());
            user.setUserInfo(UUID.randomUUID().toString());
            user.setUserName(UUID.randomUUID().toString());
            bulkRequest.add(new IndexRequest("hotel1").source(new Gson().toJson(user),XContentType.JSON));
        }
        BulkResponse bulk = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println(bulk);
    }
```



## DSL查询语法

### DSL Query的分类

Elasticsearch提供了基于JSON的DSL(Domain specific Language)来定义查询。常见的查询类型包括:

- 查询所有:查询出所有数据，一般测试用。例如:match_all
- 全文检索(full text)查询:利用分词器对用户输入内容分词，然后去倒排索引库中匹配。例如:
  - match_query 
  - multy_match_query
- 精确查询:根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段。例如:
  - ids
  - range
  - term
- 地理(geo)查询:根据经纬度查询。例如:
  - geo_distance
  - geo_bounding_box
- 复合(compound)查询:复合查询可以将上述各种查询条件组合起来，合并查询条件。例如:
  - bool
  - function_score

​	基本语法：

```
	POST /indexName/_search
	// 查询所有
	{
  "query":{
    "match_all":{
      
    }
  }
}
```

#### 全文检索查询

全文检索查询，会对用户输入内容分词，常用于搜索框搜索:

**match查询：**全文检索插查询的一种，会对用户输入的内容分词，然后去倒排索引库检索，语法：

```json
POST /hotel1/_search
{
  "query": {
    "match": {
      "userInfo": "今天的天气"
    }
  }
}
```

multi_macth：与match查询类似，只不过允许同时查询多个字段，语法:

```json
{
  "query": {
    "multi_match": {
      "query": "今天的天气",
      "fields":["userInfo","email"]
    }
  }
}
```

> **match和multi_match的区别是什么?**
>
> match：根据一个字段查询
> multi_match：根据多个字段查询，参与查询字段越多，查询性能越差

#### **精确查询**

> 建议使用copy to 类似mysql聚合索引的感觉

精确查询一般是查找keyword、数值、日期、boolean等类型字段，所以不会分词

- term：根据词条精确值查询
- range：根据值的范围查询

```json
POST /index0601/_search
## TERM查询
{
  "query": {
    "term": {
      "email": {
        "value": "2829190@qq.com"
      }
    }
  }
}
```

> term查询:根据词条精确匹配，一般搜索keyword类型、数值类型、布尔类型、日期类型字段

```json
POST /index0601/_search
## RANG查询
{
  "query": {
    "range": {
      "date": {
        "gte": "1718001958249",
        "lte":"1718001958249"
      }
    }
  }
}
```

> 范围查询:根据数值范围查询，可以是数值、日期的范围



#### 地理查询

根据经纬度查询。常见的使用场景包括：

- 携程:搜索我附近的酒店
- 滴滴:搜索我附近的出租车
- 微信:搜索我附近的人

**geo_bounding_box**:查询geo point值落在某个**矩形范围**的所有文档：

```json
POST /user_index/_search

{
  "query": {
    "geo_bounding_box": {
      "location": {
        "top_left": {
          "lat": 40,
          "lon": -70
        },
        "bottom_right": {
          "lat": 30,
          "lon": -80
        }
      }
    }
  }
}
```

**geo_distance**：查询到指定中心点**小于**某个距离值的所有文档（一个圆）

```json
POST /indexName/_search

{
  "query": {
    "geo_distance": {
      "distance": "20000km",  // 指定距离，可以是 km、miles、yd、ft 等
      "location": {
        "lat": 20.0,  // 指定纬度
        "lon": -70.0  // 指定经度
      }
    }
  }
}
```



#### 复合查询

复合(compound)查询:复合查询可以将其它简单查询组合起来，实现更复杂的搜索逻辑，例如:

- fuction_score:算分函数查询，可以控制文档相关性算分，控制文档排名。例如百度竞价

```json
"max_score": 0.18232156,
    "hits": [
      {
        "_index": "user_index",
        "_type": "_doc",
        "_id": "NgwjAZABUVoIbccrLhId",
        
        "_score": 0.18232156,
        
        "_source": {
          "userName": "名字呀1ahsahs",
          "info": "今天的天气数字是1hahhahah",
          "email": "1829190@qq.com",
          "ip": "112.2.1.1",
          "date": 1718006000782,
          "location": "21.41,31.21"
        }
      },
```

<img src="/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/8.png" alt="分值计算" style="zoom:50%;" />

**相关性算法**

当我们利用**match**查询时，文档结果会根据与搜索词条的关联度打分(score)，返回结果时按照分值降序排列。

<img src="/Users/gaoshang/IdeaProjects/JavaStudy/NoSQL/ES/img/9.png" alt="image-20240610155649782" style="zoom:50%;" />

> elasticsearch中的相关性打分算法是什么?
>
> - TF-IDF：在elasticsearch5.0之前，会随着词频增加而越来越大
> - BM25：在elasticsearch5.0之后，会随着词频增加而增大，但增长曲线会趋于水平



Function Score Query

使用 function score query，可以修改文档的相关性算分(query score)，根据新得到的算分排序。

```json
POST /indexName/_search

{
  "query": {
    "function_score": {
      "query": { // 原始查询条件，搜索文档并根据相关性打分(query score)
        "match": {
          "info": "今天"
        }
      },
      "functions": [
        {
          "filter": {// 过滤条件，符合条件的文档才会被重新算分
            "term": {
              "id": "NgwjAZABUVoIbccrLhId"
            }
          },
          "weight": "20" // 算分函数 见下面👇
        }
      ],
      "boost_mode": "multiply" // 加权模式 见下面👇
    }
  }
}
```

> **weight**
>
> 算分函数，算分函数的结果称为functionscore ，将来会与query score运算，得到新算分，
>
> 常见的算分函数有:
>
> - weight:给一个常量值，作为函数结果(function score)
> - field_value_factor:用文档中的某个字段值作为函数结果
> - random score:随机生成一个值，作为函数结果
> - script score:自定义计算公式，公式结果作为函数结果

> boost_mode
>
> 加权模式，定义function score与query score的运算方式，包括:· 
>
> - multiply：两者相乘。默认就是这个
> - replace：用function score 替换 query score
> - 其它：sum、avg、max、min

> function score query定义的三要素是什么?
> 过滤条件:哪些文档要加分
> 算分函数:如何计算function score。
> 加权方式:function score 与 query score如何运算

**符合查询 Boolean Query**

布尔查询是一个或多个查询子句的组合。子查询的组合方式有

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must not：必须不匹配，不参与算分，类似“非”
- filter：必须匹配，不参与算分

```
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ip": "112.2.1.1"
          }
        }
      ],
      "should": [
        {
          "term": {
            "ip": "112.2.1.1"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "date": {
              "gte": "1718006000782"
            }
          }
        }
      ],
      "filter": []
    }
  }
}
```



> **bool查询有几种逻辑关系?**
>
> - must:必须匹配的条件，可以理解为“与”
> - should:选择性匹配的条件，可以理解为“或”
> - must not:必须不匹配的条件，不参与打分
> - filter:必须匹配的条件，不参与打允

IP是指定IP，或者IP为这个IP，时间不大于1718006000782，过滤在这个经纬度1000000km范围内的数据

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "ip": "112.2.1.1"
          }
        }
      ],
      "should": [
        {
          "term": {
            "ip": "112.2.1.1"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "date": {
              "gte": "1718006000782"
            }
          }
        }
      ],
      "filter": [
        {
          "geo_distance": {
            "distance": "1000000km",
            "location": {
              "lat": 20,
              "lon": -70
            }
          }
        }
      ]
    }
  }
}
```









