---
layout: post
title: Spring Cloud 配置 ELK + rabbitmq/kafka 分析日志
categories: SpringCloud
description: 微服务设计与实践原则
keywords: web, micro, design, service
---

后续有需要再完善集群的教程。

整理流程如下：

# linux 设置

需要 jdk1.8 及以上

linxu 参数调优如下：

```shell
vi /etc/security/limits.d/90-nproc.conf # 改为如下
*          soft    nproc     10240

vi /etc/sysctl.conf
vm.max_map_count=655360  # 增加

# 或者 echo 'vm.max_map_count=655360' >> /etc/sysctl.conf

vi /etc/security/limits.conf  # 增加以下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096

sysctl -p # 使得参数优化设置生效
```

创建 ELK 账户：

```shell
groupadd  elk
useradd elk -g elk
chown -R elk:elk /elk
su elk
```

下载 ELK 的安装包：

- [elasticsearch-6.2.4](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.tar.gz)
- [logstash-6.2.4](https://artifacts.elastic.co/downloads/logstash/logstash-6.2.4.tar.gz)
- [kibana-6.2.4-linux-x86_64](https://artifacts.elastic.co/downloads/kibana/kibana-6.2.4-linux-x86_64.tar.gz)

# ELK 安装
## Elasticsearch

```shell
cd elasticsearch-6.2.4
cp config/elasticsearch.yml config/elasticsearch.yml.bak
vi config/elasticsearch.yml # 修改如下参数

network.host=0.0.0.0
network.port=9200

bootstrap.memory_lock: false
bootstrap.system_call_filter: false

./bin/elasticsearch # 启动
```

`kill` 对应的 java 进程即可结束服务。

以上即完成了单实例的 Elasticsearch。

## Logstash

官网教程链接:

[https://www.elastic.co/guide/en/logstash/current/input-plugins.html](https://www.elastic.co/guide/en/logstash/current/input-plugins.html "查看如何配置 logback 和 log4j")

[https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html](https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html "官网配置教程链接")

[https://www.elastic.co/guide/en/logstash/current/index.html](https://www.elastic.co/guide/en/logstash/current/index.html "官网教程链接")

### logstash 处理 redis, kafka, rabbitmq 输入源

```shell
cd logstash-6.2.4
logstash -e "input {stdin{}} output {stdout{}}" # 执行测试，输入啥就增加格式打印啥
vi config/my_logstash.conf # 写入以下内容
input {
  # For detail config for log4j as input,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-log4j.html
  # https://www.elastic.co/guide/en/logstash/current/plugins-inputs-redis.html
  redis {
    host => "192.168.1.233"
    port => 6379
    key => "logstash_yunka_redis"
    data_type => "list"
    db => 1
    # password => "" 可选
    threads => 3
  }
  # https://www.elastic.co/guide/en/logstash/current/plugins-inputs-kafka.html
  kafka {
    bootstrap_servers => ["192.168.1.236:9092"]
    client_id => "logstash_yunka"
    group_id => "yunka_log_group"
    topics => ["logstash_yunka_kafka"]
    auto_offset_reset => "latest"
  }
  rabbitmq {
    host => "172.16.100.156"
    user => "rabbitmq"
    password => "rabbitmq"
    port => 5672
    durable => true
    queue => "zipkin"
    vhost => "/"
  }
}
filter {
  #Only matched data are send to output.
}
output {
  # For detail config for elasticsearch as output,
  # See: https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html
  elasticsearch {
    hosts  => ["127.0.0.1:9200"]   # ElasticSearch host, can be array.
    index  => "yunka"              # The index to write data to.
  }
}

./bin/logstash -f /home/elk/my_logstash.conf # 即可启动 logstash 服务并应用配置的采集方式。
```

需要注意的是，支持的 input 方式常用有： syslog, log4j, http, jdbc, kafka, rabbitmq, redis, websockt, **tcp(logback)**

以上即配置 redis/kafka/rabbitmq 为总日志信息来源，elasticsearch 为 日志采集后发送的目的地。

### logstash agent 采集日志文件

```shell
vi config/my_logstash_agent.conf # 写入以下内容
input {
  # https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html
  # https://www.elastic.co/guide/en/logstash/6.2/configuration-file-structure.html
  file {
    path => "/home/elk/yunka*.log"
  }
}
filter {
  #Only matched data are send to output.
  # pattern matching logback pattern
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span},%{DATA:exportable}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
  }
}
output {
  elasticsearch {
    hosts  => ["127.0.0.1:9200"]   # ElasticSearch host, can be array.
    index  => "yunka_log"              # The index to write data to.
  }
  rabbitmq {
    host => "172.16.100.156"
    user => "rabbitmq"
    password => "rabbitmq"
    port => 5672
    durable => true
	exchange => ""
	exchange_type => "direct"
    key => "zipkin"
    vhost => "/"
  }
}
```

## Kibana

```shell
cd kibana-6.2.4-linux-x86_64
cp config/kibana.yml config/kibana.yml-bak
vi config/kibana.yml

server.port: 5601
server.host: "0.0.0.0"
elasticsearch.url: "http://ip:9200"
kibana.index: ".kibana"

./bin/kibana
curl http://localhost:5601
```

以上即完成了 Kibana 的配置和启动。

注意：若采用 logback 直接发送日志到 logstash agent 的方式，建议在 logback 配置中增加如下的 appender：

```xml
<appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <remoteHost>127.0.0.1</remoteHost>
    <port>4567</port>
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>INFO</level>
    </filter>
    <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
```

## ELK 启动 sh 脚本

```shell
vi run_elk.sh

#!/bin/bash
nohup sh /home/elk/elasticsearch-6.2.4/bin/elasticsearch > /home/elk/nohup_elasticsearch.out 2>&1 &
nohup sh /home/elk/kibana-6.2.4-linux-x86_64/bin/kibana > /home/elk/nohup_kibana.out 2>&1 &
nohup sh /home/elk/logstash-6.2.4/bin/logstash -f /home/elk/my_logstash.conf > /home/elk/nohup_logstash.out 2>&1 &
echo "done startting elk."
```

# Kafka & Zookeeper

- [kafka_2.11-1.1.0](http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/1.1.0/kafka_2.11-1.1.0.tgz)
- [zookeeper-3.4.10](http://mirrors.shu.edu.cn/apache/zookeeper/stable/zookeeper-3.4.10.tar.gz)

## 配置 Zookeeper

```shell
# 使用外置 zookeeper
cd zookeeper-3.4.10
cp conf/zoo_sample.cfg conf/zoo.cfg
vi conf/zoo.cfg

dataDir=/home/elk/zookeeper_data
clientPort=2181

chmod -R -x bin/*
chmod +x bin/zkServer.sh
./bin/zkServer.sh status|start|stop|restart

# 以下为使用内置 zookeeper
cd kafka_2.11-1.1.0
cp config/zookeeper.properties config/zookeeper.properties-bak
vi config/zookeeper.properties

dataDir=/home/elk/zookeeper_data
clientPort=2181
```

## 配置 Kafka

```shell
cp config/consumer.properties config/consumer.properties-bak
vi config/consumer.properties

bootstrap.servers=192.168.1.236:9092
group.id=yunka_log_group


cp config/server.properties config/server.properties-bak
vi config/server.properties

broker.id=0
listeners=192.168.1.236:9092
advertised.listeners=192.168.1.236:9092
log.dirs=/home/elk/ #日志存放路径，上面创建的目录
zookeeper.connect=localhost:2181 #zookeeper地址和端口，单机配置部署，localhost:2181
host.name=你的ip

cp config/producer.properties config/producer.properties-bak
vi config/producer.properties

bootstrap.servers=192.168.1.236:9092  # 消费者连接的 kafka 信息，必须为实际 IP

# 修改环境变量
vi ~/.bash_profile
JMX_PORT=9999
export JMX_PORT

rm -rf /home/elk/zookeeper_data/* /home/elk/kafka_log/*
/home/elk/kafka_2.11-1.1.0/bin/zookeeper-server-start.sh /home/elk/kafka_2.11-1.1.0/config/zookeeper.properties
/home/elk/kafka_2.11-1.1.0/bin/kafka-server-start.sh /home/elk/kafka_2.11-1.1.0/config/server.properties

# 常用运维命令
# 手动创建 topic
bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --topic test_topic --zookeeper localhost:2181
# console 消费者
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test_topic
# console 生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic
# 列出 topic
bin/kafka-topics.sh --zookeeper localhost:2181 --list
# 显示 topic 详情
bin/kafka-topics.sh --zookeeper localhost:2181 -describe -topic test_topic
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:2181 --group test_group --topic test_topic

## Zookeeper & Kafka 启动脚本

nohup sh /home/elk/kafka_2.11-1.1.0/bin/zookeeper-server-start.sh /home/elk/kafka_2.11-1.1.0/config/zookeeper.properties > /home/elk/nohup_zookeeper.out 2>&1 &
nohup sh /home/elk/kafka_2.11-1.1.0/bin/kafka-server-start.sh /home/elk/kafka_2.11-1.1.0/config/server.properties > /home/elk/nohup_kafka.out 2>&1 &
JMX_PORT=9999 nohup sh /home/elk/kafka_2.11-1.1.0/bin/kafka-server-start.sh /home/elk/kafka_2.11-1.1.0/config/server.properties > /home/elk/nohup_kafka.out 2>&1 &
echo "done startting zookeeper,kafka ."
```

## Kafka 监控工具

### KafkaOffsetMonitor

社区提供 [KafkaOffsetMonitor](https://github.com/quantifind/KafkaOffsetMonitor/releases)

`java -cp KafkaOffsetMonitor-assembly-0.2.0.jar com.quantifind.kafka.offsetapp.OffsetGetterWeb --zk 192.168.1.236:2181 --port 9092 --refresh 10.seconds --retain 1.days`

-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.port=666 -Djava.rmi.server.hostname=192.168.1.100

### kafka-manager

Yahoo 开源 [github地址](https://github.com/yahoo/kafka-manager)

```shell
cd kafka-manager-1.3.0.7
cp conf/application.conf conf/application.conf-bak
vi conf/application.conf

kafka-manager.zkhosts="localhost:2181"

bin/kafka-manager -Dconfig.file=/path/to/application.conf -Dhttp.port=8080
```

### kafkatool

[kafkatool](http://www.kafkatool.com/) 一个 java 桌面应用

# 应用集成

## Spring Cloud Sleuth 支持

```yml
spring:
  application:
    name: spring-cloud-eureka-producer
  cloud:
    inetutils:
      ignored-interfaces: ethx
  sleuth:
    sampler:
      percentage: 1.0
  zipkin:
#    base-url: http://192.168.1.241:8087
    sender:
      type: kafka
    rabbitmq:
      queue: zipkin_queue
    kafka:
      topic: zipkin_topic
  kafka:
    bootstrap-servers: 192.168.1.236:9092
    client-id: ${spring.application.name}_${server.port}
  rabbitmq:
    host: 172.16.100.156
    port: 5672
    username: rabbitmq
    password: rabbitmq
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

可视情况，使用 sleuth 将应用调用情况的日志发送到 web(zipkin), 或者 rabbitmq/kafka 发送到 ELK.

在Spring Cloud 中详细使用 ELK 可参考另一篇文章：

asdas

...
