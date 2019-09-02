---
layout: post
title:  "All the sums"
date:   2019-09-01 15:59:53 -0700
categories: tips
tag: [spring boot,microservice,monitoring,elasticsearch]
---

## Tips
微服务的环境下，整合整个服务的log和metrics是极为重要也是很有难度的任务。
spring boot cloud原生环境下，借助zuul gateway做所有service的routing, 可以加上sleuth和zipkin做到整合log，并且借助tracing Id来debug整个pipeline。
这篇博客讲解的比较清楚: [链接](https://windmt.com/2018/04/24/spring-cloud-12-sleuth-zipkin/)

如果使用service mash，Spring boot原生的service就会有些吃不开。
一下是自己工作中的一些tips。
FileBeat + logStash + elasticsearch是一套比较标的业界log处理架构。

![LogWithElasticSearch](/public/img/LogArch1.png)
使用filebeat的好处是在服务治理的时候，可以把filebeat作为daemon service. (比如用kubernetes可以每一个pod设立一个filebeat)

在做filebeat config yml的时候，只需要declear provider type是docker, file beat就是读取docker console上print的log， 并发到logstash; logstash进elasticsearch。
```yml
output:
  logstash:
    hosts: ["${LOGSTASH_URL}"]
    # Disable pipelining and set ttl value to stop "connection reset by peer" errors - https://www.pivotaltracker.com/story/show/163806665
    pipelining: 0
    ttl: 60s

filebeat.modules:
  - module: nginx
  - module: logstash

filebeat.autodiscover:
  providers:
  - type: docker
    # Set cleanup_timeout to provide file harvester time to capture shutdown logs - https://www.pivotaltracker.com/story/show/163340250
    cleanup_timeout: 120s
```
这套流程处理log十分的有用高效，然而处理metics的效率非常低，公司内部设立grafana board的时候建立了几个dash board, 为了博取数据，不得不用ragex去elasticsearch query了不少log, 当选取的dash board日期范围太大的时候，尝尝grafana container无响应。而且使用这个架构，开发的时候会log很多敏感的数据在log中，安全和debug都会有问题。
```json 
grafana board config example
{
    "datasource": "$environment",
    "enable": false,
    "hide": false,
    "iconColor": "#F2CC0C",
    "name": "Large Fos Syncs",
    "query": "service_name: \"example-service\" AND short_message.keyword: /.* Starting update for company to process [0-9]{4,} updates .*/",
    "showIn": 0,
    "tagsField": "tenantIdCode",
    "textField": "short_message"
}
```

在整体服务升级到spring boot 2.0以后，metrics的架构慢慢向orometheus靠拢。
每一个application下开启了一个micrometer的端口，从push metrics改进到prometheus主动poll数据。
![LogWithElasticSearch](/public/img/LogArch2.png)

Mvn dependency
```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

application yml部分
```yml
# Labels to enable prometheus scraping for metricbeat
LABEL prometheus.scrape="true"
LABEL prometheus.url="/actuator/prometheus"
LABEL prometheus.port="8081"
```

最后，在数据使用段，可以使用prometheus自带的UI，或者grafana都行。
另外提的是，elasticsearch也支持mircometer, 可以用这个端口提取数据，所以这个改动并不会伤筋动骨，可以缓慢递进到新的dash board

唯一的“缺点”是，code里需要inject MeterRegistry，在关键的节点push进数据。

Timer example
```java
MeterRegistry registry = new SimpleMeterRegistry();
List<String> list = new ArrayList<>(4);
 
Gauge gauge = Gauge
  .builder("cache.size", list, List::size)
  .register(registry);
```