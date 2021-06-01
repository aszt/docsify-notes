# ElasticSearch

> Elasticsearch 是一个分布式、RESTful 风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。

# 1. 概述
Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎，无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。 但是，Lucene只是一个库。想要发挥其强大的作用，你需使用Java并要将其集成到你的应用中。Lucene非常复杂，你需要深入的了解检索相关知识来理解它是如何工作的。 Elasticsearch也是使用Java编写并使用Lucene来建立索引并实现搜索功能，但是它的目的是通过简单连贯的RESTful API让全文搜索变得简单并隐藏Lucene的复杂性。 不过，Elasticsearch不仅仅是Lucene和全文搜索引擎，它还提供：

- 分布式的实时文件存储，每个字段都被索引并可被搜索
- 实时分析的分布式搜索引擎
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

而且，所有的这些功能被集成到一台服务器，你的应用可以通过简单的RESTful API、各种语言的客户端甚至命令行与之交互。上手Elasticsearch非常简单，它提供了许多合理的缺省值，并对初学者隐藏了复杂的搜索引擎理论。它开箱即用（安装即可使用），只需很少的学习既可在生产环境中使用。Elasticsearch在Apache 2 license下许可使用，可以免费下载、使用和修改。 随着知识的积累，你可以根据不同的问题领域定制Elasticsearch的高级特性，这一切都是可配置的，并且配置非常灵活。

# 2. 核心概念
1. 索引（inxde）

  

  ES 存储数据的地方，可以理解成关系型数据库中的数据库概念

2. 映射（mapping）

  

  mapping 定义了每个字段的类型、字段所使用的分词器等。相当于关系型数据库中的表结构

3. 文档（document）

  

  ES 中的最小数据单元，常以 json 显示。一个 document 相当于关系型数据库中的一行数据

4. 倒排索引

  

  一个倒排索引由文档中所有不重复的列表构成，对于其中每个词，对应一个包含它们的文档 id 列表

5. 类型（type）

  

  一种 type 就像一类表。如用户表、角色表等。在 ES7.X 默认 type 为 _doc

  ES5.X 中一个 index 可以有多种  type

  ES6.X 中一个 index 只能有一种  type

  ES7.X 以后，将逐步移除  type  这个概念，现在的操作已经不再使用，默认 _doc

6. 分词器

  

  将一段文本，按照一定逻辑，分析成多个词语的一种工具
  ES 内置分词器对中文很不友好，处理方式为：一个字一个词

# 3. 准备工作
## 3.1 maven工程创建
**1. 创建 Maven 工程，引入 jar 包**
```xml
<dependency>
	<groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.4.0</version>
</dependency>
<dependency>
	<groupId>org.elasticsearch.client</groupId>
	<artifactId>elasticsearch-rest-client</artifactId>
	<version>7.4.0</version>
</dependency>
<dependency>
	<groupId>org.elasticsearch</groupId>
	<artifactId>elasticsearch</artifactId>
	<version>7.4.0</version>
</dependency>
```
**2.创建 ES 客户端**
```java
// 创建ES客户端对象
RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
	new HttpHost(
			"47.116.70.154",
			9200,
			"http"
	)
));
```
**3.创建 ES 客户端2.0**
```java
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "elasticsearch")
public class ElasticSearchConfig {

    private String host;
    private int port;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }

    @Bean
    public RestHighLevelClient client() {
        return new RestHighLevelClient(RestClient.builder(
                new HttpHost(
                        host,
                        port,
                        "http"
                )
        ));
    }

}
```

# 4 索引(index)

## 4.1 postman 方式
- 增加索引--->
  PUT：http://47.116.70.154:9200/goods_index

- 查询索引信息--->
  GET：http://47.116.70.154:9200/goods_index

- 查询多个索引--->
  GET：http://47.116.70.154:9200/goods_index,goods_index2

- 查询所有索引--->
  GET：http://47.116.70.154:9200/_all

- 删除索引--->
  DELETE：http://47.116.70.154:9200/goods_index

- 关闭索引--->
  POST：http://47.116.70.154:9200/goods_index/_close

- 打开索引--->
POST：http://47.116.70.154:9200/goods_index/_open

## 4.2 kibana 方式
- 创建索引：
PUT person
- 查询索引：
GET person
- 删除索引：
DELETE person

