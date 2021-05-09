### 更新文档

```json
// 更新操作 
1. 会覆盖原来的数据：
PUT blog/_doc/1
{
  "title":"666"
}
2. 部分更新，不会覆盖原来的数据
POST blog/_update/1
{
  "script": {
    "lang": "painless",
    "source":"ctx._source.title=params.title",
    "params": {
      "title":"jieqiyue"
    }
  }
}
// 再来一个更新操作
POST spring_boot/_update/79
{
  "script": {
    "lang": "painless",
    "source":"ctx._source.title=params.title",
    "params": {
      "title":"new_title"
    }
  }
}
3.使用参数带update的更新方式
POST spring_boot/_doc/79/_update
{
  "doc":{
    "title":"thredd"
  }
}
```

以上三种都可以更新文档。

还有一些高级更新，

##### 条件更新

```json
// 条件更新
GET spring_boot/_doc/79/_source

POST spring_boot/_update/79
{
  "script": {
    "lang": "painless",
    "source": "if (ctx._source.age==\"88\"){ctx._source.age=\"123\"}else{ctx.op=\"none\"}"
  }
}
// 根据查询来更新，这个貌似也可以转换为条件更新？
POST spring_boot/_update_by_query
{
  "script": {
    "source": "ctx._source.age=\"8\"",
    "lang": "painless"
  },
  "query": {
    "term": {
      "age":"6"
    }
  }
}
```

### 删除文档

```json
// 删除文档
DELETE spring_boot/_doc/79
```

##### 查询删除

```json
POST blog/_delete_by_query
{
  "query":{
    "term":{
      "title":"666"
    }
  }
}
```

也可以删除某一个索引下的所有文档：

```
POST blog/_delete_by_query
{
  "query":{
    "match_all":{
      
    }
  }
}
```

###  批量操作

首先需要将所有的批量操作写入一个 JSON 文件中，然后通过 POST 请求将该 JSON 文件上传并执行。

