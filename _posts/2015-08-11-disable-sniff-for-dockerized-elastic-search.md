---
title:  "Elastic Search在docker里的坑"
date:   2015-08-09 10:00:00
description: Elastic Search在docker环境中的坑
---
Elastic Search很好搭建，使用方便，然而，在docker环境里坑还不少。这里遇到一个`transport client`连不上ES的问题。

##问题
Elastic Search由docker启动运行，放在A机器上，9200，9300均已经mapping到宿主机对应的端口。Java Client在B机器上。
在B机器上curl ES服务的9200端口显示一切正常，并且查询也能工作，但是Java Transport Client却怎么都连不上。一直报如下错误：

```
no Elasticsearch node available
```

##解决方法
我就纳闷了，一切都正常为啥总是连不上呢？查了很多资料后发现，是因为Elastic Search被放进docker之后，一些Client机制不能正常工作了。这个时候只需要把Transport Client的Sniff关闭即可。
