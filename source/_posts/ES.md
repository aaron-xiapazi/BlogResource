---
title: Elasticsearch学习
categories: 笔记
tags: [es]
date: 2020-02-09
---

## Elasticsearch 实现多字段高亮检索、文件检索、数据同步

### Elasticsearch知识：

- **ES概述简介**

  *Elasticsearch是一个分布式的基于REST接口的为云而设计的搜索引擎。*

  Elasticsearch不仅仅是Lucene和全文搜索引擎，它还提供：

  - 分布式的实时文件存储，每个字段都被索引并可被搜索
  - 实时分析的分布式搜索引擎
  - 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

  为什么选择ES做搜索引擎：

  - 很简便的横向扩容，既能由数百台到万台机器搭建满足PB级的快速搜索，也能搭建单机版服务小公司。
  - 功能点多但使用比较简便，开箱即用，性能优化比较简单
  - 生态圈丰富，社区活跃，apache 2.0开源。适配多种工具。提供java api和restful接口，ELK架构处理海量日志，丰富的插件(分词器、search-gurad等安全插件)
  
- **ES 相关术语概念**

  1.  **Cluster**:Cluster也就是集群的意思。Elasticsearch集群由一个或多个节点组成，可通过其集群名称进行标识。通常这个Cluster 的名字是可以在Elasticsearch里的配置文件中设置的。

  2.  **node**:单个Elasticsearch实例。 在大多数环境中，每个节点都在单独的盒子或虚拟机上运行。一个集群由一个或多个node组成。

  3.  **Document**:Elasticsearch是面向文档的，这意味着您索引或搜索的最小数据单元是文档。

  4.  **type**:类型是文档的逻辑容器，类似于表是行的容器。 

  5.  **index**  :在Elasticsearch中，索引是文档的集合。

  6.  **shard** :由于Elasticsearch是一个分布式搜索引擎，因此索引通常会拆分为分布在多个节点上的称为分片的元素。

  7.  **replica** :默认情况下，Elasticsearch为每个索引创建一个主分片和一个副本。

### ES DEMO
*说明：此例适用于elasticsearch6.4.3，其他版本尚未测试*

1. **实现目标：**
  
   - 针对于文档的标题，内容及相关属性（如标签、时间），提供输入关键字即可搜索的能力
   - 关键字在搜索结果中高亮显示
   - 搜索功能实现对于文件的检索，如word、pdf等。（使用Ingest Attachment Processor）
- ES索引中的文档与mysql数据库中的数据同步（使用logstash）
  
2. **环境要求：**

   - elasticsearch 6.4.3

     es下载地址：https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-4-3

     安装指导：https://blog.csdn.net/art_code/article/details/90499981

   - ik分词器6.4.3

     *PS：为什么选择IK分词器？支持中文；功能强大；开源；仍被维护。*

     ik下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.4.3/elasticsearch-analysis-ik-6.4.3.zip

     安装指导：https://www.cnblogs.com/chenmc/p/9525163.html

   - 其他工具：Ingest Attachment Processor、logstash

3. **工具类的使用：**

对比开源的相关es搜索工具类（如Bboss，elasticsearch-RHL等）且

根据项目需求（检索条件不复杂，仅针对于文档）。认为spring-boot-starter-data-elasticsearch（可认为spring-data官方的工具类）较适合于目前知识库的elasticsearch工具类选用。maven工程中pom文件如下

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

4. **工具类的API使用概述及配置（详情见DEMO）**

appliction.properties参数

```
## Elasticsearch配置文件（必须）
## 该配置和Elasticsearch的elasticsearch.yml中的配置信息有关
spring.data.elasticsearch.cluster-name=my-application
spring.data.elasticsearch.cluster-nodes=127.0.0.1:9300
```

创建接口继承ElasticsearchRepository，其中<Demo>为自定义的实体类

```
public interface DemoRepository extends ElasticsearchRepository<Demo,Long> {
```

在实现类中，使用根据自定义实体类创建好的接口及ES的模板类

```
@Autowired
private ItemRepository itemRepository;
@Autowired
private ElasticsearchTemplate elasticsearchTemplate;
...(具体操作见DEMO代码)
```

​      GitHub仓库地址：

