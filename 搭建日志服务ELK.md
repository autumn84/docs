# 搭建日志服务ELK

## 目标
收集集群日志

## 软件
Elasticsearch + Logstash + Kibana 

1. Elasticsearch : Elasticsearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases.
2. Logstash : Logstash is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash”.
3. Kibana : Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack.

参考：[Log Management for Docker Swarm with ELK Stack](https://botleg.com/stories/log-management-of-docker-swarm-with-elk-stack/)

## 安装

### Elasticsearch安装及配置

```
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.6.1
```

### Logstash安装及配置
```
docker run --rm -it --name logstash --link elasticsearch -d -v ~/logstash.conf:/usr/share/logstash/pipeline/logstash.conf -v ~/log:/usr/share/logstash/log docker.elastic.co/logstash/logstash:6.6.1
```

### Kibana
```
docker run -d --name kibana -p 5601:5601 --link elasticsearch -e ELASTICSEARCH_URL=http://elasticsearch:9200 docker.elastic.co/kibana/kibana:6.6.1
```