## 4.3 JavaAPI
### 4.3.1 创建索引
```java
@Test
public void addIndex() throws IOException {
	// 1、使用client获取操作索引的对象
	IndicesClient indicesClient = client.indices();

	// 2、具体操作，获取返回值
	CreateIndexRequest createRequest = new CreateIndexRequest("itheima");
	CreateIndexResponse response = indicesClient.create(createRequest, RequestOptions.DEFAULT);

	// 3、根据返回值判断结果
	System.out.println(response.isAcknowledged());
}
```
### 4.3.2 查询索引
```java
@Test
public void queryIndex() throws IOException {
    IndicesClient indices = client.indices();
    GetIndexRequest getRequest = new GetIndexRequest("itcast");
    GetIndexResponse response = indices.get(getRequest, RequestOptions.DEFAULT);
    // 获取结果
    Map<String, MappingMetaData> mappings = response.getMappings();
    for (String key : mappings.keySet()) {
        System.out.println(key + ":" + mappings.get(key).getSourceAsMap());
    }
}
```
### 4.3.3 删除索引
```java
@Test
public void deleteIndex() throws IOException {
    IndicesClient indices = client.indices();
    DeleteIndexRequest deleteRequest = new DeleteIndexRequest("itheima");
    AcknowledgedResponse response = indices.delete(deleteRequest, RequestOptions.DEFAULT);
    System.out.println(response.isAcknowledged());
}
```

### 4.3.4 判断索引是否存在
```java
@Test
public void existIndex() throws IOException {
    IndicesClient indices = client.indices();
    GetIndexRequest getIndexRequest = new GetIndexRequest("itcast");
    boolean exists = indices.exists(getIndexRequest, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

# 5 映射(mapping)
(1). 简单数据类型
1. 字符串
   - text：会分词，不支持聚合 
   - keyword: 不会分词，将全部内容作为一个词条，支持聚合
2. 数值
   - long
   - integer
   - short
   - byte
   - double
   - float
   - half_float
   - scaled_float
3. 布尔
   - boolean
4. 二进制
   - binary
5. 范围类型
   - integer_range
   - float_range
   - long_range
   - double_range
   - date_range
6. 日期
   - date


(2). 复杂数据类型
1. 数组：[]
2. 对象：{}

## 5.1 添加映射
### 5.1.1 脚本命令
(1) 第一种：先创建索引再添加映射
```kibana
# 创建索引
PUT person

# 添加映射
PUT person/_mapping
{
  "properties":{
    "name":{
      "type":"keyword"
    },
    "age":{
      "type":"integer"
    }
  }
}
```
(2) 第二种：创建索引并添加映射
```kibana
PUT person
{
  "mappings": {
    "properties": {
      "name":{
        "type":"keyword"
      },
      "age":{
        "type":"integer"
      }
    }
  }
}
```
(3)添加字段
```kibana
PUT person/_mapping
{
  "properties": {
    "address":{
      "type":"text"
    }
  }
}
```

### 5.1.2 JavaAPI
添加索引及映射
```java
@Test
public void addIndexAndMapping() throws IOException {
	// 1、使用client获取操作索引的对象
	IndicesClient indicesClient = client.indices();

	// 2、具体操作，获取返回值
	CreateIndexRequest createRequest = new CreateIndexRequest("itcast");

	// 2.1设置mappings
	String mapping = "{\n" +
			"      \"properties\" : {\n" +
            "        \"address\" : {\n" +
            "          \"type\" : \"text\",\n" +
            "          \"analyzer\" : \"ik_max_word\"\n" +
            "        },\n" +
            "        \"name\" : {\n" +
            "          \"type\" : \"keyword\"\n" +
            "        }\n" +
            "      }\n" +
            "    }";
	createRequest.mapping(mapping, XContentType.JSON);

	CreateIndexResponse response = indicesClient.create(createRequest, RequestOptions.DEFAULT);

	// 3、根据返回值判断结果
	System.out.println(response.isAcknowledged());
}
```
## 5.2 查询映射
### 5.2.1 脚本命令
```kibana
GET person/_mapping
```
### 5.2.2 JavaAPI
```java
@Test
public void queryIndex() throws IOException {
	IndicesClient indices = client.indices();
    GetIndexRequest getRequest = new GetIndexRequest("itcast");
    GetIndexResponse response = indices.get(getRequest, RequestOptions.DEFAULT);
    // 获取结果
    Map<String, MappingMetaData> mappings = response.getMappings();
    for (String key : mappings.keySet()) {
        System.out.println(key + ":" + mappings.get(key).getSourceAsMap());
    }
}
```

# 6 文档(document)
## 6.1 添加文档
### 6.1.1 脚本命令
```kibana
# 添加文档，指定id(PUT、POST都可)
PUT person/_doc/1
{
  "name":"张三",
  "age":20,
  "address":"北京海淀区"
}

