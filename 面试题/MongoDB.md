# MongoDB

分布式文件存储 NoSql 数据库，C++ 编写，主要针对 web 应用提供可扩展高可用的数据存储解决方案

最像关系型数据库的 nosql 数据库，在某些方面可以替代 mysql，比如 mongodb 拥有数据与数据的引用、排序、分页、聚合等

它提供了丰富的查询语言

mongo 存储数据是 key-value 结果的 BSON （二进制 json）文档

支持建立索引

## 概念

| SQL术语     | MongoDB术语 | 说明                  |
| ----------- | ----------- | --------------------- |
| database    | database    | 数据库                |
| table       | collection  | 数据库表/集合         |
| row         | document    | 行/文档               |
| column      | field       | 字段/域               |
| index       | index       | 索引                  |
| table joins |             | 表连接，MongoDB不支持 |
| primary key | primary key | 主键                  |

### 数据库

一个 mongodb 中可以建立多个数据库

默认数据库为 “db”，该数据库默认存储在 /data/db 目录中 （安装时可以默认或指定一个存在的目录）

单个实例可容纳多个独立的数据库

### 集合

存在于数据库，没有固定的结构，可以插入不同格式和类型的数据，通常情况下插入集合的数据都有一定的关联性

### 文档

## 数据类型

| 数据类型  | 说明         | 解释                                | 举例                  |
| --------- | ------------ | ----------------------------------- | --------------------- |
| String    | 字符串       | UTF-8编码的字符串才是合法的         | {“v”: "value"}        |
| Integer   | 整形数值     |                                     | {“v”: 1}              |
| Boolean   | 布尔值       |                                     | {“v"： true}          |
| Double    | 双精度浮点数 |                                     | {“v”： 3.14}          |
| ObjectID  | 对象ID       | 用于创建文档ID                      | {“v”： Object( ) }    |
| Array     | 数组         |                                     | {"array": ["a", "b"]} |
| Timestamp | 时间戳       |                                     |                       |
| Object    | 内嵌文档     | 文档可以作为文档中某个 key 的 value | {“o”: {"foo": "bar"}} |
| Null      | 空值         | 空值或未定义的对象                  | {"v": null}           |
| Date      | 日期         |                                     | {“date”： new date()} |
| Regular   | 正则表达式   | 文档中包含正则表达式，遵循 js 语法  | {"v": /kkb/i}         |
| Code      | 代码         | 可以包含 js 代码                    | {"x"： funcation(){}} |

##  底层原理

部署方案

单机部署、副本集（主备）部署、分片部署、副本集与分片混合部署

### 副本集与分片混合部署

集群部署方案有三类角色:  实际数据存储节点，配置文件存储节点和路由接入节点

- 实际数据存储节点的作用就是存储数据
- 路由接入节点的作用是在分片的情况下起到负载均衡的作用
- 配置文件存储节点的作用是存储片键与 chunk 以及 chunk 与 server 的映射关系

## 索引

### 单字段索引

升序 / 降序

1 / 0

```
db.comment.createIndex({userid:1}) 
```



### 复合索引

```
db.comment.createIndex({userid:1}) 
```

### 其他索引

#### 地理空间索引

返回结果时使用平面几何的二维索引

返回结果时使用球面几何的二维球面索引

#### 文本索引



#### 哈希索引

对字段值的散列进行索引

只支持相等匹配，不支持基于范围的查询

默认 _id 索引，不可删除

唯一索引，值不能重复

分片集群中，通常使用 _id 作为片键

### 命令

#### 查看索引

```
db.collection.getIndexes()
```

#### 创建索引

```
db.comment.createIndex({userid:1,nickname:-1})
```

移除索引

```
 db.comment.dropIndex({userid:1})
 db.collection.dropIndexes()
```

