# RabbitMQ

docker

docker run --restart=always -d --hostname my-rabbit --name ecomm-rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management



![image-20220423122019363](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220423122019363.png)



## Broker

接收和分发消息的应用，RabbitMQ Server 就是 Message Broker

## Virtual Host

出于多租户和安全因素设计，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ Server 提供的服务，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange / queue

## Connection 

publisher / consumer 和 broker 之间的 TCP 连接

## Channel

如果每次访问 RabbitMQ 都建立在一个 connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率低。

Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。

作为轻量级的 connection 极大减少了操作系统建立 TCP connection 的开销

## Exchange

message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key ，分发消息到 queue 中去。常用的类型有 direct、topic、fanout

## Queue

消息最终被送到这里等待 consumer取走

## Binding

exchange 和 queue 之间的虚拟连接，binding中包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

## 交换机

Exchanges

生产者生产的消息从不会直接发送到队列，只能将消息发送到交换机

实际上，通常生产者甚至都不知道这些消息传递到哪些队列中

一方面接收来自生产者的消息，另一方面将它们推入队列

交换机必须确切知道如何处理收到的消息

由交换机的类型决定，应该把这些消息放到哪些队列还是丢弃

### 类型

一个队列绑定键是 #， 这个队列将接收所有数据，等同于 fanout

队列绑定键当中没有 # 和 * 出现，等同于 direct 

#### 直接

direct

完全匹配、单播模式

路由键与队列名完全匹配

消息中的路由键（routing key）如果和Binding中的binding key 一致，交换器就将消息发送到对应的队列中

#### 主题

topic

将路由键和绑定键的字符串切分成单词，这些单词之间用.点间隔开

识别通配符

"#"  匹配 0 个或多个单词

"*"  匹配一个单词

单词列表最多超过 255 个字节 

#### 扇出

fanout

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上

不处理 routing key，只是简单的将队列绑定到交换器上

转发消息是最快的 

#### 标题（几乎不用）

headers

## 模式

### 简单模式 

Hello World



### 工作队列 

Work queues

避免立即执行资源密集型任务，而不得不等待它完成。

安排任务在之后执行。把任务封装为消息并将其发送到队列

在后台运行的工作进程将弹出任务并最终执行作业

当有多个工作线程时，这些工作线程将一起处理这些任务

#### 轮询分发消息

#### 消息应答

##### 自动应答

消息发送后立即被认为已经传送成功

这种模式需要在高吞吐量和数据传输安全性方面做权衡

没有对传递的消息数量进行限制

仅适用于消费者可以高效处理消息的情况下

##### 手动应答

###### 消息自动重新入队

如果消费者由于某些原因失去连接（通道已关闭，连接已关闭或 TCP 连接丢失），导致消息未发送 ACK 确认

RabbitMQ 自动将消息重新排队

### 发布/订阅 

Publish/Subscribe



### 路由 

Routing 



### 主题 

Topics



### 发布确认 

Publisher Confirms

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都会被指派一个唯一的 ID （从 1 开始），一旦消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者（包含消息的唯一 ID），这就使得生产者知道消息已经正确到达目的队列

如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列化，此外 broker 也可以设置 basic，ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理

confirm 模式最大的好处在于异步，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果 rabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用同样可以在回调方法中处理该 nack 消息

#### 单个确认发布

发布一个消息之后被确认发布，后续的消息才能继续发布

在指定时间范围内这个消息没有被确认将抛出异常

缺点： 发布速度特别慢

#### 批量确认发布



#### 异步确认发布 

回调函数，监听发布消息的成功、失败事件

##### 如何处理异步未确认消息

每次发布的消息保存在记录集合中，在消息确认成功的回调函数删除记录，

发送结束后集合中存在的记录就是未确认消息

#### 回退消息

Mandatory 参数 true

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息

如果该消息不可路由，那么消息会被直接丢弃，生产者是不知道消息被丢弃的

```c#
BasicPublish(mandatory: true);
channel.BasicReturn += (sender, ea) => {
	Console.WriteLine("message return");
};
```



#### 备份交换机

当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理

通常备份交换机的类型为 fanout，这样就能把所有消息都投递到与其绑定的队列中

![image-20220424155026948](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220424155026948.png)

（回退消息）mandatory 参数与备份交换机两者同时开启，备份交换机的优先级更高

## 运维

### 内存配置

配置使用的最大内存，/etc/rabbitmq/rabbitmq.conf 

相对值 vm_memory_high_watermark.relative，所在物理机器的内存比例，默认0.4，建议配置为0.4-0.7间

绝对值 vm_memory_high_watermark.absolute：所在物理机器的内存的绝对量