# 添加文档，不指定id(只能POST)
POST person/_doc
{
  "name":"李四",
  "age":22,
  "address":"上海浦东新区"
}

```
### 6.1.2 JavaAPI
```java
/**
 * 添加文档，使用map作为数据
 */
@Test
public void addDoc() throws IOException {
    // 数据对象，Map
    Map data = new HashMap();
    data.put("name","大胖");
    data.put("address","北京昌平");
    // 1.获取操作文档的对象
    IndexRequest request = new IndexRequest("itcast").id("1").source(data);
    // 添加数据，获取结果
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);
    // 打印响应结果
    System.out.println(response.getId());
}

/**
 * 添加文档，使用对象作为数据
 */
@Test
public void addDoc2() throws IOException {
    // 数据对象，javaObject
    Person person = new Person("2","小胖",15,"上海浦东新区");
    // 将对象转为json
    String data = JSON.toJSONString(person);

    // 1.获取操作文档的对象
    IndexRequest request = new IndexRequest("itcast").id(person.getId()).source(data,XContentType.JSON);
    // 添加数据，获取结果
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);

    // 打印响应结果
    System.out.println(response.getId());
}
```

## 6.2 查询文档
### 6.2.1 脚本命令
```kibana
# 根据id查询文档
GET person/_doc/1

# 查询所有文档
GET person/_search
```

### 6.2.2 JavaAPI
```java
@Test
public void findDOcById() throws IOException {
    GetRequest getRequest = new GetRequest("itcast","2");
    // getRequest.id("2");
    GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);
    System.out.println(response.getSourceAsString());
}
```

## 6.3 修改文档
### 6.3.1 脚本命令
```kibana
# 修改文档(id存在则修改，否则添加)
PUT person/_doc/1
{
  "name":"张三",
  "age":20,
  "address":"北京海淀区"
}
```

### 6.3.2 JavaAPI
```java
/**
 * 修改文档：添加文档时，如果id存在则修改，id不存在则添加
 */
@Test
public void updateDoc() throws IOException {
    // 数据对象，javaObject
    Person person = new Person("2","小胖22222",15,"上海浦东新区");
    // 将对象转为json
    String data = JSON.toJSONString(person);

    // 1.获取操作文档的对象
    IndexRequest request = new IndexRequest("itcast").id(person.getId()).source(data,XContentType.JSON);
    // 添加数据，获取结果
    IndexResponse response = client.index(request, RequestOptions.DEFAULT);

    // 打印响应结果
    System.out.println(response.getId());
}
```

## 6.4 删除文档
### 6.4.1 脚本命令
```kibana
# 删除文档
DELETE person/_doc/QiUfqXkB4Xml28ZmYlSP
```

### 6.4.2 JavaAPI
```java
@Test
public void delDoc() throws IOException {
    DeleteRequest deleteRequest = new DeleteRequest("itcast","2");
    DeleteResponse response = client.delete(deleteRequest, RequestOptions.DEFAULT);
    System.out.println(response.getId());
}
```

# 7 IK分词器
## 7.1 概述
- IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包
- 是一个基于Maven构建的项目
- 具有60万字/秒的高速处理能力
- 支持用户词典扩展定义
- 下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

## 7.2 分词模式
IK分词器有两种分词模式：ik_max_word（细粒度拆分，来回分词）和ik_smart（粗粒度）模式

### 7.2.1 粗粒度
```kibana

# ik分词器，粗粒度分词
GET _analyze
{
  "analyzer": "ik_smart",
  "text":"我爱北京天安门"
}


{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "北京",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "天安门",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    }
  ]
}
```

### 7.2.2 细粒度
```kibana
# ik分词器，细粒度分词
GET _analyze
{
  "analyzer": "ik_max_word",
  "text":"我爱北京天安门"
}

