## gRPC

来自 google，开源框架

RPC 一种非常简洁的实现，解决了 很多 RPC 的问题

允许为 RPC(Remote Procedure Call)，定义请求和响应，处理一切剩余问题

速度快，效率高

基于HTTP/2构建，低延迟，支持流，与开发语言无关

可很简单的插入身份认证、负载均衡、日志和监控等功能

![image-20220228151016906](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220228151016906.png)



## Protocol Buffer

## gRPC原理

![image-20220228181847099](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220228181847099.png)

### 步骤

#### 定义消息

#### 生成代码

#### 开发Client/Server

## 生命周期

创建隧道

创建Client

Client 发送请求

Server 发送

发送/接收消息

## 身份认证

### 不认证， 不安全连接

### TLS/SSL连接

### 基于Goole Token 的身份认证

### 自定义的身份认证提供商

## 消息传输类型

### 一元消息

一个请求，一个响应

rpc 方法名（请求类型） returns （响应类型）

![image-20220228190904965](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220228190904965.png)

### server streaming （流）

server把数据streaming 回给client

每次返回部分数据

rpc 方法名（请求类型） returns （streaming 响应类型）

流式视频

### client streaming 

client 把数据streaming 回给 server

server 保持等待，直到接收完所有数据

rpc 方法名（streaming 请求类型） returns （响应类型）

上传数据

### 双向streaming

rpc 方法名（streaming 请求类型） returns （streaming 响应类型）

## 消息类型

当 gRPC 使用Protocol Buffer 作为传输协议的时候，Protocol Buffer 里所有的规则仍然适用

会添加一些额外的规则和语法，以便让 gRPC 能和它完美配合

### Service关键字

```
service Employee{

}
service Employee{
	rpc GetByName(Request) returns (Response)
}
```

## .NET Core-gRPC Server

Grpc.AspNetCore

## .NET Core-gRPC Client

依赖包

Grpc.Net.Client、Google.Protobuf、Grpc.Tools

