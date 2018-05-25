# rocketmq

### 前置条件
假定安装了以下软件:

- 推荐64bit OS, Linux/Unix/Mac系统;
- 64bit JDK 1.8+;
- Maven 3.2.x
- Git


### 从发布版下载并构建
点击 [这里](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip) 下载4.2.0发行版源代码. 你也可以点击 [这里](http://rocketmq.apache.org/release_notes/release-notes-4.2.0/)下载二进制发行版.

现在执行以下命令来解压4.2.0源版本并构建.
~~~bash
  > unzip rocketmq-all-4.2.0-source-release.zip
  > cd rocketmq-all-4.2.0/
  > mvn -Prelease-all -DskipTests clean install -U
  > cd distribution/target/apache-rocketmq
~~~ 
### 启动 Name Server服务
~~~bash
  > nohup sh bin/mqnamesrv &
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
~~~
### 启动Broker
~~~bash
  > nohup sh bin/mqbroker -n localhost:9876 &
  > tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
~~~ 
###  发送和接收消息
在发送/接收消息之前，我们需要告诉客户端服务器的位置。 RocketMQ提供了多种方式来实现这一点。 为了简单起见，我们使用环境变量NAMESRV_ADDR
~~~bash
 > export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
 ~~~
### 关闭服务
~~~bash
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
~~~