{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "北京",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "天安门",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "天安",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "门",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 5
    }
  ]
}
```
# 8 查询文档
**创建索引，添加映射,指定使用ik分词器**
```kibana
# es 默认使用的分词器是standard,一个字一个词
PUT person
{
  "mappings": {
    "properties": {
      "name":{
        "type":"keyword"
      },
      "address":{
        "type":"text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```

## 8.1 词条查询(term)
词条查询不会分析查询条件，只有当词条和查询字符串完全匹配时才匹配搜索

遇到映射字段是Text类型，分词时可能会命中
```kibana
GET person/_search
{
  "query": {
    "term": {
      "address": {
        "value": "北京"
      }
    }
  }
}
```

## 8.2 全文查询(match)
全文查询会分析查询条件，先将查询条件进行分词，然后查询，求并集
```kibana
GET person/_search
{
  "query": {
    "match": {
      "address": "北京昌平"
    }
  }
}
```

# 9 Bulk 批量操作
将文档的增删改查一系列操作，通过一次请求全部做完，减少网络传输次数。

## 9.1 脚本命令
```kibana
# 批量操作
# 1.删除1号记录
# 2.添加8号记录
# 3.修改2号记录 名称为“二号”
POST _bulk
{"delete":{"_index":"person","_id":"1"}}
{"create":{"_index":"person","_id":"8"}}
{"name":"八号","address":"北京"}
{"update":{"_index":"person","_id":"2"}}
{"doc":{"name":"二号"}}

```

## 9.2 JavaAPI
```java
@Test
public void testBulk() throws IOException {
    // 创建BulkRequest对象，整合所有操作
    BulkRequest bulkRequest = new BulkRequest();

    /**
     * # 1.删除3号记录
     * # 2.添加6号记录
     * # 3.修改8号记录 名称为“八号”
     */
    DeleteRequest deleteRequest = new DeleteRequest("person", "3");
    bulkRequest.add(deleteRequest);

    Map map = new HashMap();
    map.put("name", "六号");
    IndexRequest indexRequest = new IndexRequest("person").id("6").source(map);
    bulkRequest.add(indexRequest);

    Map map2 = new HashMap();
    map2.put("name", "八号");
    UpdateRequest updateRequest = new UpdateRequest("person", "8").doc(map2);
    bulkRequest.add(updateRequest);

    // 执行批量操作
    BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
    RestStatus status = response.status();
    System.out.println(status);
}
```

# 10 各种查询
## 10.1 准备工作
### 10.1.1 创建表结构
```kibana
# 创建索引
PUT goods
{
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer":"ik_smart"
      },
      "price":{
        "type":"double"
      },
      "createTime":{
        "type":"date"
      },
      "categoryName":{
        "type":"keyword"
      },
      "brandName":{
        "type":"keyword"
      },
      "spec":{
        "type":"object"
      },
      "saleNum":{
        "type":"integer"
      },
      "stock":{
        "type":"integer"
      }
    }
  }
}
```
### 10.1.2 添加测试记录
```kibana
# 添加记录
POST goods/_doc/1
{
  "title":"小米手机",
  "price":1000,
  "createTime":"2019-12-01",
  "categoryName":"手机",
  "brandName":"小米",
  "saleNum":3000,
  "stock":10000,
  "spec":{
    "网络制式":"移动4G",
    "屏幕尺寸":"4.5"
  }
}
```
### 10.1.3 批量导入数据
```java
public class Goods {

    private Long id;
    private String title;
    private double price;
    private int stock;
    private int saleNum;
    private Date createTime;
    private String categoryName;
    private String brandName;
    private Map spec;

    // 接收数据库的信息
    // 在转换JSON时，忽略该字段
    @JSONField(serialize = false)
    private String specStr;
}
```
```java
@Test
public void importData() throws IOException {
    // 1.查询所有数据
    List<Goods> goodsList = goodsMapper.findAll();
    // System.out.println(goodsList.size());

    // 2.bulk导入
    BulkRequest bulkRequest = new BulkRequest();

    // 2.1 循环goodsList,创建IndexRequest添加数据
    for (Goods goods : goodsList) {
        // 2.2 设置spec规格信息 Map的数据 specStr:{}
        goods.setSpec(JSON.parseObject(goods.getSpecStr(), Map.class));
        IndexRequest indexRequest = new IndexRequest("goods");
        indexRequest.id(goods.getId() + "").source(JSON.toJSONString(goods), XContentType.JSON);

        bulkRequest.add(indexRequest);
    }

    client.bulk(bulkRequest, RequestOptions.DEFAULT);
}
```
## 10.2 matchAll查询
查询所有文档

### 10.2.1 脚本命令
```kibana
# 默认情况下，es一次展示10条数据,通过from和size来控制分页
# took：执行时间（毫秒）
# timed_out：是否超时
# _shards：集群下分片信息
# hits：命中，total：条数，relation：操作方式
# max_score：得分
# hits：命中数据，数组


GET goods/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 100
}
```

### 10.2.2 JavaAPI
```java
/**
 * 查询所有
 * 1.matchAll
 * 2.将查询结果封装为Goods对象，装载到List中
 * 3.分页。默认显示10条
 */
@Test
public void matchAll() throws IOException {
    // 2.构建查询请求对象，指定查询索引名称
    SearchRequest searchRequest = new SearchRequest("goods");

    // 4.创建查询条件构建器SearchSourceBuilder
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 6.查询条件
    // matchAllQuery:查询所有文档
    QueryBuilder query = QueryBuilders.matchAllQuery();

    // 5.指定查询条件
    sourceBuilder.query(query);

    // 3.添加查询条件构建器
    searchRequest.source(sourceBuilder);

    // 8.添加分页信息
    sourceBuilder.from(0);
    sourceBuilder.size(100);

    // 1.查询。获取查询结果
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    // 7.获取命中对象
    SearchHits searchHits = searchResponse.getHits();

    // 7.1获取总记录数
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);

    // 7.2获取hits数据，数组
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        // 获取json字符串格式的数据
        String sourceAsString = hit.getSourceAsString();
        // 转为java对象
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }

}
```

## 10.3 term查询
不会对查询条件进行分词，适用于keyword类型数据，不会分词，查什么是什么

遇到映射字段是Text类型分词时会命中

### 10.3.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "term": {
      "categoryName": {
        "value": "平板电视"
      }
    }
  }
}
```
### 10.3.2 JavaAPI
```java
/**
 * termQuery:词条查询
 */
@Test
public void testTermQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    QueryBuilder query = QueryBuilders.termQuery("title", "华为");
    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```


## 10.4 match查询
会对查询条件进行分词

然后将分词后的查询条件和词条进行等值匹配

默认取并集（OR）

注意：遇到单个无法分词查询条件时无法查询到结果（例如：华为，查华，存储时不会分词，匹配不到）

### 10.4.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "match": {
      "title": "华为TCL"
    }
  }
}

GET goods/_search
{
  "query": {
    "match": {
      "title": {
        "query": "华为TCL"
        , "operator": "and"
      }
    }
  }
}
```
### 10.4.2 JavaAPI
```java
/**
 * matchQuery:词条分词查询
 */
@Test
public void testMatchQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    MatchQueryBuilder query = QueryBuilders.matchQuery("title", "华为TCL");
    // 取交集
    query.operator(Operator.AND);
    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

## 10.5 模糊查询
### 10.5.1 wildcard (模糊查询)
会对查询条件进行分词。还可以使用通配符
- ？（任意单个字符）
- *（0个或多个字符）

**注：前面千万不要写通配符，将会进行全部倒排索引扫描，建立索引会失效，性能不高**

#### 10.5.1.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "wildcard": {
      "title": {
        "value": "华*"
      }
    }
  }
}
```

#### 10.5.1.2 JavaAPI
```java
/**
 * 模糊查询:WildcardQuery
 */
@Test
public void testWildcardQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    WildcardQueryBuilder query = QueryBuilders.wildcardQuery("title", "华*");
    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

### 10.5.2 regexp (正则查询)
取决于正则表达式的性能，正则性能不高

#### 10.5.2.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "regexp": {
      "title": "\\w+(.)*"
    }
  }
}
```

#### 10.5.2.2 JavaAPI
```java
@Test
public void testRegexpQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    RegexpQueryBuilder query = QueryBuilders.regexpQuery("title", "\\\\w+(.)*");
    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

### 10.5.3 prefix (前缀查询)
一般用于keyword类型，对keyword支持较好，text分词会有错误数据

#### 10.5.3.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "prefix": {
      "brandName": {
        "value": "TC"
      }
    }
  }
}
```

#### 10.5.3.2 JavaAPI
```java
@Test
public void testPrefixQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    PrefixQueryBuilder query = QueryBuilders.prefixQuery("brandName", "TC");
    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

## 10.6 范围查询 (range)
查询指定字段在指定范围内包含值
### 10.6.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 500,
        "lte": 600
      }
    }
  },
  "sort": [
    {
      "price": {
        "order": "asc"
      }
    }
  ]
}
```

### 10.6.2 JavaAPI
```java
@Test
public void testRangeQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 范围查询
    RangeQueryBuilder query = QueryBuilders.rangeQuery("price");

    // 指定下线(gte:大于等于)
    query.gte(500);

    // 指定上线(lte:小于等于)
    query.lte(600);

    sourceBuilder.query(query);

    // 排序
    sourceBuilder.sort("price", SortOrder.DESC);

    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```


## 10.7 queryString 查询
会对查询条件进行分词

然后将分词后的查询条件和词条进行等值匹配

默认取并集（OR）

可以指定多个查询字段（与match唯一不同的）

### 10.7.1 脚本命令
```kibana
GET goods/_search
{
  "query": {
    "query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND TCL"
    }
  }
}

# simple_query_string：不支持连接符
GET goods/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["title","categoryName","brandName"], 
      "query": "华为 AND TCL"
    }
  }
}
```

### 10.7.2 JavaAPI
```java
@Test
public void testQueryString() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    QueryStringQueryBuilder query = QueryBuilders.queryStringQuery("华为手机").field("title").field("categoryName").field("brandName").defaultOperator(Operator.AND);


    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

## 10.8 布尔查询
boolQuery：对多个查询条件连接。连接方式：

- must（and）：条件必须成立
- must_not（not）：条件必须不成立
- should（or）：条件可以成立
- filter：条件必须成立，性能比must高，不会计算得分
- 
### 10.8.1 脚本命令
```kibana
# 写法一：
GET goods/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "brandName": {
              "value": "华为"
              }
            }
          },
          {
          "match": {
            "title": "联通"
          }
        }
      ]
    }
  }
}
```

```kibana
# 写法二：优化，第一次使用must，后面就可以用filter查询提高性能
GET goods/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "brandName": {
              "value": "华为"
              }
            }
          }
      ],
      "filter": {
        "term": {
          "title": "联通"
        }
      }
    }
  }
}
```

### 10.8.2 JavaAPI
```java
@Test
public void testBoolQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    BoolQueryBuilder query = QueryBuilders.boolQuery();

    query.must(QueryBuilders.termQuery("brandName","华为"));
    query.filter(QueryBuilders.matchQuery("title","联通"));
    query.filter(QueryBuilders.rangeQuery("price").gte(500).lte(600));


    sourceBuilder.query(query);
    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);
        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }
}
```

## 10.9 聚合查询
### 10.9.1 指标聚合
相当于MySQL的聚合函数。max、min、avg、sum等
#### 10.9.1.1 脚本命令
```kibana
# 指标聚合：聚合函数（查最贵华为产品）
GET goods/_search
{
  "query": {
    "match": {
      "title": "华为"
    }
  },
  "aggs": {
    "max_price": {
      "max": {
        "field": "price"
      }
    }
  }
}
```

### 10.9.2 桶聚合
相当于MySQL的group by操作。不要对text类型的数据进行分组，会失败
#### 10.9.2.1 脚本命令
```kibana
# 桶聚合：分组（查所有手机品牌）
GET goods/_search
{
  "query": {
    "match": {
      "title": "手机"
    }
  },
  "aggs": {
    "goods_brands": {
      "terms": {
        "field": "brandName",
        "size": 100
      }
    }
  }
}
```
#### 10.9.2.2 JavaAPI
```java
/**
 * 聚合查询：桶聚合，分组查询
 * 1.查询title包含手机的数据
 * 2。查询品牌列表
 */
