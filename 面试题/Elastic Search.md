# Elastic Search

docker 安装

```
docker run --restart=always -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch
```

## 为什么能存储海量数据

#### 节点 node（master，数据节点）



#### 分片 sharding（master， slave）

每一个分片都是一个 luence 引擎，有着完整的存储及索引分析能力

分配文档到不同的容器或分片中，文档可以存储在一个或多个节点中

按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡

复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失

### 分布式特性

将集群中任一节点的请求路由到存有相关数据节点

集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

## 为什么能做到近乎实时搜索



## 为什么能做到高可用
