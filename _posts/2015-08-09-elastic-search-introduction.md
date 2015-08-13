---
title:  "小白也能玩搜索"
date:   2015-08-09 10:00:00
description: Elastic Search入门
tags:
  - elasticsearch
  - docker
---

你是否为了讨女朋友的欢心而偷偷搜索过她最想要的礼物？是否在女朋友生病的时候心急地搜过医院和药物？是否在吵架过后默默地搜过和好方法？

都没有么？那么单身汪你好，至少你可以有搜索！接下来，本文就简要介绍搜索利器Elastic Search和怎样在5分24秒内用正确的姿势使用它。

##安装
如果你有linux，并且恰好也有docker， 那么请运行如下命令：

```
sudo docker run -d -p 9200:9200 -p 9300:9300 elasticsearch
```

你要是看到一串id， 恭喜你， 你已经有了自己的搜索了！就是这么简单！

我们来验证一下搜索服务：

```
ubuntu:~$ curl localhost:9200
{
  "status" : 200,
  "name" : "Adonis",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.7.1",
    "build_hash" : "b88f43fc40b0bcd7f173a1f9ee2e97816de80b19",
    "build_timestamp" : "2015-07-29T09:54:16Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```

返回值200！你成功了！这个结果除了告诉你Elastic Search已经启动好之外，还显示了版本号，build信息，lucene版本等信息。

