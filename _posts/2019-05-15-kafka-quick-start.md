---
layout:     post
title:      Kafka快速入門
subtitle:   kafka個人速記
date:       2019-05-15
author:     dushenzhi
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - kafka
    - 笔记
    - 消息队列
---

下载kafka，链接：http://kafka.apache.org/downloads

* 安装

``` bash
tar -xzf kafka_2.12-2.2.0.tgz
cd kafka_2.12-2.2.0
```

* 启动zk

``` bash
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

* 启动 Kafka 服务

``` bash
bin/kafka-server-start.sh -daemon  config/server.properties
```

* 创建一个topic

``` bash
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

* 查看 Kafka 中的 topic 列表

``` bash
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

* 发送消息

``` bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

* 消费消息

``` bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```







