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

# 3. 脚本及API操作
## 3.1 API 准备工作
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

## 3.2 索引(index)

### 3.2.1 postman 方式
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

### 3.2.2 kibana 方式
- 创建索引：
PUT person
- 查询索引：
GET person
- 删除索引：
DELETE person

### 3.2.3 JavaAPI
#### 3.2.3.1 创建索引
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
#### 3.2.3.2 查询索引
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
#### 3.2.3.3 删除索引
```java
@Test
public void deleteIndex() throws IOException {
    IndicesClient indices = client.indices();
    DeleteIndexRequest deleteRequest = new DeleteIndexRequest("itheima");
    AcknowledgedResponse response = indices.delete(deleteRequest, RequestOptions.DEFAULT);
    System.out.println(response.isAcknowledged());
}
```

#### 3.2.3.3 判断索引是否存在
```java
@Test
public void existIndex() throws IOException {
    IndicesClient indices = client.indices();
    GetIndexRequest getIndexRequest = new GetIndexRequest("itcast");
    boolean exists = indices.exists(getIndexRequest, RequestOptions.DEFAULT);
    System.out.println(exists);
}
```

## 3.3 映射(mapping)
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

### 3.3.1 添加映射
#### 3.3.1.1 脚本命令
(1) 第一种：先创建索引再添加映射
```kibanna
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
```kibanna
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

#### 3.3.1.2 JavaAPI
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
### 3.3.2 查询映射
#### 3.3.2.1 脚本命令
```kibanna
GET person/_mapping
```
#### 3.3.2.2 JavaAPI
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

## 3.4 文档(document)