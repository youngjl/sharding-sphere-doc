+++
pre = "<b>3.1. </b>"
title = "数据分片"
weight = 1
chapter = true
+++

## 背景

传统的将数据集中存储至单一数据节点的解决方案，在性能和可用性两方面已经难于满足互联网的海量数据场景。
由于关系型数据库大多采用B+树类型的索引，在数据量超过阈值的情况下，索引高度的增加也将使得磁盘访问的IO次数增加，进而导致查询性能的大幅下降；同时高并发访问请求也使得集中式数据库成为系统的最大瓶颈。 
在传统的关系型数据库无法满足需要的情况下，将数据存储至原生支持NoSQL的尝试越来越多，但NoSQL对SQL的不兼容性以及生态圈的不完善，使得它们在与关系型数据库的博弈中始终无法完成致命一击，而关系型数据库的地位依然不可撼动。

为解决关系型数据库面对海量数据由于数据量过大而导致的性能问题时，将数据进行分片是行之有效的解决方案，而将集中于单一节点的数据拆分并分别存储到多个数据库或表，称为分库分表。
分库可以有效分散高并发量，分表虽然无法缓解并发量，但仅跨表仍然可以使用数据库原生的ACID事务。而一旦跨库，涉及到事务的问题就会变得无比复杂。

分库分表一般有两种拆分方式。按照业务拆分的方式称为垂直拆分。例如，将用户库和订单库拆分到不同的数据库中。垂直拆分可以缓解数据量和访问量带来的问题，但无法根治，如果垂直拆分之后的用户和订单数量依然超过单节点所能承载的阈值，则需要水平拆分来进一步处理。
将一个表中的数据按照一定的业务规则拆分至不同表和数据库中，称之为水平拆分。例如，原来的订单数据在order_ds.t_order表中，如果按照订单的user_id将订单拆分为2个库，再按照订单的order_id在每个库中分成4个表，那么拆分的结果则是order_ds0.t_order0, order_ds0.t_order1, order_ds0.t_order2, order_ds0.t_order3, order_ds1.t_order0, order_ds1.t_order1, order_ds1.t_order2, order_ds1.t_order3。
这只是简单的水平拆分案例，在实际使用中，将库和表拆分的更加分散也是十分常见的。

虽然数据分片解决了性能问题，但也额外的引入了其他问题。面对如此散乱的分库分表之后的数据，应用开发和运维人员对数据库的操作变得异常繁重就是其中的重要挑战之一。他们需要知道什么样的数据需要从哪个具体的数据库的分表中去获取。**透明化分库分表所带来的影响，让使用方尽量像使用一个数据库一样使用水平拆分之后的数据库，是分库分表中间件的主要功能。**
