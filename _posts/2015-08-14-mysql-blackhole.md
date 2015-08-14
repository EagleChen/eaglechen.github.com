---
title:  MySQL的blackhole引擎
date:   2015-08-14 10:00:00
description: MySQL的黑科技
tags:
  - mysql
---

MySQL的存储引擎中MyISAM和InnoDB比较出名。最新，因为一些特殊的需求，了解到了另外一个引擎Blackhole，就跟名字一样，这简直就是MySQL的黑科技！

如果创建一个表的时候，使用了Blackhole引擎，那么这个表的数据不会被写入。举个例子，你有一个表abc，执行insert等操作可以成功，然而INSERT操作之后，数据并不会被存储下来而是丢弃了。这个时候，如果执行SELECT操作，会发现结果是空。

这让我想到了一个朋友，每次跟他一起吃了饭（INSERT），问他吃饱了没有（SELECT），他都会说“没有”。然后他会继续补刀各种零食，直到下一个正餐的时间到来，开始继续吃主食。

我的朋友这么吃， 显然有其意义所在，他获得了满足啊，他长了肉啊，他可以衬托其他人瘦啊（偷笑）！然而，Blackhole对MySQL来说，有啥子意义呢？

意义在于，一个MySQL Server如果作为多层Replication的中间节点，那么它可以在保证binlog正确的同时，不增加额外的延迟（因为根本不保存数据呀）。当然，还有另外一个意义，就是MySQL Server作为Replication的最终Slave的时候，相当于可以帮助忽略一些表，起到类似于--replicate-ignore-db的作用。那你要问了，为啥不直接用--replicate-ignore-db呢？不是我不想用，是因为我要用的Amazon RDS丧心病狂地把它禁掉了(已哭晕在厕所)...