5. **文件检索的实现**

   ingest attachment plugin允许Elasticsearch通过使用Apache文本提取库Tika提取通用格式（例如PPT，XLS和PDF）的文件附件。 Apache Tika工具包可从一千多种不同的文件类型（例如PPT，XLS和PDF）中检测并提取元数据和文本。 所有这些文件类型都可以通过一个界面进行解析，从而使Tika对搜索引擎索引，内容分析，翻译等有用。

   源字段必须是base64编码的二进制。 

   - **创建attachment pipeline**

     我们可以在我们的Ingest node上创建一个叫做pdfattachment的pipleline:
     
     ```
     
     PUT _ingest/pipeline/pdfattachment
     {
       "description": "Extract attachment information encoded in Base64 with UTF-8 charset",
       "processors": [
         {
           "attachment": {
             "field": "file"
           }
         }
       ]
     ```
     
   - **转换pdf文件并上传pdf文件的内容到Elasticsearch中**

   对于ingest attachment plugin来说，它的数据必须是Base64的。我们可以在网站[Base64 encoder](https://www.giftofspeed.com/base64-encoder/)来进行转换。针对我们的情况，我们直接通过脚本的方法来进行操作：

   ```
   #!/bin/bash
   encodedPdf=`cat sample.pdf | base64`
   json="{\"file\":\"${encodedPdf}\"}"
   echo "$json" > json.file
   curl -XPOST 'http://localhost:9200/pdf-test1/_doc?pipeline=pdfattachment&pretty' -H 'Content-Type: application/json' -d @json.file
   ```

6. **数据同步的实现**

   **需满足条件**

   - 在将 MySQL 中的文档写入 Elasticsearch 时，Elasticsearch 中的 "_id" 字段必须设置为 MySQL 中的 "id" 字段。这可在 MySQL 记录与 Elasticsearch 文档之间建立一个直接映射关系。如果在 MySQL 中更新了某条记录，那么将会在 Elasticsearch 中覆盖整条相关记录。请注意，在 Elasticsearch 中覆盖文档的效率与[更新操作](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/getting-started-update-documents.html)的效率一样高，因为从内部原理上来讲，更新便包括删除旧文档以及随后对全新文档进行索引。
   - 当在 MySQL 中插入或更新数据时，该条记录必须有一个包含更新或插入时间的字段。通过此字段，便可允许 Logstash 仅请求获得在轮询循环的上次迭代后编辑或插入的文档。Logstash 每次对 MySQL 进行轮询时，都会保存其从 MySQL 所读取最后一条记录的更新或插入时间。在下一次迭代时，Logstash 便知道其仅需请求获得符合下列条件的记录：更新或插入时间晚于在轮询循环中的上一次迭代中所收到的最后一条记录。

   **Logstash通过管道同步数据：**

   ```
   input {
     jdbc {
       jdbc_driver_library => "<path>/mysql-connector-java-8.0.16.jar"
       jdbc_driver_class => "com.mysql.jdbc.Driver"
       jdbc_connection_string => "jdbc:mysql://<MySQL host>:3306/es_db"
       jdbc_user => <my username>
       jdbc_password => <my password>
       jdbc_paging_enabled => true
       tracking_column => "unix_ts_in_secs"
       use_column_value => true
       tracking_column_type => "numeric"
       schedule => "*/5 * * * * *"
       statement => "SELECT *, UNIX_TIMESTAMP(modification_time) AS unix_ts_in_secs FROM es_table WHERE (UNIX_TIMESTAMP(modification_time) > :sql_last_value AND modification_time < NOW()) ORDER BY modification_time ASC"
     }
   }
   filter {
     mutate {
       copy => { "id" => "[@metadata][_id]"}
       remove_field => ["id", "@version", "unix_ts_in_secs"]
     }
   }
   output {
     # stdout { codec =>  "rubydebug"}
     elasticsearch {
         index => "rdbms_sync_idx"
         document_id => "%{[@metadata][_id]}"
     }
   ```

   

参考文档:

1. https://elasticstack.blog.csdn.net/article/details/99443042      （基础概念）
2. https://elasticstack.blog.csdn.net/article/details/102728604    （入门指南）
3. https://elasticstack.blog.csdn.net/article/details/99413578         (安装文档)
4. https://elasticstack.blog.csdn.net/article/details/104171230     （对文档搜索）
5. https://elasticstack.blog.csdn.net/article/details/103874185     （数据同步）



[示例代码](https://github.com/CreaterAlan/elasticsearch-demo)