如果你不用Linux或者Docker，放心，情况也不复杂。首先你需要Java，然后去[官网](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)下载，解压，运行既可。##使用
Elastic Search的初级功能十分易用。Restful请求就可以完成一切操作。所以接下来的体验，你只需要一个命令行即可，但是为了直观方便，你也可以选择官方推荐的[Marvel](https://www.elastic.co/products/marvel)或者[Postman](https://www.getpostman.com/)。

相信很多人都是吃货，下面，我用中国第九大菜系食堂菜作为搜索的原数据演示Elastic Search的用法。

###插入数据
首先来一道勉强正常的菜，苹果炖牛肉。


```
ubuntu:~$ curl -XPOST localhost:9200/food/canteen?pretty -d '{title: "苹果炖牛肉", source: "sjtu"}'
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HEQxfW68mQc8rk4AS",
  "_version" : 1,
  "created" : true
}
```

有点迷糊？那来稍微介绍一点背景知识，看看上面的命令都做了啥。

Elastic Search使用document方式存储， 类似MongoDB。它不仅高效而且分布式扩展性极佳，基于赫赫有名的Lucene搭建，在搜索界的地位就如同杰伦小公举在歌坛的地位一般。很多人可能有关系型数据库开发的经验，下面来张Elastic Search和MySQL的术语类比图，帮助理解。


| MySQL    | Elastic Search                |
|----------|-------------------------------|
| Database | Index                         |
| Table    | Type                          |
| Row      | Document                      |
| Column   | Field                         |
| Schema   | Mappping                      |
| Index    | Everything Indexed by default |
| SQL      | Query DSL                     |

上面例子中，`localhost:9200`是服务地址，Elastic Search默认利用9200作为Rest服务的端口，9300作为内部通信接口和Java客户端接口。`food`是index，`canteen`是type，里面的数据形成document。值得注意的是，跟其他NoSQL数据库一样，Schema不需要预先定义，直接插入数据即可，Elastic Search会智能地分析每个Field的值并自动建立索引。每个document都有一个id，如果不指定则会自动生成。可以看到插入成功之后，生成了` "_id" : "AU8HEQxfW68mQc8rk4AS"`。插入命令使用Post请求，方便使用和测试。

另外，在上面的例子中，我加入了`?pretty`主要是让结果的json显示更好看。

###获取数据
虽然数据已经存好了，但是没有直接取到，总是心有不安，下面这条命令即可以查看数据：

```
ubuntu:~$ curl -XGET localhost:9200/food/canteen/AU8HEQxfW68mQc8rk4AS?pretty
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HEQxfW68mQc8rk4AS",
  "_version" : 1,
  "found" : true,
  "_source":{title: "苹果炖牛肉", source: "sjtu"}
}
```

可以看到，请求改为了`GET`，请求形式为`/index/type/id`。

###更新数据
再来一道菜，番茄炒菠萝，用到我们刚才说的`POST`请求。

```
ubuntu:~$ curl -XPOST localhost:9200/food/canteen?pretty -d '{title: "番茄炒菠萝", source: "sjtu"}'
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HQW0-W68mQc8rk4Ab",
  "_version" : 1,
  "created" : true
}
```

等等，这道菜是武汉大学的，看来我们得更新一下`source`。也许聪明的你已经猜到，发个`PUT`请求就可以搞定了。

```
ubuntu:~$ curl -XPUT localhost:9200/food/canteen/AU8HQW0-W68mQc8rk4Ab?pretty -d '{title: "番茄炒菠萝", source: "whu"}'
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HQW0-W68mQc8rk4Ab",
  "_version" : 2,
  "created" : false
}
```

可以看到`_version`变成了2，表示更新过了。再`GET`一下确认结果。

```
ubuntu:~$ curl -XGET localhost:9200/food/canteen/AU8HQW0-W68mQc8rk4Ab?pretty
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HQW0-W68mQc8rk4Ab",
  "_version" : 2,
  "found" : true,
  "_source":{title: "番茄炒菠萝", source: "whu"}
}
```

可以看到，`_source`，也就是插入的原始数据，确实改变了。
###删除数据
继续来一道菜，月饼炒辣椒！

```
ubuntu:~$ curl -XPOST localhost:9200/food/canteen?pretty -d '{title: "月饼炒辣椒", source: "fjnu"}'
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HRPoOW68mQc8rk4Ac",
  "_version" : 1,
  "created" : true
}
```

算了，好像太丧心病狂了，对吃货的幼小心灵产生了暴击，我们还是把它删掉吧。

```
ubuntu:~$ curl -XDELETE localhost:9200/food/canteen/AU8HRPoOW68mQc8rk4Ac?pretty
{
  "found" : true,
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HRPoOW68mQc8rk4Ac",
  "_version" : 2
}
```

`GET`验证一下是否真的被删除了。

```
ubuntu:~$ curl -XGET localhost:9200/food/canteen/AU8HRPoOW68mQc8rk4Ac?pretty
{
  "_index" : "food",
  "_type" : "canteen",
  "_id" : "AU8HRPoOW68mQc8rk4Ac",
  "found" : false
}
```

显示`"found" : false`，可见删除确实成功了。
###搜索数据
Elastic Search支持相当复杂的搜索情形。像“有水果有肉有主食红黄白相间成粒状”这种可以把MySQL的检索虐得死去活来的查询，Elastic Search可以轻松告诉你这应该是名菜“哈密瓜年糕牛肉粒”。下面看看最简单却很实用的搜索：查询所有值。

```
ubuntu:~$ curl -XGET localhost:9200/food/canteen/_search?pretty
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "food",
      "_type" : "canteen",
      "_id" : "AU8HEQxfW68mQc8rk4AS",
      "_score" : 1.0,
      "_source":{title: "苹果炖牛肉", source: "sjtu"}
    }, {
      "_index" : "food",
      "_type" : "canteen",
      "_id" : "AU8HQW0-W68mQc8rk4Ab",
      "_score" : 1.0,
      "_source":{title: "番茄炒菠萝", source: "whu"}
    } ]
  }
}
```

可以看到，形如`GET _search`即可。结果会显示所有给定index和type下的document。`hits`下面包含找到的具体内容。`_score`表示相关程度，分数越高表示越相关，搜索引擎对于结果的排序都是通过类似的机制完成。
再来一个简单地查询，查找`source`是`sjtu`的菜。

```
ubuntu:~$ curl -XGET localhost:9200/food/canteen/_search?pretty -d '{"query":{"match":{"source":"sjtu"}}}'
{
  "took" : 35,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.30685282,
    "hits" : [ {
      "_index" : "food",
      "_type" : "canteen",
      "_id" : "AU8HEQxfW68mQc8rk4AS",
      "_score" : 0.30685282,
      "_source":{title: "苹果炖牛肉", source: "sjtu"}
    } ]
  }
}
```

这里用到了`match query`来搜索，可以看到`_score`的变化。
##结语
本文对Elastic Search的使用作了最简单的介绍，适合新手入门。上述例子简单直观，命令可以直接运行，亲测有效。有兴趣的读者可以自己动手实践，加入更多诸如“糖宝炖骨头”这样的黑暗料理。