```
vm_memory_high_watermark.relative = 0.4
vm_memory_high_watermark.absolute = 2GB
```

默认情况下，采用相对值配置，默认 vm_memory_high_watermark.relative = 0.4

### 内存换页

rabbitmq 使用的内存量快达到配置的极限前，会尝试将队列中的消息从内存中换页到磁盘以释放内存空间，持久和非持久的消息都会换页到磁盘中，其中持久化的消息本身在磁盘就有一个副本，所以在换页转移的过程中，持久化的消息会先从内存中被清除

配置项为 vm_memory_high_watermark_paging_ratio，当使用的内存量达到配置上限的多少比例时，将同等比例内存量的消息换页到磁盘中，默认值 0.5

假设rabbitmq所在物理机器的内存为1000MB，vm_memory_high_watermark.relative=0.4，vm_momery_high_watermark_paging_ratio=0.5。那么当rabbitmq使用的内存量达到400MB前，就会将200MB的消息从内存换页到磁盘中


## TTL

一条消息或者该队列中所有的消息的最大存活时间

一条消息设置了 TTL 属性，或者进入了设置了 TTL 属性的队列，那么这条消息在 TTL 设置时间内没有被消费，则会成为死信

如果同时设置了队列的 TTL 和消息的 TTL ，那么会使用较小的值

不设置 TTL ，表示消息永远不会过期

TTL 设置为 0，表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃

- 消息设置TTL

  消息过期，不一定会马上被丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间

- 队列设置TTL

  一旦消息过期，就会被队列丢弃，如果配置了死信队列就丢到死信队列中

## 持久化机制

### 交换机持久化

### 队列持久化

### 消息持久化

### 配置文件刷盘策略

"rabbitmq.conf"

disk_free_limit.relative = 3.0

disk_free_limit.absolute = 50MB

queue_index_embed_msgs_below = 4096  # 小于4096字节的消息直接嵌入索引文件

```
rabbitmq-disk-monitor --threshold 0.8 # 磁盘使用超过80%告警
```

## 死信队列

死信，无法被消费的消息

由于某些原因导致队列中的某些消息无法被消费，这样的消息如果没有后续的处理就变成了死信，

有死信自然就有了死信队列

应用场景：为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制

当消费发生异常，将消息投入死信队列中

### 死信的来源

1、消息被消费方否定确认（channel.basicNack或channel.basicReject），并且requeue属性被设置为false

2、消息在队列中的存活时间超过设置的TTL时间（队列中消息的最大存活时间），同时配置了队列的TTL和消息的消息的TTL，较小的值将会被使用

3、消息队列的消息数量已经超过最大队列长度

该消息将成为“死信”，如果配置了死信队列，该消息会被丢进死信队列中，无配置则丢弃

为需要使用死信的业务队列配置一个死信交换机，死信交换机可以共用一个

死信队列是绑定在死信交换机的队列
```
channel.exchangeDeclare("dlx.exchange", "topic", true, true, false, null); // 声明死信交换机
channel.queueDeclare("dlx.queue", true, false, null); // 声明死信队列
channel.queueBind("dlx.queue", "dlx.exchange", "#");    //    死信交换机绑定死信队列

arguments.put("x-dead-letter-exchange", "dlx.exchange");
channel.queueDeclare(queueName, true, false, false, arguments); // 声明业务队列，绑定死信交换机
channel.queueBind(queueName, exchangeName, routingKey);
```

## 延迟队列

队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理。

就是用来存放需要在指定时间被处理的 元素的队列

### 使用场景

- 订单在十分钟内未支付则自动取消
- 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒
- 用户注册成功，三天内没有登录则进行短信提醒
- 用户发起退款，三天内没有得到处理则通知相关人员
- 预定会议后，需要在预定的时间点前10分钟提醒与会人员

### 实现原理

#### 基于死信队列实现

routing key 路由到不同的TTL队列实现延迟

TTL 不应该绑定到队列上，而应该由生产者发送消息时设置，只用一个队列实现不同延迟效果

不需要因为延时时间不同就创建一个队列

```
Map<String, Object> args = new HashMap<>();
args.put("x-delayed-type", "direct");    // 定义交换机类型 direct 、fanout、topic
channel.exchangeDeclare("delayed_exchange", "x-delayed-message", true, false, args);
// 发送延迟消息
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .headers(new HashMap<String, Object>() {{ put("x-delay", 5000); }}) // 延迟5秒
    .build();
channel.basicPublish("delayed_exchange", "routing_key", props, message.getBytes());
```

##### 存在问题

