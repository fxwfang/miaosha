## docker 
* redis 安装
```
docker run -d --name redis -p 6379:6379 redis:latest redis-server --appendonly yes --requirepass "password"

docker run -d 后台运行
--name redis  服务名称
-p 6379:6379  将容器6379映射到主机的6379端口
redis-server --appendonly yes 在容器执行redis-server启动命令，并打开redis持久化配置
--request "youpass" 设置密码

```
* 连接redis
```
docker exec -it '容器ID' redis-cli

~ % docker exec -it '9803a' redis-cli
127.0.0.1:6379> dbsize
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth 123456

```

* Redis 数据映射

```java
1. redis镜像
https://hub.docker.com/_/redis
https://zhuanlan.zhihu.com/p/70110697

/*
有问题：
docker run -d --name redis -p 6379:6379  redis-server --appendonly yes -v /Users/fangxiaowei/software/redis/data:/data  -v /Users/fangxiaowei/software/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf redis:latest --requirepass "123456"
*/

// 能正确运行： https://www.jianshu.com/p/2f95680f21c5
docker run -d -p 6379:6379 -v $PWD/conf/redis.conf:/usr/local/etc/redis/redis.conf -v $PWD/data:/data --name docker-redis redis:latest  redis-server /usr/local/etc/redis/redis.conf  --appendonly yes --requirepass "123456" 

```



redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool:

# redis JedisConnectionException: Could not get a resource from the pool 的八种可能的原因:

https://blog.csdn.net/testcs_dn/article/details/43052585

### Zookeeper

* [docker inspect命令查看镜像详细信息](https://www.cnblogs.com/carriezhangyan/p/10845697.html)
* 安装 https://www.cnblogs.com/LUA123/p/11428113.html
* 

### mysql

docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

# ava.sql.SQLException: Unable to load authentication plugin ‘caching_sha2_password‘.避坑指南:

https://blog.csdn.net/w605283073/article/details/88096598



Access denied for user ‘root’@‘172.17.0.2’ (using password: YES):

https://blog.csdn.net/qq_40020447/article/details/105680283

# 新版SQL授权用户时报错 near 'IDENTIFIED BY '密码' with grant option' at line 1:

https://blog.csdn.net/vin_1991216/article/details/82632710





### Rabbitmq

docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:3-management

访问

http://localhost:15672/

默认用户名和密码为`guest`/ `guest`



### org.springframework.amqp.rabbit.listener.BlockingQueueConsumer$DeclarationException: Failed to declare queue(s):[miaosha.queue]

https://blog.csdn.net/ouzhuangzhuang/article/details/84839104



# 记一次使用Jedis客户端获取不到资源（Could not get a resource from the pool）的填坑经历

https://blog.csdn.net/qq_21583077/article/details/88225728

这次的报错主要原因： com.geekq.miaosha.redis.redismanager 的IP配置有问题


### 坑 在Docker容器创建好之后，可能会发现容器时间跟宿主机时间不一致，此时需要同步它们的时间，让容器时间跟宿主机时间保持一致。
https://www.cnblogs.com/qinlangsky/p/11698978.html
主要式程序运行的时候，秒杀时间和自己存到数据库到时间不一致，最后尝试用now()插入时间，发现时间
日期跟系统时间对不上，想到自己是用docker环境，就明白是docker时间有问题

/etc/localtime 只是一个link文件，还需要把真正的文件拷贝到容器里面
a9990fdc2d:/# mkdir -vp /var/db/timezone/zoneinfo/Asia
$ docker cp /var/db/timezone/zoneinfo/Asia/Shanghai 1ea9990fdc2d:/var/db/timezone/zoneinfo/Asia/Shanghai

### 解决数据库时间相差14小时
docker时间与宿主机时间一致，mysql的now()入库也是对的，但是查询出来的时间却相差14个小时。
修改配置如下：
```
# com.mysql.jdbc.Driver 是 mysql-connector-java 5中的；
# com.mysql.cj.jdbc.Driver 是 mysql-connector-java 6中的,需要设置时区。
datasource:
  driver-class-name: com.mysql.cj.jdbc.Driver
  url: jdbc:mysql://ip:port/drgs-engine?
  useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai
```