例如新建一个名为 aaa.json 的文件，内容如下：![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmxjmzP3pwKJILMZdMnJQqu4Tiby3TxMNWemwqZLaicKDgicmduSsc3DuVH5xcnugLoicZAPlzQ9JRq8w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

aaa.json 文件创建成功后，在该目录下，执行请求命令，如下：

```
curl -XPOST "http://localhost:9200/user/_bulk" -H "content-type:application/json" --data-binary @aaa.json
```

执行完成后，就会创建一个名为 user 的索引，同时向该索引中添加一条记录，再修改该记录，最终结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/GvtDGKK4uYmxjmzP3pwKJILMZdMnJQqu2KT3WfxoiaibOIBgqLiam5ic6U8Yyu3Y1hD0t0qTKSbntse5NWQcaWibJMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 并发控制

> 之前的版本

使用 version 和 version_type 来进行控制。**这个version是针对某一个文档的。**

```json
PUT blog/_doc/2?version=2423&version_type=external_gte
{
  "title":"7adfsdfaaa"
}
```

> 现在的版本控制

```json
PUT blog/_doc/4?if_seq_no=17&if_primary_term=3
{
   "title":"4444aaa"
}
```

`_seq_no` 是对于整个索引来说的，对于索引的增加或者是删除等操作都会将这个增加。但是对于其中的某一个文档来说还是不会增加的。

### elasticsearch 简介及发展历史 

![image-20210504160856119](../../TyporaImages/image-20210504160856119.png)

![image-20210504161018796](../../TyporaImages/image-20210504161018796.png)

### 安装elasticsearch

![image-20210504163534505](../../TyporaImages/image-20210504163534505.png)

![image-20210504164932329](../../TyporaImages/image-20210504164932329.png)

### 基本概念

- 文档 Document

![image-20210504180022042](../../TyporaImages/image-20210504180022042.png)

![image-20210504180233117](../../TyporaImages/image-20210504180233117.png)

![image-20210504180704340](../../TyporaImages/image-20210504180704340.png)

![image-20210504180712983](../../TyporaImages/image-20210504180712983.png)

![image-20210504181554712](../../TyporaImages/image-20210504181554712.png)

![image-20210505230153729](../../TyporaImages/image-20210505230153729.png)

![image-20210505230214750](../../TyporaImages/image-20210505230214750.png)

### 节点，集群，分片，副本

![image-20210507145907147](../../TyporaImages/image-20210507145907147.png)

![image-20210507145921380](../../TyporaImages/image-20210507145921380.png)

![](../../TyporaImages/image-20210507150859486.png)

分片就是对于一个索引的拆分。默认情况下，创建一个索引会创建一个 1 个分片，并且为每一个分片创建一个副本。因为如果单个索引放在一台机器上，性能不够。分片不是越多越好，一般要根据总数据量来进行计算。

建议：（仅参考）

   1、每一个分片数据文件小于30GB

   2、每一个索引中的一个分片对应一个节点

   3、节点数大于等于分片数

### crud

> 索引

一、创建索引

1. 发送put请求，比如说发送 put 请求到 localhost:9200/test，就创建了一个test索引。
2. 通过head插件进行创建。head插件就是浏览器上的一个插件。
3. 使用kibana发送 put test 请求。

注意两点：

- 索引名称不能重复
- 索引名称不能有大写字母

二、更新索引

​	1. 修改分片数量：

```
PUT book/_settings
{
  "number_of_replicas": 2
}
```

三、查看索引

```
GET book/_settings
GET _all/_settings
```

四、删除索引

```
DELETE test
```

删除一个不存在的索引会报错。

五、关闭索引

```
POST book/_close
```

六、索引别名

```
// 添加索引
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "book",
        "alias": "book_alias"
      }
    }
  ]
}
 // 移除索引
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "book",
        "alias": "book_alias"
      }
    }
  ]
}

GET /book/_alias

// 查看某一个别名对应的索引（book_alias 表示一个别名）：
GET /book_alias/_alias

GET /_alias
```

> 文档

![image-20210507152641060](../../TyporaImages/image-20210507152641060.png)

```
// 写入一条文档
PUT book/_doc/1
{
  "title":"三国演义"
}
```

权限相关：

```
// 关闭索引的写权限:
PUT book/_settings
{
  "blocks.write": true
}
```

一、新增文档

指定id的方式：

```
PUT blog/_doc/1
{
  "title":"6. ElasticSearch 文档基本操作",
  "date":"2020-11-05",
  "content":"微信公众号**江南一点雨**后台回复 **elasticsearch06** 下载本笔记。首先新建一个索引。"
}
```

```
// 添加成功后返回值如下：
{
  "_index" : "blog",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

- _index 表示该文档属于哪个索引。
- _type 表示文档的类型。
- _id 表示文档的 id。
- _version 表示文档的版本（更新文档，版本会自动加 1，针对一个文档的）。
- result 表示执行结果。
- _shards 表示分片信息。
- `_seq_no` 和 `_primary_term` 这两个也是版本控制用的（针对当前 索引）。而_version 针对的是文档。

不指定id的方式：

```
POST blog/_doc
{
  "title":"666",
  "date":"2020-11-05",
  "content":"微信公众号**江南一点雨**后台回复 **elasticsearch06** 下载本笔记。首先新建一个索引。"
}
```

二、获取文档

获取单个文档

```
GET blog/_doc/RuWrl3UByGJWB5WucKtP
```

也可以批量获取文档

```
GET blog/_mget
{
  "ids":["1","RuWrl3UByGJWB5WucKtP"]
}
```

三、更新文档

更新整个文档：

这种方式会将原文档全部覆盖！

```
PUT blog/_doc/RuWrl3UByGJWB5WucKtP
{
  "title":"666"
}
```

更新分文档：

```
POST blog/_update/1
{
  "script": {
    "lang": "painless",
    "source":"ctx._source.title=params.title",
    "params": {
      "title":"666666"
    }
  }
}
```

更新的请求格式：POST {index}/_update/{id}

在脚本中，lang 表示脚本语言，painless 是 es 内置的一种脚本语言。source 表示具体执行的脚本，ctx 是一个上下文对象，通过 ctx 可以访问到 `_source`、`_title` 等。

通过上面可以看出，更新的格式就是最后要加上一个id值来指定某一个具体的文档。但是我们也可以使用查询更新。就是通过给定一个查询条件，然后将查询出来的文档进行更新。

```
POST blog/_update_by_query
{
  "script": {
    "source": "ctx._source.content=\"888\"",
    "lang": "painless"
  },
  "query": {
    "term": {
      "title":"666"
    }
  }
}
```

注意看这里查询更新的格式，最后并没有指定id值。

四、删除文档

删除和update一样，也是有两种。一种是直接指定id删除，一种是根据查询来删除。

```
1. 	DELETE blog/_doc/TuUpmHUByGJWB5WuMasV

2. 	POST blog/_delete_by_query
    {
      "query":{
        "term":{
          "title":"666"
        }
      }
    }
    
  删除一个索引下面的所有文档  
3.	POST blog/_delete_by_query
    {
      "query":{
        "match_all":{

        }
      }
    }
```

也可以通过Bulk API进行批量操作：

https://blog.csdn.net/prestigeding/article/details/84205785

五、其它

https://blog.csdn.net/ZYC88888/article/details/100727714#:~:text=%E4%B8%80%E8%88%AC%E7%9A%84%EF%BC%8Celas,%E7%BE%A4%E4%B8%8D%E5%B9%B3%E8%A1%A1%E7%9A%84%E6%83%85%E5%86%B5%E3%80%82

当新增一个文档的时候，这个文档究竟是存在哪个分片上的呢？

![img](https://img-blog.csdnimg.cn/20190911101509931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly96eWM4OC5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70)

当一个索引被创建以后，一般分片数都不允许更改。原因就是当更改了分片数之后，通过原来的所有的路由（routing）值都会无效，文档就再也找不到了。因为长度变化了，但是hash值没有变。

https://www.elastic.co/guide/cn/elasticsearch/guide/current/distrib-read.html

https://www.elastic.co/guide/cn/elasticsearch/guide/current/distributed-search.html

**在博客中说，文档会被分散在节点中，elasticsearch自己都不知道文档具体在哪个分片上面，所以得要发送给所有的分片。那为什么不能够再根据hash算法算出文档具体在哪个分片上，然后直接去那个分片上面拿数据不就行了？**

取回单个文档确实是先通过hash算法来计算出文档所在分片，然后再去这个分片的所有副本中选择一个来进行查询（负载均衡），然后再将结果返回的。此时可能就是要去副本分片上面查询，但是结果现在还是存放在主分片上面的。所以，可能会返回空。但是主分片中是有值的。

所以说要分清楚对于单个文档做查询还是说是分布式搜索。对于分布式搜索，每一个分片都有可能有满足你搜索条件的结果。

对于***查询接口***来说，默认会搜索所有的分片：

```
GET route_test/_search?routing=key1,key2 
{
  "query": {
    "match": {
      "data": "b"
    }
  }
}
```

### 数据类型

 [www.cnblogs.com/shoufeng/p/10692113.html](https://www.cnblogs.com/shoufeng/p/10692113.html#:~:text=1 核心数据类型 1.... 2 复杂数据类型 2.1  数组类型,4.1  IP类型 4.... 5 参考资料 6 版权声明)

https://zhuanlan.zhihu.com/p/136981393

### 映射参数

- analyzer 与 search_analyzer 参数：

```
PUT blog
{
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "ik_smart"
      }
    }
  }
}
```

​	创建索引的时候指定分词器是ik分词器。

​	search_analyzer 是搜索的时候指定分词器的。

- normalizer 

  ```
  // 未标准化之前
  PUT blog
  {
    "mappings": {
      "properties": {
        "author":{
          "type": "keyword"
        }
      }
    }
  }
  // 标准化之后
  PUT blog
  {
    "settings": {
      "analysis": {
        "normalizer":{
          "my_normalizer":{
            "type":"custom",
            "filter":["lowercase"]
          }
        }
      }
    }, 
    "mappings": {
      "properties": {
        "author":{
          "type": "keyword",
          "normalizer":"my_normalizer"
        }
      }
    }
  ```

- doc_values 和 fielddata

  doc_values 默认是开启的，如果确定某个字段不需要排序或者不需要聚合，那么可以关闭 doc_values。而fielddata默认是关闭的。fielddata是用在text字段上面的，在text第一次做聚合或者是排序的操作的时候。那么一般来说，不会再text字段上面做这种操作。而且这个fielddata是存在内存里面的。

  对于非text字段默认是开启的。那么就能够直接在这个字段上面做排序的操作。但是，如果关闭了doc_values的话，再去用这个字段做排序的话，就会报错。

- enabled

  某一个字段是否需要被建立倒排索引。enabled为false的字段是不能够被搜索到的。