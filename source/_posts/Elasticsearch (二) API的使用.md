---
title: Elasticsearch (二) API的使用
categories: 笔记
tags: [elasticsearch]
date: 2020-02-26
---

# Index API

Index API 允许我们储存一个JSON格式的文档，使数据可以被搜索。

文档通过index，type，id唯一确定。我们可以自己提供一个ID，也可以使用Index API为我们生成一个。

生成JSON格式的document有以下方式

- 手动拼接 String json = {"..."}

- Map方式 Map json = new HashMap...

- 序列化方式，es已经使用了jackson。可以直接使用它把javaben转为json

  ```English
  // instance a json mapper 
  ObjectMapper mapper = new ObjectMapper();// create once, reuse 
  // generate json 
  byte[] json = mapper.writeValueAsBytes(csdn); 
  IndexResponse response = client.prepareIndex("fendo", "fendo date") .setSource(json) .get();
  System.out.println(response.getResult());
  ```

- ES内置了一个帮助类：XContentBuilder来生成JSON文档（暂略）

# GET API

当单线程时：（下面的例子是 operationThreaded 设置为 false ）

```
GetResponse response = client.prepareGet("twitter", "tweet", "1" ) .setOperationThreaded(false) .get();
```

# DELETE API

```
DeleteResponse response = client.prepareDelete("twitter", "tweet ", "1") .setOperationThreaded(false) .get();
```



# DELETE BY QUERY API

按条件删除

```
BulkByScrollResponse response = DeleteByQueryAction.INSTANCE.newRequestBuilder(client) .filter(QueryBuilders.matchQuery("gender", "male")) //查 询条件 .source("persons") //index(索引名) .get(); //执行 long deleted = response.getDeleted(); //删除文档的数量
```

# UPDATE API

有两种方式更新索引：

- UpdateRequest

  ```
  UpdateRequest updateRequest = new UpdateRequest(); updateRequest.index("index"); updateRequest.type("type"); updateRequest.id("1"); updateRequest.doc(jsonBuilder() .startObject() .field("gender", "male") .endObject()); client.update(updateRequest).get();
  ```

- prepareUpdate

```
client.prepareUpdate("ttl", "doc", "1") .setScript(new Script("ctx._source.gender = \"male\"" , ScriptService.ScriptType.INLINE, null, null)).get();
//脚本可以是本地文件存 储的，如果使用文件存储的脚本，需要设置 //ScriptService.ScriptType.FILE 
client.prepareUpdate("ttl", "doc", "1") .setDoc(jsonBuilder()  .startObject() .field("gender", "male") .endObject()) .get();
```

还有一个Upset 更新插入，如果存在文档就更新，如果不存在就插入

```
IndexRequest indexRequest = new IndexRequest("index", "type", "1 ") .source(jsonBuilder() .startObject() .field("name", "Joe Smith") .field("gender", "male") .endObject()); 
UpdateRequest updateRequest = new UpdateRequest("index", "type", "1") .doc(jsonBuilder() .startObject() .field("gender", "male") .endObject()) .upsert(indexRequest);  client.update(updateRequest).get();
```

# Multi Get Api

一次获取多个文档

```
MultiGetResponse multiGetItemResponses = client.prepareMultiGet( ) .add("twitter", "tweet", "1") //一个id的方式 
.add("twitter", "tweet", "2", "3", "4") //多个id的方式 
.add("another", "type", "foo") //可以从另外一个索引获取 
.get();
for (MultiGetItemResponse itemResponse : multiGetItemResponses) { 
//迭代返回值
GetResponse response = itemResponse.getResponse(); if (response.isExists()) { String json = response.getSourceAsString();  } }
```

[TOC]

------

更多请查看https://inventory-manager.oss-cn-beijing.aliyuncs.com/data/elasticsearch-java.pdf