@Test
public void testAggQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    MatchQueryBuilder query = QueryBuilders.matchQuery("title", "手机");
    sourceBuilder.query(query);

    // goods_brands:返回数据名称，brandName:分组字段名
    AggregationBuilder agg = AggregationBuilders.terms("goods_brands").field("brandName").size(100);
    sourceBuilder.aggregation(agg);

    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

    // 获取聚合结果
    Aggregations aggregations = searchResponse.getAggregations();
    Terms goods_brands = (Terms)aggregations.asMap().get("goods_brands");
    List<? extends Terms.Bucket> buckets = goods_brands.getBuckets();
    List brands = new ArrayList();
    for (Terms.Bucket bucket : buckets) {
        Object key = bucket.getKey();
        brands.add(key);
    }
    for (Object brand : brands) {
        System.out.println(brand);
    }

}
```
## 10.10 高亮查询
高亮三要素，默认 em 标签：
- 高亮字段
- 前缀
- 后缀

### 10.10.1 脚本命令
```java
GET goods/_search
{
  "query": {
    "match": {
      "title": "华为"
    }
  },
  "highlight": {
    "fields": {
      "title": {
        "pre_tags": "<aa>",
        "post_tags": "</aa>"
      }
    }
  }
}
```

### 10.10.2 JavaAPI
```java
/**
 * 高亮查询：
 * 1.设置高亮
 *      * 高亮字段
 *      * 前缀
 *      * 后缀
 * 2.将高亮了的字段数据，替换原有数据
 */
