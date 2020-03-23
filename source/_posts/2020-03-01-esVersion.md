---
title: Elasticsearch(三)并发冲突
date: 2020-03-01 00:58:34
tags: elasticsearch
categories: 笔记
---

ES是乱序的，异步并发的

我们的请求是并发的，这意味着，请求到达的目的地是不可控的，乱序的。

如果是乱序的，很可能出现一个问题，新的version文档被旧的version文档覆盖掉，或直接抛异常

- ES的乐观锁

ElasticSeach用_version来保证变更以正确的执行，如果旧版本的文档在新版本之后到达，则会被忽略

新建一个文档，这个时候我们可以看到新文档的版本号_version=1：

```
PUT /website/blog/1/_create
{
  "title" : "this is title" , 
  "txt" : "just do it"
}

```

现在尝试通过重建文档索引来保存修改数据：

请求成功，并且响应体告诉我们 _version 已经递增到 2

```
PUT /website/blog/1?version=1
{
  "title" : "this is test" , 
  "txt" : "just do it"
}
```

- 通过Elasticsearch外部版本控制

  version_type=external，只有当你提供的version比_version大的时候，才能修改完成

  version_type=external能够修改的条件就是：提供的版本号必须比_version大

  现在我们更新这个文档，指定一个新的 version 号是 10 ：

  ```
  PUT /website/blog/2?version=10&version_type=external
  {
    "title": "My first external blog entry",
    "text":  "This is a piece of cake..."
  }
  ```

  如果此时插入版本号比现在的_version小的，就会报错：

  

- 重复提交retry_on_conflict

  elasticsearch设计的目的就是多用户的海量数据操作；

  那么可能存在这样场景：A进程接收到请求尝试去检索(retrieve)和重建索引(reindex)某个文档C，B进程也接收到请求检索(retrieve)和重建索引(reindex)文档C；

  那么这个时候就会出现：其中一个进程提前修改了文档C，然后另一个进程在做检索的时候，因为_version改变了，所以匹配不到文档C，操作就会失败，然后数据丢失

  这就是在并发操作的时候经常出现的现象；

  解决：

  对于多用户的更新操作，文档被修改了并不要紧，如果出现了匹配不到的现象，我们只要重新在操作一遍就可以了；所以需要使用关键字retry_on_conflict（默认0）

  ```
  POST /website/pageviews/1/_update?retry_on_conflict=5
  {
     "script" : "ctx._source.views+=1",
     "upsert": {
         "views": 0
     }
  }
  
  ```

  retry_on_conflict=5 代表如果出现失败，最大可以重复五次的update操作