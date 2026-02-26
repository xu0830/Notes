# docker

```
yum update
#yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum makecache

yum install docker-ce

yum remove docker-ce docker-ce-selinux container-selinux docker docker-ce-cli  -y 

#Linux安装docker 
yum -y install docker-ce

#构建镜像
docker build -t net6v0305（镜像名）:1740（tags 名称） -f Dockerfile .

#创建并运行容器
docker run -itd -p 8082:80 [--rm] --name docker8082 net6v0305:1740

docker create 

docker start

#	拉取最新镜像
docker pull nginx:latest

#	挂载单独的配置文件nginx。conf
docker run -d -p 8085:80 -v /home/tools/nginx:/var/log/nginx -v /home/tools/nginx/nginx.conf:/etc/nginx/nginx.conf --name mynginx nginx 

docker ps -a 

docker rm -f $(docker ps -aq)

docker rmi -f $(docker images -q)   

docker stop $(docker ps -q) & docker rm [-f(强制删除)] $(docker ps -aq) 

docker exec -it mysql bash

docker login -u xucanjie -p xucanjie88!?

docker container update [OPTIONS] [容器名称]

#创建集群
docker swarm init --advertise-addr 192.168.231.129

#介入节点
docker swarm join --token SWMTKN-1-1ajuy75ene7t3p0rc1m2ju0swqjj0pqaadu0rmuhzuj8fqwnen-5hlbxh1xofv34movm4l2zpwia 192.168.231.129:2377

docker service create -p 8082:80 --name mydotnet6 xucanjie/net6v0305:v1

systemctl daemon-reload

查看所有节点
docker node ls

扩容缩容
docker service update --replicas=4 mydotnet6
docker service update --replicas=1 mydotnet6

docker run --restart=always -d --name mysql -v productdata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -e bind-address=0.0.0.0 mysql:8.0

docker run -itd --name myredis -p 6379:6379 redis

#卷
/var/lib/docker/volumes/


```

### docker image 

极度精简的 Linux 程序运行环境

1、可定制的”安装包“

2、不建议有运行时需要修改的配置文件（redis-ngnix）

3、尽量重用 docker hub 的基础镜像

### 镜像

包含应用程序所需的模板文件

### 容器

由镜像创建的应用程序实例，彼此隔离

### 为什么会流行？

镜像和容器是 docker 的关键构建模板

### 缺点

复杂的应用程序，编写 dockerfile 文件、创建镜像、测试，耗时耗力

### redis sentinel

```
主从复制
1、获取配置文件
wget http://download.redis.io/redis-stable/redis.conf
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1 

2、配置 master
# 关闭保护模式
protected-mode no 

# 让redis服务后台运行
daemonize yes 

# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码. 只是练习配置，就不使用密码认证了)
# requirepass masterpassword 

# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"


3、配置 slave
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1 

# 关闭保护模式
protected-mode no 

# 让redis服务后台运行
daemonize yes 

# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码)
requirepass masterpassword 

# 设定主库的密码，用于认证，如果主库开启了requirepass选项这里就必须填相应的密码
masterauth <master-password>

# 设定master的IP和端口号，redis配置文件中的默认端口号是6379
# 低版本的redis这里会是slaveof，意思是一样的，因为slave是比较敏感的词汇，所以在redis后面的版本中不在使用slave的概念，取而代之的是replica
# 将35.236.172.131做为主，其余两台机器做从。ip和端口号按照机器和配置做相应修改。
replicaof 35.236.172.131 6379

# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"

4、启动容器
docker run -it --name redis-3 -v /root/redis.conf:/usr/local/etc/redis/redis.conf -d -p 6379:6379 redis

docker exec -it redis-3 bash

slaveof masterip port 

5、添加sentinel
获取配置文件
wget http://download.redis.io/redis-stable/sentinel.conf


# 让sentinel服务后台运行
daemonize yes 

# 修改日志文件的路径
logfile "/var/log/redis/sentinel.log"

# 修改监控的主redis服务器
# 最后一个2表示，两台机器判定主被动下线后，就进行failover(故障转移)
sentinel monitor mymaster 35.236.172.131 6379 2


docker run -it --name sentinel -p 26379:26379 -v /root/sentinel.conf:/usr/local/etc/redis/sentinel.conf -d redis

docker exec -it sentinel bash

mkdir /var/log/redis
touch /var/log/redis/sentinel.log

#启动哨兵
redis-sentinel /usr/local/etc/redis/sentinel.conf 
```