如果延迟队列中第一个消息延时时间很长，第二个消息延时时间很短，第二个消息并不会优先得到执行

按照消息顺序执行

#### 解决方案

基于 **rabbitmq_delayed_message_exchange** 插件

bin目录下启动命令

```
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

按照插件后，增加了交换机类型 x-delayed-message

TTL 的判定由投递到交换机开始

##   幂等性

### 消息重复消费

消费者返回 ack 时网络中断，MQ 未收到确认消息

#### 解决思路

一般使用全局 ID 或者唯一标识比如时间戳，每次消费消息时用该 id 先判断该消息是否已消费过

### 消费端的幂等性保障

订单生成的业务高峰期，生产端有可能就会重复发送消息，这时消费端就需要实现幂等性

- 唯一 ID + 指纹码机制，利用数据库主键去重
- 利用 redis 的原子性去实现

### 唯一 ID + 指纹码机制

业务规则或者时间戳生成的唯一信息码

查询这个 id 是否存在数据库中，优势是简单

劣势是高并发时，单个数据库会有写入性能瓶颈，需要采用分库分表提升性能

不是最推荐的方式

#### Redis 原子性

利用 redis 执行 setnx 命令，天然具有幂等性

## 优先级队列

大客户优先处理

0-255 越大越优先执行

**队列需要设置优先级队列 x-max-priority 队列最大优先级**

**消息需要设置消息的优先级 proprity**

消费者需要等待消息已经发送到队列中，队列中的消息才能进行排序

## 惰性队列

会尽可能的将消息存入磁盘中，而在消息者消费到相应的消息时才会被加载到内存中

一个重要的设计目标是能够支持更多的消息存储

当消费者由于各种各样的原因（消费者下线，宕机等）而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了

默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。

当 RabbitMQ 需要释放内存的时候，会将内存中的消息持久化至磁盘中，这个操作会耗费较长的时间也会阻塞队列的操作，进而无法接收新的消息

### 两种模式

x-queue-mode

default 和 lazy

发送一百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2 GB，惰性队列仅占用 1.5 MB

## 集群原理

1. 修改机器名 

   ```
   vim /etc/hostname
   ```

2. 配置各个节点的 hosts 文件，让各个节点能互相识别对方

   ```
   vim /etc/hosts
   	192.168.231.129 node1
       192.168.231.130 node2
       192.168.231.131 node3
   ```

3. 确保各个节点的 cookie 文件使用的是同一个值

   在node1执行命令

   ```
   scp /var/lib/rabbitmq/.erlang.cookie root@node2:/var/lib/rabbitmq/.erlang.cookie
   scp /var/lib/rabbitmq/.erlang.cookie root@node3:/var/lib/rabbitmq/.erlang.cookie
   ```

4. 重新启动 RabbitMQ 服务和 Erlang 虚拟机，在三台节点分别执行以下命令

   ```
   rabbitmq-server -detached
   ```

5. 在节点 2 执行

   ```
   rabbitmqctl stop_app
   (rabbitmqctl stop 会将 Erlang 虚拟机关闭，rabbitmqctl stop_app 只关闭 RabbitMQ 服务)
   rabbitmqctl reset
   rabbitmqctl join_cluster rabbit@node1
   rabbitmqctl start_app(只启动应用服务)
   ```

6. 在节点3执行

   ```
   rabbitmqctl stop_app
   rabbitmqctl reset
   rabbitmqctl join_cluster rabbit@node2
   rabbitmqctl start_app
   ```

7. 查看集群状态

   ```
   rabbitmqctl cluster_status
   ```

8. 需要重新设置用户

   ```
   创建账号
   rabbitmqctl add_user admin 123
   设置用户角色
   rabbitmqctl set_user_tags admin administrator
   设置用户权限
   rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
   ```

9. 解除集群节点

   ```
   node2 和 node3 机器分别执行
   rabbitmqctl stop_app
   rabbitmqctl reset
   rabbitmqctl start_app
   rabbitmqctl cluster_status
   
   node1 执行
   rabbitmqctl forget_cluster_node rabbit@node2
   ```

## 镜像队列

1. 启动三台集群节点

2. 随便找一个节点添加policy

## Haproxy + Keepalived 高可用负载均衡

![image-20220424210912730](C:\Users\xucan\AppData\Roaming\Typora\typora-user-images\image-20220424210912730.png)

###  Haproxy 实现负载均衡

1. 下载 haproxy （在 node1 和 node2）

   yum -y install haproxy

2. 修改 node1 和 node2 的 haproxy.cfg

   vim /etc/haproxy/haproxy.cfg

   需要修改红色 IP 为当前机器 IP

3. 在两台节点启动 haproxy

   haproxy -f /etc/haproxy/haproxy.cfg

   ps -ef | grep haproxy

4. 访问地址

### Keeplilved 实现双机（主备）热备

1.下载 keepalived

yum -y install keepalived

2.节点 node1 配置文件

vim /etc/keepalived/keepalived.conf

把资料里面的 keepalived.conf 修改之后替换

3.节点 node2 配置文件

需要修改 global_defs 的 router_id,如:nodeB

其次要修改 vrrp_instance_VI 中 state 为"BACKUP"；

最后要将 priority 设置为小于 100 的值

4.添加 haproxy_chk.sh

(为了防止 HAProxy 服务挂掉之后 Keepalived 还在正常工作而没有切换到 Backup 上，所以这里需要编写一个脚本来检测 HAProxy 务的状态,当 HAProxy 服务挂掉之后该脚本会自动重启

HAProxy 的服务，如果不成功则关闭 Keepalived 服务，这样便可以切换到 Backup 继续工作)

vim /etc/keepalived/haproxy_chk.sh(可以直接上传文件)

修改权限 chmod 777 /etc/keepalived/haproxy_chk.sh

5.启动 keepalive 命令(node1 和 node2 启动)

systemctl start keepalived

6.观察 Keepalived 的日志

tail -f /var/log/messages -n 200

7.观察最新添加的 vip

ip add show

8.node1 模拟 keepalived 关闭状态

systemctl stop keepalived 

9.使用 vip 地址来访问 rabbitmq 集群

## Federation Exchange

数据转发

地域网络延迟，不同地域间的数据同步

配置交换机策略，设置上下游节点关系

1. 需要保证每台节点单独运行

2. 在每台机器上开启 federation 相关从插件

   rabbitmq-plugins enable rabbitmq_federation

   rabbitmq-plugins enable rabbitmq_federation_management
   
   ```
   rabbitmqctl set_parameter federation-upstream f1 '{"uri":"amqp://guest:guest@192.168.231.1:5675","ack-mode":"on-confirm"}'
   
   ```
   
  
## Federation Queue

配置队列策略，设置上下游节点关系

添加policy

## Shovel

 与 federation 类似，持续地从一个 Broker 中的队列拉取数据并转发至另一个 Broker 中的交换器

 作为源端的队列和作为目的端交换器可以同时位于同一个 Broker，也可以位于不同的 Broker 上

1.开启插件(需要的机器都开启)

rabbitmq-plugins enable rabbitmq_shovel

rabbitmq-plugins enable rabbitmq_shovel_management


## 确保消息发送、接收

### 发送方确认机制

信道需要设置为confirm模式，在信道发布的消息都会分配一个唯一 ID

消息投递到队列（**持久化的消息deliveryMode=2需要确认写入磁盘**）后，信道会发送一个确认给生产者（包含消息的唯一 ID）

如果RabbitMQ发生内部错误导致消息丢失，会发送一条nack消息给生产者

发送方确认模式是异步的，等待确认的同时可以继续发送消息，确认消息到达生产者后，触发ack或nack回调

#### 交换机没找到匹配的队列

##### Direct/Fanout/Topic/Headers 交换机

返回basic.ack（确认成功），但消息会被静默丢弃，RabbitMQ 认为消息已成功路由到交换机

```
channel.confirmSelect();
channel.basicPublish("", "non_existent_queue", null, message.getBytes());
channel.addConfirmListener((deliveryTag, multiple) -> {
    System.out.println("消息被ACK，但实际未进入队列！");
}, (deliveryTag, multiple) -> {
    System.out.println("消息被NACK");
});
```
#####  强制标志（Mandatory）

设置 mandatory = true，强制要求消息必须被路由到至少一个队列，否则触发 ReturnListener

```
channel.confirmSelect();    //    启用确认模式
channel.addReturnListener((replyCode, replyText, exchange, routingKey, properties, body) -> {
    System.out.println("消息未被路由到队列！原因: " + replyText);
});
// 发布消息时设置 mandatory=true
channel.basicPublish("exchange", "unroutable_key", true, null, message.getBytes());
```

### 消费方确认机制

消息者在声明队列时，可以指定noAck参数

noAck=false，RabbitMQ会等待消费者返回ack信号后才删除消息

noAck=true,消息被消费后会立即删除

RabbitMQ 不会为未ack的消息设置超时时间

判断消息是否需要重新投递的唯一依据是该消费者的连接是否断开

如果消费者返回ack之前断开了连接，RabbitMQ会重新分发给下一个消费者（可能存在消息重复消费的隐患，需要去重）


