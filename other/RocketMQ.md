## Quick start

1. [下载](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.3.0/rocketmq-all-4.3.0-bin-release.zip)

在 ~/.bash_profile 配置

`export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home`

并使之生效 `source ~/.bash_profile`。

2. 启动 nameserver

```bash
nohup sh bin/mqnamesrv &
tail -f ~/logs/rocketmqlogs/namesrv.log
```

3. 启动broker

```bash
nohup sh bin/mqbroker -n localhost:9876 &
tail -f ~/logs/rocketmqlogs/broker.log
```

4. 收发消息demo

先配置 ~/.bash_profile

```bash
export NAMESRV_ADDR=localhost:9876
source ~/.bash_profile
```

发消息

```bash
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

收消息

```bash
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

5. 关闭server

```bash
sh bin/mqshutdown broker
sh bin/mqshutdown namesrv
```

## Simple Message Example

RocketMQ 发消息的三种方式：

- reliable synchronous
- reliable asynchronous
- one-way

## Order Message

RocketMQ 支持有序的消息，这个顺序是 FIFO

## Architecture

**NameServer Cluster**

服务发现和路由

- 接收Broker的注册并通过心跳机制来检查Broker是否存活
- 每个nameserver都包含Broker集群的全部路由信息，以及clients的查询队列

**Broker Cluster**

消息存储。有 Topic 和 Queue 两种机制。支持 push/pull 模型。有容错机制...


## 参考

- http://rocketmq.apache.org/
