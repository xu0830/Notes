## Sql Server

## 组件

![image-20220322101340558](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220322101340558.png)

### 缓冲池 Buffer Pool

### 数据文件 mdf

### 事务日志 ldf

### 关系引擎

#### 命令解析器

#### 查询执行器

#### 查询优化器

### 存储引擎

#### 事务管理器

#### 数据访问方法

#### 缓冲区管理器

### 网络接口

## 数据分页

DataPage

IAMPage

8kb/page 任何一条数据不能跨页存储： 数据长度不能超过8096，varchar最长varchar（8096）

text 存储超过8kb，会存在另外一个page，数据页只存储位置，效率低

# Lucene

全文检索工具包

把数据拆分

不规则的数据拆分成词存储