@Test
public void testHighLightQuery() throws IOException {
    SearchRequest searchRequest = new SearchRequest("goods");
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

    // 1.查询title包含手机的数据
    MatchQueryBuilder query = QueryBuilders.matchQuery("title", "华为");
    sourceBuilder.query(query);

    // 设置高亮
    HighlightBuilder highLightBuild = new HighlightBuilder();
    // 设置三要素
    highLightBuild.field("title");
    highLightBuild.preTags("<aa>");
    highLightBuild.postTags("</aa>");
    sourceBuilder.highlighter(highLightBuild);

    searchRequest.source(sourceBuilder);
    SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);
    SearchHits searchHits = searchResponse.getHits();
    // 获取记录数
    long value = searchHits.getTotalHits().value;
    System.out.println("总记录数：" + value);
    SearchHit[] hits = searchHits.getHits();
    List<Goods> goodsList = new ArrayList<>();
    for (SearchHit hit : hits) {
        String sourceAsString = hit.getSourceAsString();

        // 转为java
        Goods goods = JSON.parseObject(sourceAsString, Goods.class);

        // 获取高量结果，替换goods中的title
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        HighlightField highlightField = highlightFields.get("title");
        Text[] fragments = highlightField.getFragments();

        // 替换
        goods.setTitle(fragments[0].toString());

        goodsList.add(goods);
    }
    for (Goods goods : goodsList) {
        System.out.println(goods);
    }

}
```

# 11 重建索引
随着业务需求的变更，索引的结构可能发生改变

Elasticsearch的索引一旦创建，只允许添加字段，不允许改变字段。因为改变字段，需要重建倒排索引，影响内部缓存结构，性能太低。

那么此时，就需要重建一个新的索引，并将原有索引的数据导入到新索引中。

1. 问题分析

```kibana
# 1.新建student_index_v1。索引名称必须全部小写
PUT student_index_v1
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "date"
      }
    }
  }
}

# 2.添加数据
PUT student_index_v1/_doc/1
{
  "birthday":"1999-11-11"
}

# 3.业务变更了，需要改变birthday字段的类型为text
# 修改失败
PUT student_index_v1/_doc/1
{
  "birthday":"1999年11月11日"
}
```
2. 数据拷贝

```kibana
# 1.创建新的索引 student_index_v2
# 2.将student_index_v1 数据拷贝到 student_index_v2

PUT student_index_v2
{
  "mappings": {
    "properties": {
      "birthday":{
        "type": "text"
      }
    }
  }
}

# _reindex 拷贝数据
POST _reindex
{
  "source": {
    "index": "student_index_v1"
  },
  "dest": {
    "index": "student_index_v2"
  }
}

GET student_index_v2/_search

PUT student_index_v2/_doc/2
{
  "birthday":"1999年11月11日"
}
```
3. 建立别名

思考：现在java代码中操作es，还是使用的student_index_v1老的索引名称

- 改代码（不推荐）
- 索引别名（推荐）

```kibana
# 步骤：
# 0. 先删除student_index_v1
# 1. 给student_index_v2起个别名 student_index_v1

DELETE student_index_v1

POST student_index_v2/_alias/student_index_v1

GET student_index_v1/_search
```
