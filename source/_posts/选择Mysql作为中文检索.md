---
title: 选择Mysql作为中文检索方案...
date: 2020-03-07 21:19:42
tags: 检索
categories: 笔记
---



### 1. 选用Elasticsearch搜索引擎作为搜索模块的实现

Elasticsearch是一个高度可伸缩的开源全文搜索和分析引擎。可以快速和接近实时地存储、搜索和分析大量数据。可以满足搜索功能的实际场景。


优点：功能强大，天生支持分布式，扩展性强，支持海量数据搜索，插件丰富，文档完备，平台已有相关Elasticsearch API，支持高亮查询，支持文件检索。

缺点：资源占用过大

### 2. MongoDB数据库的全文索引作为搜索模块的实现

MongoDB是通用功能的非Restful风格的NoSQL数据库，文档以Bson格式存储，主要用于数据存储，MongoDB 3.2+ 支持了对中文的检索

 

优点：相对于Elasticsearch更轻量，支持全文检索，部署方便，有扩容方案

缺点: 对于中文的全文检索支持不友好，暂无中文分词器

 

如图为测试MongoDB对中文全文检索的能力是否支持，因无法对中文完成分词所以无法满足搜索场景

![img](https://inventory-manager.oss-cn-beijing.aliyuncs.com/blog_image/mongodb_search.jpg) 

### 3. MySQL5.7的全文检索作为搜索模块的实现


优点：支持中文的全文检索，与平台现有架构相符，与Elasticsearch相比无须将数据二次插入到其他数据库中。

缺点：对海量数据支持性能较差

以下为Mysql检索性能测试数据对比

说明：

1 测试数据集中每条数据被建立索引的字段长度平均为700个字符。

2 测试结果皆为建立完全文索引后的查询速度结果。

3 因全文搜索有三种模式，测试选用更符合场景的NATURAL LANGUAGE MODE和BOOLEAN MODE进行测试。

4 高频字段测试为测试集中较多文档内容中都含有该关键字的查询,SQL使用为优化后的分页查询方法。

 

| 查询方式/数据量    | 50万      | 100万    | 200万    | 500万    |
| ------------------ | --------- | -------- | -------- | -------- |
| LANGUAGE MODE      | <0.1s     | 0.3-0.7s | 0.3-1.0s | 1.1s-4s  |
| BOOLEAN MODE       | <0.05s    | 0.1-0.3s | 0.1-0.5s | 0.1-0.7s |
| 高频字段查询(优化) | 0.05-0.5s | 0.3-1s   | 0.8s-3s  | 4-10s    |


下图是测试100万条数据过程中，对于出现”喜欢”关键字的查询结果截图。

![img](https://inventory-manager.oss-cn-beijing.aliyuncs.com/blog_image/mysql_search_ch.jpg) 



## 五、结论

如果百万级以下的数据量就用mysql
如果不差钱或者就想要好的，就选es
如果只做英文且想要性价比就选mongodb

