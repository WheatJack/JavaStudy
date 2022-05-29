## RobbitMq学习

>https://www.bilibili.com/video/BV1cf4y157sz?spm_id_from=333.337.search-card.all.click
>
>MQ用途：限流削峰、异步解耦、数据收集（业务日志、监控数据、用户行为等）



## 分布式消息队列RocketMQ



## 1、基本概念

### 消息 Message

消息是指：消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须数据一个主题



### 主题 Topic

Topic表示一类消息的集合，每个主题包含了若干消息，每条消息只能属于一个主题，是RocketMq进行消息订阅的基本单位。

Topic:message	1:N		message:Topic	 1:1

一个生产者可以同时发送多种Topic 的消息；而以恶搞消费者只能对某种特定的Topic感兴趣，即只可以订阅和消费一种Topic的消息



### 标签 Tag

为消息设置的标签，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同的业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMq提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。



### 队列 Queue

存储消息的物理实体，一个Topic可以包含多个Queue，每个Queue中存放的就是该Topic的消息，一个Topic的Queue也被称为一个Topic中的消息的分区（Partition）

一个Topic的Queue中的消息只能被一个消费组中的一个消费者消费。一个Queue中的消息不允许同一个消费者中的多个消费者同时消费

<img src="/Users/gaoshang/Library/Application Support/typora-user-images/image-20220502164235484.png" alt="image-20220502164235484" style="zoom: 75%;" />



### 消息标识 MessageId/Key

RocketMQ中每个消息拥有唯一的MessageId，且可以携带具有业务标识的Key，以方便对消息的查询。不过需要注意的是，MessageId有两个：在生产者send() 消息时会自动生成一个MessageId（msgId），都消息到达Broker后，Broker也会自动生成一个MessageId（offsetMsgId）。MsgId、offsetMsgId与Key都称为消息标识。

* **msgId**：由producer端生成，其生成规则为：producerIp + 进程pid + MessageClientIdSetter类的calssLoader的hashcode + 当前时间 + AutonomicIntger自增计数器
* **offsetMsgId**：由broker端生成，其生成规则为：brokerIp+物理分区的offset（Queue中的偏移量）
* **key**：由用户指定的业务相关的唯一标识



## 2、系统架构

![image-20220502170033984](https://tva1.sinaimg.cn/large/e6c9d24egy1h1u615dfxoj20mb098mxn.jpg)



### Producer

消息生产者，负责生产消息。producer通过MQ的负载均衡模块选择相应的Broker集群队列中进行消息投递，投递的过程支持快速失败并且低延迟。

RocketMQ中的消息生产者都是以生产者组（producer Group）的形式出现的。生产者组是同一类生产者的集合，这类Producer发送相同的Topic类型的消息，一个生产者组可以同时发送多个主题的消息。



### Consumer

消息消费者，负责消费消息。一个消息消费者会从Broker服务器中获取到消息，并对消息进行相关业务处理。

RocketMq中的消息消费者都是以消费者组（Consumer Group的形式出现的）消费者是同一类消费者的集合，这类Consumer消费的是同一个Topic类型的消息。消费者组使得在消息消费方面，实现负载均衡（**对于Queue来说，不是对消息进行负载均衡**）和容错的目标变得非常容易。

![image-20220502171820367](https://tva1.sinaimg.cn/large/e6c9d24egy1h1u6jmrvjqj20mc0923z8.jpg)

 消费者组中的Consumer的数量应该小于等于订阅Topic的Queue数量。如果超出Queue的数量，则多处的Consumer将不能消费信息。

![image-20220502172023912](https://tva1.sinaimg.cn/large/e6c9d24egy1h1u6lrgx55j20m50aj3zh.jpg)

不过，一个Topic类型的消息可以被多个消费者组同时消费

>1）消费者组只能消费一个Topic的消息，不能同时消费多个Topic消息
>
>2）一个消费者组中的消费者必须订阅完全相同的Topic



### Name Server

#### 功能介绍

NameServer是一个Broker与Topic路由的注册中心，支持Broker的动态注册与发现。

主要包括两个功能：

* **Broker管理**：接受Broker集群的注册信息并且保存下来作为路由信息的基本信息；提供心跳检测机制，检查Broker是否还存活
* **路由信息管理**：每个NameServer中都保存着Broker集群的整个路由信息和用于客户端查询的队列信息。producer和consumer通过NameServer可以获取整个Broker集群的路由信息，从而进行消息的投递和消费。



#### 路由注册

NameServer通常也是以集群的方式部署，不过NameServer是无状态的，即NameServer集群中的各个节点是无差异的。各节点间相互不进行信息通讯。那各节点中的数据是如何进行数据同步的呢？在Broker节点启动时，轮训NameServer列表，与每个NameServer节点建立长连接，发起注册请求。在NameServer内部维护着一个Broker列表，用来动态存储Broker的信息。

>注意，这里与其其他的ZK、Eureka、Nacos等注册中心不同的地方。
>
>这种NameServer的无状态方式，有什么优缺点：
>
>优点：NameServer集群搭建简单，扩容简单
>
>缺点：对于Broker，必须明确指出所有的NameServer地址。否则未指出的将不会去注册。也正因为如此，NameServer并不能随便扩容。因为，若Broker不重新配置，新增的NameServer对于Broker来说是不可见的，其不会向这个NameServer进行注册。

Broker节点为了证明自己是活着的，为了维护与NameServer间的长连接，会将最新的信息以心跳包的形式上报给NameServer，每30s发送一次心跳。心跳包中包含BrokenId、Broker地址（Ip+port）、Broker名称、Broker所属集群名称等等。NameServer在接收到心跳包后，会更新心跳时间戳，记录这个Broker的最新存活时间。



#### 路由剔除

由于Broker关机、宕机、网络抖动等原因，NameServer没有收到Broker的心跳，NameServer可能会将其从Broker列表中剔除。

NameServer中有一个定时任务，每隔10S就会扫描一次Broker表，查看每一个Broker的最新心跳时间戳距离当前时间是否超过120S，如果超过，则会判定Broker失效，然后将其从Broker列表中剔除。

>扩展：对于RocketMq日常运维工作，例如Broker升级，需要停掉Broker的工作，OP（运维）需要怎么做？
>
>OP需要将Broker的读写权限禁用，一旦client（consumer、producer）想broker发送请求，都会收到broker的No_permission响应，然后client会进行对其他Broker的重试。
>
>当OP观察到这个Broker没有流量后，再关闭它，实现Broker从NameServer的移除。



#### 路由发现

RocketMq的路由发现采用的是Pull模型。当Topic路由信息出现变化时，NameServer不会主动推送给客户端，而是客户端定时拉取主题最新的路由。默认客户端每30s会拉取一次最新的路由。

>扩展：
>
>**1）Push模型：**推送模型，其实效性较好，是一个“发布-订阅”模型，其需要维护一个长连接。而长连接的维护是需要资源成本的，该模型适合于的场景：
>
>* 实时性要求较高
>* Client的数量不多，Server数据变化频繁
>
>**2）Pull模型：**拉取模型。存在的问题是，实时性较差。
>
>**3）Long Polling模型：**长轮询模型（Nacos）。是对Push和Pull模型的整合，充分利用了这两种模型的优势，屏蔽了它们的劣势
>
>*保持长连接，但是持续的时间不长。会持续尝试Pull*



#### 客户端NameServer选择策略

>这里的客户端指的是Producer与Consumer

客户端在配置时必须协商NameServer集群的地址，那么客户端到底连接的是哪个NameServer节点呢？客户端首先会产生一个随机数，然后再与NameServer节点数量取模，此时得到的就是所要连接的节点索引，然后就会进行连接。如果连接失败，则会采用round-robin策略，逐个尝试着去连接其他节点。

首先采用的是***随机策略***进行的选择，失败后采用的是***轮询策略***。

>扩展：zookeeper client是如何选择zookeeper server的？？？
>
>简单来说就是，经过两次Shuffle（随机），然后选择第一台zookeeper server
>
>详细说就是，将配置文件中的zookeeper server的地址进行第一次shuffle，然后再随机选择一个。这个选择出的一般都是一个hostname。然后获取到该hostname对应的所有的IP，在对这些IP进行第二次shuffle，从shuffle过的结果中去第一个server地址进行连接。



### Broker

#### 功能介绍

Broker充当着消息中转角色，负责存储消息、转发消息。Broker在RockerMQ系统中负责接收并存储从生产者发送来的消息，同时为消费者的拉取请求做准备。Broker同时也存储着消息相关的元数据，包含消费者组消费进度偏移offset、主题、队列等。

> Kafka 0.9版本之后，offset是存放在Broker中的，之前的版本是存放在Zookeeper中的

#### 模块构成

功能示意图如下：

![image-20220502211219380](https://tva1.sinaimg.cn/large/e6c9d24egy1h1udb4j0gbj20kz0bzweq.jpg)

 

| Remoting Module    | 整个Broker的实体，负责处理来自clients端的请求。而这个Broker实体则由以下模块构成 |
| ------------------ | ------------------------------------------------------------ |
| **Client Manager** | **客户端管理器，负责接收、解析、客户端（Producer/Consumer）请求，管理客户端。例如：维护Consumer的Topic订阅信息** |
| **Store Service**  | 存储服务。提供方便简单的API接口，处理消***息存储到物理硬盘***和***消息查询***功能 |
| **HA Service**     | **高可用服务，提供Master Broker和Slave Broker之间的数据同步功能** |
| **Index Service**  | **索引服务。根据特定的Message key，对投递到Broker的消息进行索引服务，同时也提供根据Message Key对消息进行快速查询的功能** |



#### 工作流程

1. 启动NameServer，NameServer启动后开始监听端口，等待Broker、Producer、Consumer连接。

2. 启动Broker时，Broker会与所有的NameServer建立并保持长连接，然后每30s向NameServer定时发送心跳包

3. 收发消息前，可以先创建Topic，创建Topic时需指定该Topic需要存储到哪些Broker上，当然，在创建Topic时会将Topic与Broker的关系写入到NameServer中。不过这步是可选的，也可以在发送消息时自动创建Topic。

   >

4. Producer发送消息，启动时先跟NameServer集群中的某一台建立长连接，并从NameServer中获取路由信息，即当前发送的Topic消息的Queue与Broker的地址（IP+Port）的映射关系。然后根据算法策略从队列选择一个Queue，与队列所在的Broker建立长连接从而向Broker发消息。当然，在获取路由信息后，Produce会首先将路由信息缓存到本地，在每30s从NameServer更新一次路由信息。

5. Consumer和Producer类似，跟其中一台NameServer建立长连接，获取其所订阅Topic的路由信息，然后根据算法策略从路由信息中获取到其所要消费的Queue，然后直接跟Broker建立长连接，开始消费其中的消息。Consumer在获取路由信息后，同时也会每30s从NameServer更新一次路由信息。不过不同于Producer的是。Consumer还会向Broker发送心跳，以确保Broker的存活状态



#### Topic创建流程

手动创建Topic时，有两种模式：

* 集群模式：改模式下创建的Topic在该集群中，所有的Broker中的Queue数量是相同的。

* Broker模式：该模式下创建的Topic在该集群中，每个Broker中的Queue数量可以不同。

自动创建Topic时，默认采用的是Broker模式，会为每个Broker默认创建四个Queue

 

#### 读/写队列

从物理上来讲，读/写队列是同一个队列。所以，不存在读/写队列数据同步问题。读/写队列是逻辑上进行区分的概念。一般情况下，读/写对垒数量是相同的。

例如，创建Topic时，设置的写队列数量为8，读队列数量为4，此时系统会创建8个Queue，分别是0 1 2 3 4 5 6 7。Producer会将消息写入到这8个队列中，但Consumer只会消费0 1 2 3这4个队列中的消息，4 5 6 7中的消息是不会被消费到的。

再如，创建Topic时设置的写队列数量为4，读队列数量为8，此时系统会创建8个Queue，读队列数量为8，此时系统会创建8个Queue，分别是 0 1 2 3 4 5 6 7。Producer会将消息写入到0 1 2 3 这四个队列中，但是Consumer只会消费 0 1 2 3 4 5 6 7 这8个队列中的消息，但是4 5 6 7 中是没有消息的。此时假设Consumer Group中包含两个Consumer，Comsumer1 消费0 1 2 3，而Consumer2消费4 5 6 7。但是实际情况是，Consumer2是没有任何消息可以消费的。

**也就是说，当读/写队列数量设置为不同时，总是有问题的。那么，为什么要这样设计呢？**

这样设计的目的是为了，方便Topic的Queue的缩容。

```
例如，原来创建的Topic中包含16个Queue，如何能够使其Queue缩容为8个，还不会丢失消息？可以动态修改写队列数量为8，读队列数量不变。此时新的消息只会写入到前8个队列，而消费都消费的却是16个队列中的数据。当发现后8个Queue中的消息消费完毕后，就可以再将读队列数量动态设置为8.整个缩容过程，没有丢失任何消息。
```

perm用于设置当前创建Topic的操作权限：**2表示只写，4表示只读，6表示读写**



## 3、RocketMq的安装和启动

### 单机安装与启动

#### 软件下载

下载地址：

```
https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.9.3/rocketmq-all-4.9.3-bin-release.zip
```

#### 修改初始内存

修改runserver.sh

```shell
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m
-- 修改为
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```



修改runbroker.sh

```shell
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m"
```



#### 启动服务

启动NameServer

```shell
  > nohup sh bin/mqnamesrv &
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
```

启动Broker

```shell
  > nohup sh bin/mqbroker -n localhost:9876 &
  > tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success...
```

```
-- 注册成功
2022-05-03 22:29:56 INFO brokerOutApi_thread_2 - register broker[0]to name server localhost:9876 OK
```



#### 消息测试

```shell
 > export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId= ...

 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

**发送消息**

```
SendResult [sendStatus=SEND_OK, msgId=7F000001A3E530F399910F2AB16903E1, offsetMsgId=C0A8BF8700002A9F000000000002E858, messageQueue=MessageQueue [topic=TopicTest, brokerName=pinyoyougou-docker, queueId=3], queueOffset=247]
SendResult [sendStatus=SEND_OK, msgId=7F000001A3E530F399910F2AB19D03E2, offsetMsgId=C0A8BF8700002A9F000000000002E918, messageQueue=MessageQueue [topic=TopicTest, brokerName=pinyoyougou-docker, queueId=0], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=7F000001A3E530F399910F2AB1B203E3, offsetMsgId=C0A8BF8700002A9F000000000002E9D8, messageQueue=MessageQueue [topic=TopicTest, brokerName=pinyoyougou-docker, queueId=1], queueOffset=249]
SendResult [sendStatus=SEND_OK, msgId=7F000001A3E530F399910F2AB1B503E4, offsetMsgId=C0A8BF8700002A9F000000000002EA98, messageQueue=MessageQueue [topic=TopicTest, brokerName=pinyoyougou-docker, queueId=2], queueOffset=248]
```

**消费消息**

```
ConsumeMessageThread_please_rename_unique_group_name_4_3 Receive New Messages: [MessageExt [brokerName=pinyoyougou-docker, queueId=0, storeSize=192, queueOffset=157, sysFlag=0, bornTimestamp=1651588850813, bornHost=/192.168.191.135:55076, storeTimestamp=1651588850822, storeHost=/192.168.191.135:10911, msgId=C0A8BF8700002A9F000000000001D518, commitLogOffset=120088, bodyCRC=1319182829, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1651589065951, UNIQ_KEY=7F000001A3E530F399910F2A9C7D0272, CLUSTER=DefaultCluster, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 54, 50, 57], transactionId='null'}]] 

```

**关闭服务**

```shell
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```





### 控制台的安装

#### 下载

```
https://github.com/apache/rocketmq-externals/releases/tag/rocketmq-console-1.0.0
```

**修改配置**

```properties
server.contextPath=
server.port=7000
#spring.application.index=true
spring.application.name=rocketmq-console
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
spring.http.encoding.force=true
logging.config=classpath:logback.xml
#if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
rocketmq.config.namesrvAddr=192.168.191.135:9876
#if you use rocketmq version < 3.5.8, rocketmq.config.isVIPChannel should be false.default true
rocketmq.config.isVIPChannel=
#rocketmq-console's data path:dashboard/monitor
rocketmq.config.dataPath=/tmp/rocketmq-console/data
#set it false if you don't want use dashboard.default true
rocketmq.config.enableDashBoardCollect=true
```

#### 添加依赖

```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>

<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>

```



#### 打包

```
mvn clean package -Dmaven.test.skip=true
```



#### 启动

```
java -jar rocketmq-console-ng-1.0.0.jar
```



#### 访问

```
http://localhost:7000/#/
```

![image-20220503233211356](https://tva1.sinaimg.cn/large/e6c9d24egy1h1vmz1yhxnj21hb0q3jtf.jpg)



### 集群搭建理论



### 磁盘阵列RAID



### 集群搭建





### mqadmin命令

>在mq解压目录的bin目录下有一个mqadmin命令，该命令是一个运维指令，用于对MQ的主题，集群，broker等信息进行管理

修改bin/tool.sh

在运行mqadmin命令之前，先要修改mq解压目录下bin/tools.sh 配置的jdk的ext目录位置。本机的ext目录在/usr/java/jdk1.8.0——161/jre/lib/ext

运维文档：https://github.com/apache/rocketmq/blob/master/docs/cn/operation.md





## RockeMQ工作原理

### 一、消息的产生

#### 消息的生产过程

Producer可以将消息写入到某Broker中的某Queue中，其经历如下过程：

* Producer发送消息之前，会先向NameServer发出获取消息Topic的**路由信息的**请求
* NameServer返回该Topic的**路由表**及**Broker列表**
* Producer根据代码中指定的Queue选择策略，从Queue列表中选择一个队列，用于后续存储消息
* Producer对消息做一些特殊处理，例如消息本身超过4M，会对其进行压缩
* Producer向选择出的Queue所在的Broker发出RPC请求，将消息发送到选择出的Queue

>路由表：实际是一个Map，Key为Topic名称，value是一个QueueData实例列表。QueueData并不是一个Queue对应一个QueueData，而是一个Broker中该Topic的所有Queue对应一个QueueData。即，只要涉及到该Topic的Broker，一个Broker对应一个QueueData。QueueData包含brokerName。简单来说，路由表的key为Topic的名称，value则为所有涉及到该Topic到BrokerName列表



#### Queue的选择算法

对于无序消息，其Queue选择算法，也称为投递算法，常见的有两种：

**轮询算法**

默认选择算法。该算法保证了每个Queue中可以均匀的获取到消息。

>该算法存在一个问题：由于某些原因，在某些Broker上的Queue可能投递延迟较严重。从而导致Producer 的缓存队列中出现较大的消息堆积，影响消息的投递性能。

**最小投递延迟算法**

该算法会统计每次消息投递的时间延迟，然后根据统计出的结果将消息投递到时间延迟最小的Queue。如果延迟相同，则采用轮询算法投递。该算法可以有效提升消息的投递性能。

> 该算法也存在一个问题：消息在Queue上的分配不均匀。投递延迟小的Queue其可能会存在大量的消息。而对该Queue的消费者压力会增大，降低消息的消费能咯，可能导致MQ中的消息堆积。



### 二、消息的存储

RocketMQ中的消息存储在本地文件系统中，这些相关文件默认存储在当前用户主目录下的store目录下

![image-20220504144842684](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wdgkb41oj20s60a8wg8.jpg)

| Abort            | 该文件在Broker启动后会自动创建，正常关闭Broker，该文件会自动消失。若没有启动Broker的情况下，发现这个文件是存在的，则说明之前的Broker的关闭时非正常关闭的。 |
| ---------------- | ------------------------------------------------------------ |
| **checkpoint**   | 存放着commitlog、consumequeue、index文件的最后刷盘时间戳     |
| **commitlog**    | 存放着commitlog文件，而消息是写在commitlog文件中的           |
| **config**       | 存放着Broker运行期间的一些配置数据                           |
| **consumequeue** | 存放着consumequeue文件，队列就存放在这个目录下               |
| **index**        | 其中存放着消息索引文件indexFile                              |
| **lock**         | 运行期间使用到全局资源锁                                     |



#### commitlog文件

>说明：在很多资料总commitlog目录中的文件简单就称为commitlog文件。但是在源码中，该文件被命名为mappedFile。

**目录与文件**

commitlog目录中存放着很多mappedFile文件，当前Broker中的所有消息都落盘到这些mappedFile文件中。mappedFile文件大小为1G（小于等于1G），文件名由20位十进制数构成，表示当前文件的第一条消息的起始位移偏移量。

>第一个文件名一定是20位0构成的。因为第一个文件的第一条消息的偏移量commitlog offset为0
>
>当第一个文件放满时，则会自动生成第二个文件继续存放消息。假设第一个文件大小是1073741820（1G=1073741824字节），则第二个文件名就是00000000001073741820
>
>以此类推，第n个文件名应该是前n-1个文件大小之后
>
>一个Broker中所有mappedFile文件的commitlog offset是连续的

需要注意的是，一个Broker中仅包含一个commitlog目录，所有的mappedFile文件都说是存放在该目录下的。即无论当前Broker中存放着多少Topic的消息，这些消息都说被顺序写入到了mappedFile文件中的。也就是说，这些消息在Broker中存放时并没有按照Topic进行分类存放的。

>mappedFile 文件是顺序读写的文件，所有其访问效率很高
>
>无论是SSD磁盘还是SATA磁盘，通常情况下，顺序存取效率都会高于随机存储



**消息单元**

![image-20220504153212370](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wepthrdqj20u70ayace.jpg)

mappedFile文件内容由一个个的消息单元构成。每个消息单元中包含消息总长度MsgLen、消息的物理位置physicalOffset、消息体内容Body、消息体长度BodyLength、消息主题Topic、Topic长度TopicLength、消息生产者BornHost、消息发送时间戳BornTimestamp、消息所在的队列QueueId、消息在Queue中存储的偏移量QueueOffset等近20余项消息相关属性。

> 需要注意到，消息单元中是包含Queue相关属性的。所以我们在后续的学习中，就需要十分留意commitlog与queue间的关系是什么？？
>
> 一个mappedFile文件第m+1个消息单元的commitlog offset偏移量
>
> **L(m+1) = L(m) + MsgLen(m)  (m>=0)**



#### **consumequeue**

![image-20220504201912417](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wn0fsof7j20uy0ic781.jpg)





**索引条目**

![image-20220504201433769](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wmvnlgldj20bb05rdfv.jpg)

每个consumequeue文件可以包含30w个索引条目，每个索引条目包含了三个消息重要属性：消息在mappedFile文件中的偏移量CommitLog Offset、消息长度、消息Tag的hashcode值。这个三个属性占20个字节，所以每个文件的大小是固定的30W*20字节

>一个consumequeue文件中所有消息的Topic一定是相同的。但每条信息的Tag可能是不同的



#### 对文件的读写

![image-20220504202134564](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wn2wokb6j20mb0c6q3n.jpg)



**消息写入**

一条消息进入到Broker后经历了以下几个过程才最终被持久化。

* Broker根据queueId，获取到该消息对应索引条目要在consumequeue目录中的写入偏移量，即Queueoffset
* 将queueId、queueOffset等数据，与消息一起封装为消息单元
* 将消息单元写入到commitlog
* 同时形成消息索引条目
* 将消息索引条目分发到相应的consumequeue



**消息拉取**

* 当Consumer来拉取消息时会经历以下几个步骤：

  * Consumer获取到其要消费消息所在Queue的**消费偏移量offset**，计算出其要消费消息的**消息offset**

    >消费offset即消费进度，consumer对其某个Queue的消费offset，即消费到了该Queue的第几条消息
    >
    >消息offset = 消息offset +  1

  * Consumer向Broker发送拉取请求，其中会包含要拉取消息的Queue、消息Offset及消息Tag

  * Broker计算在该consumequeue中的queueOffset

    >queueOffset = 消息iffset * 20字节

  * 从该queueOffset处开始向后查找第一个指定Tag的索引条目

  * 解析该索引条目的前8个字节，即可以定位到该消息在commitlog中的commitlog offset

  * 从对应commitlog offset中读取消息单元，并发送给Consumer

    

**性能提升**

RocketMq中，无论是消息本身还是消息索引，都是存储在磁盘上的，其不会影响消息的消费吗？当然不会，其实RockerMq的性能在目前的MQ产品中性能是非常高的。因为系统通过一系列相关机制大大提升了性能。

首先，RocketMQ对文件的读写操作是通过**mmap零拷贝**进行的，将其对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率。

其次，consumequeue中的数据顺序存放的，还引入了PageCache的预读取机制，使得对consumequeue文件的读取几乎接近内存读取，即时在有消息堆积情况下也不会影响性能。

>PageCache机制，页缓存机制，是OS对文件的缓存机制，用于加速对文件的读写操作。一般来说，程序对文件进行顺序读写的速度几乎接近于内存读写速度，主要原因是由于OS使用PageCache机制对读写访问操作进行性能优化，将一部分的内存用做PageCache。
>
>* 写操作：OS会将数据写入到PageCache中，随后会以异步方式由pdflush （page dirty flush） 内核线程将Cache中的数据刷盘到物理磁盘
>* 读操作：若用户要读取数据，其首先会从PageCache中读取，若没有命中，则OS在物理磁盘上加载该数据到PageCache的同时，也会顺序对其相邻数据块中的数据进行预读取。

RockerMQ中可能会影响性能的是对commitlog文件的读取。因为对commitlog文件来说，读取消息时会产生大量的随机访问，而随机访问会严重影响性能。不过，如果选择合适的系统IO调度算法，比如设置调度算法为Deadline（采用SSD固态硬盘的话），随机读取性能也会有所提升。

# TODO：P48















### 三、indexFile



### 四、消息的消费



### 五、订阅关系的一致性



### 六、offset管理



### 七、消费幂等性



### 八、消息堆积与消费延迟



### 九、消息的清理



# TODO：P71







## RocketMQ应用

### 一、普通消息

#### 消息发送分类

Producer对于消息的发送方式也有多种选择，不同的方式会产生不同的系统效果。



##### 同步发送消息

同步发送消息是指，Producer发出一条消息后，会在收到MQ返回的ACK之后才发下一条消息。该方式的消息可靠性最高，但消息的发送效率太低。

![image-20220504210654073](https://tva1.sinaimg.cn/large/e6c9d24egy1h1woe1vludj20es0ah74l.jpg)

##### 异步发送消息

异步发送消息是指，Producer发出消息后无需等待MQ返回ACK，直接发送下一条消息。该方式的消息可靠性得到保障，消息发送效率也可以。

![image-20220504210813284](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wofg41gcj20gf0bhdg9.jpg)



##### 单向发送消息

单向发送消息是指，Prodcuer仅负责发送消息，不等待、不处理MQ的ACK，该发送方式时MQ也不返回ACK。该方式的消息发送效率最高，但是消息可靠性差。

![image-20220504210920264](https://tva1.sinaimg.cn/large/e6c9d24egy1h1wogladqnj20gv0c1mxe.jpg)



### 二、顺序消息

#### 什么是顺序消费

消息有序指的是可以**按照消息的发送顺序来消费(FIFO)**。RocketMQ可以严格的保证消息有序，可以分为分区有序或者全局有序。

顺序消费的原理解析，在默认的情况下消息发送会采取**Round Robin轮询方式**把消息发送到不同的queue(分区队列)；而消费消息的时候从多个queue上拉取消息，这种情况发送和消费是不能保证顺序。但是如果控制发送的顺序消息只依次发送到同一个queue中，消费的时候只从这个queue上依次拉取，则就保证了顺序。当发送和消费参与的queue只有一个，则是全局有序；如果多个queue参与，则为分区有序，即相对每个queue，消息都是有序的。



#### 为什么需要顺序消费

下面用订单进行分区有序的示例。一个订单的顺序流程是：创建、付款、推送、完成。订单号相同的消息会被先后发送到同一个队列中，消费时，同一个**OrderId**获取到的肯定是同一个队列。

![image-20220522115606894](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h1mnu6xdj20mq09rwf9.jpg)

  



#### 有序性分类

根据有序范围的不同，RocketMQ可以严格地保证两种消息的有序性：分区有序与全局有序

##### 全局有序

当发送和消费参与的Queue只有一个时所保证的有序是整个Topic中消息的顺序，称为**全局有序。**

>在创建Topic时指定Queue的数量，有三种指定方式：
>
>1) 在代码中创建Producer时，可以指定自动创建的Topic的Queue数量
>2) 在RocketMQ可视化控制台中手动创建Topic时，指定Queue数量
>3) 使用mqAdmin命令手动创建Topic时指定Queue数量

![image-20220522115955379](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h1qif2h2j20jn07fmx9.jpg)

##### 分区有序

如果有多个Queue参与，其仅可以保证在该Queue分区队列上的消息顺序，则称为分区有序。

>如何实现Queue的选择，在定义Producer时我们可以指定消息队列选择器，而这个选择器是我们自己实现了MessageQueueSelector接口定义的。
>
>在定义选择器的选择算法时，一般需要使用选择key。这个选择key可以是消息key也可以是其他数据。但无论谁做选择key，都不能重复，都是唯一的。
>
>一般性的选择算法是，让选择key（获其hash值）与该Topic所包含的Queue的数量取模，其结果即为选择出的Queue的QueueId。
>
>**取模算法存在一个问题：不同选择key与Queue数量取模结果可能会是相同的，即不同的选择key的消息可能出现相同的Queue，即同一个Consumer可能会消费到不同选key的消息。这个问题如何解决？？** 
>
>**一般做法：**从消息中获取到选择key，对其进行判断。若是当前consumer需要消费的消息，则直接消费，否则，什么也不做。这种做法要求选择key要能够随着Consumer获取到。此时使用消息key做为选择key是比较好的做法 。
>
>**以上做法会不会出现新的问题呢？？**
>
>以上做法会不会出现如下新的问题呢？不属于那个Consumer的消息被拉取走了，那么应该消费该消息的Consumer是否还能在消费该消息呢？同一个Queue中的消息不可能被同一个Group中不同的Consumer同时消费。所以，消费一个Queue的不同选择key的消息的Consumer一定属于不同的Group。而不同的Group中的Consumer间的消费是相互隔离的，互不影响的。



![image-20220522120439961](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h1vhy7vgj20lv0a5aak.jpg)



### 三、延时消息

#### 什么是延时消息

> 当消息写入大Broker后，在指定的时长后才可被消费处理的消息，称为延时消息。

采用RocketMq的延时消息可以实现定时任务的功能，而无需使用定时器。典型的应用场景是：电商交易中潮湿未支付关闭订单的场景，12306平台订票系统超时未支付取消订票的场景

>在电商平台中，订单创建时会发送一条延迟消息。这条消息将会在30分钟后投递给后台业务系统（Consumer），后台业务系统收到该消息后会判断对应的订单是否已经完成支付。如果未完成，则取消订单，将商品再次放回到库存；如果完成支付，则忽略。
>
>在12306平台中，车票预订成功后就会发送一条延迟消息。这条消息将会在45分钟后投递给后台业务系统Consumer中，后台业务系统收到该消息后会判断对应的订单是否已经完成支付。如果未完成，则取消预订，将车票再次放回票池；如果未完成支付，则忽略



#### 延迟等级

延时消息的延迟时长**不支持随意时长**的延迟，是通过特定的延迟等级来指定的。延迟等级定义在RocketMq服务端的**MessageStoreConfig**类中的如下变量中：

```java
// org/apache/rocketmq/store/config/MessageStoreConfig.java

private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
```

即，若指定的延时等级未3，则表示延时时长未10s，即延迟等级是从1开始计数的。

当然如果需要自定义的延时等级，可以通过在broker加载的配置中新增如下配置（例如下面增加了一天这个等级）。配置文件在RocketMQ安装目录下的conf目录中。

![image-20220522151823081](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h7h0bu3ij20lv01gjre.jpg)



#### 延时消息实现原理

 ![image-20220522152046407](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h7jhxbdnj20mn0d6mxz.jpg)





##### 修改消息

![image-20220522154411639](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h87w5a24j20cq0fagmc.jpg)

Producer将消息发送到Broker后，Broker会首先将消息写入到commitlog文件中，然后需要将其分发到相应的consumerQueue中。不过，在分发之前，系统会判断消息是否带有延时等级，若没有，则直接正常分发；若有，则需要经历一个复杂的过程：

* 修改消息的Topic未Schedule_Topic_XXXX

* 根据延时等级，在consumerqueue目录中Schedule_Topic_XXX主题下创建出相应的queueId目录与consumerQueue文件（如果没有这些目录与文件的话）

  >延迟等级delayLevel与QueueId的对应关系为queueId = delayLevel -1
  >
  >
  >
  >需要注意，在创建queueId目录时，并不是一次性将所有延迟等级对应的目录全部创建完毕，而是用到哪个延迟等级就创建哪个目录

![image-20220522160210749](https://tva1.sinaimg.cn/large/e6c9d24egy1h2h8qkjl27j20jc03faa8.jpg)

* 修改消息索引单元内容。索引单元中的**Message Tag HashCode**部分原本存放的是消息的Tag的Hash值。现修改为消息的**投递时间**。投递时间是指该消息被重新修改为原Topic后再次被写入到commitLog中的时间。***投递时间=消息存储时间+延迟等级时间***。消息存储时间指的是消息被发送到Broker时的时间戳。

* 将消息索引写入到Schedule_Topic_XXXX主题下相应的consumequeue中

  >***Schedule_Topic_XXXX目录中各个延时等级Queue中的消息是如何排序的？***
  >
  >
  >
  >**是按照消息投递时间排序的。**一个Broker中的桶以等级的所有延时消息会被写入到consumequeue目录中 Schedule_Topic_XXXX目录下相同的Queue中。即一个Queue中消息投递时间的延迟等级时间是相同的。那么投递时间就取决于消息存储时间了。**即按照消息被发送到Broker的时间进行排序的。**



##### 投递延时消息

Broker内部有一个延迟消息服务类ScheduleMessageService，其会消费Schedule_Topic_XXXX中的消息，即按照每条消息的投递时间，将延迟消息投递到目标Topic中，不过，在投递之前会从commitLog中将原来写入的消息再次读出，并将其原本的延迟等级设置为0，即原消息变为了一条不延迟的普通消息。然后再次将消息投递到目标Topic中。

>**ScheduleMessageService** 在Broker启动时，会创建并启动一个定时器**Timer**，用于执行相应的定时任务。系统会根据延时等级的个数，定义相应数量的TimerTask，**每个TimerTask负责一个延迟等级消息**的消费与投递。**每个TimerTask都会检测相应Queue队列的第一条消息是否到期**。若第一条消息未到期，则后面的所有消息更不会到期（消息是按照投递时间排序的）；若第一条消息到期了，则将该消息投递到目标Topic，即消费该消息。



##### 将消息重新写入commitlog

延时消息服务类ScheduleMessageService 将延迟消息再次发送给commitLog，并再次形成新的消息索引条目，分发到相应的Queue

>这其实就是一次普通消息发送。只不过这次的消息Producer是延迟消息服务类**ScheduleMessageService**





### 四、事务消息

#### 问题引入

> 需求场景：工行用户A向建行用户B转账1万元

![image-20220522180212238](https://tva1.sinaimg.cn/large/e6c9d24egy1h2hc7ige22j20k507x3yo.jpg)

 ![image-20220522180339518](/Users/gaoshang/Library/Application Support/typora-user-images/image-20220522180339518.png)

1) 工行系统发送一个给B增款1万元的同步消息M给Broker

2) 消息被Broker成功接收后，向工行系统发送成功ACK

3) 工行系统收到成功ACK后用户A中扣款1万元

4) 建行系统从Broker中获取到消息M

5) 建行系统消费消息M，即向用户B中增加一万元

   >其中是有问题的：
   >
   >若第三步的扣款操作失败，但消息已经成功发送到了Broker。对于MQ来说，只要消息写入成功，那么这个消息就可以被消费。此时建行系统中用户B增加了一万元。出现了数据不一致问题。



#### 解决思路

解决思路：让第1、2、3步具有原子性，要么全部成功，要么全部失败。即消息发送成功后，必须要保证扣款成功。如果扣款失败，则回滚发送成功的消息。而该思路即使用事务消息。这里要使用**分布式事务**解决方案。



![image-20220522210944625](https://tva1.sinaimg.cn/large/e6c9d24egy1h2hhmr2dumj20ft0em753.jpg)

**使用事务消息来处理该需求场景：**

1. 事务管理器TM向事务协调器TC发起指令，开启**全局事务**

2. 工行系统发一个给B增款一万元的事务M 给TC

3. TC会向Broker发送**半事务消息prepareHalf**，将消息M预提交到Broker。此时的建行系统是看不到Broker中的消息M的

4. Broker会将预提交执行结果给**TC**

5. 如果预提交失败，则TC向TM上报预提交失败的相应，全局事务结束；如果预提交成功，TC会调用工行系统的**回调操作**，去完成工行用户A的**预扣款**1万元的操作

6. 工行系统会向TC发送预扣款执行结果，即**本地事务**的执行状态

7. TC收到预扣款执行结果后，会将结果上报给TM

   > 预扣款执行结果存在三种可能性
   >
   > ```java
   > package org.apache.rocketmq.client.producer;
   > public enum LocalTransactionState {
   >     COMMIT_MESSAGE,
   >     ROLLBACK_MESSAGE,
   >     UNKNOW;
   >     private LocalTransactionState() {
   >     }
   > }
   > ```

8. TM会根据上报结果向TC发出不同的指令

   * 若预扣款成功（本地事务状态为**COMMIT_MESSAGE**），则TM向TC发送**Global Commit**指令
   * 若预扣款失败（本地事务状态为**ROLLBACK_MESSAGE**），则TM向TC发送**Global Rollback**指令
   * 若出现未知状态（本地事务为UNKONW），则会触发工行系统的本地事务状态**回查操作**。回查操作会将回查结果 即COMMIT_MESSAGE或ROLLBACK_MESSAGE给TC。TC将结果上报给TM，TM会再向TC发送最终确认指令Global Commit或者Global Rollback 

9. TC在接收到指令后会向Broker与工行系统发出确认指令

   * TC接收的若是Global Commit指令，则向Broker与工行系统发送Branch Commit 指令，此时Broker中的消息M才可被建行系统所看到的；此时的工行用户A的扣款操作才真正被确认
   * TC接收到若是Global Rollback指令，则向Broker与工行系统发送Branch Rollback指令。此时Broker中的消息M将被撤销；工行用户A中的扣款操作将被回滚

   >**以上方案就是为了确保消息投递与扣款操作能够在一个事务中，要成功都成功，有一个失败则全部回滚。**
   >
   >
   >
   >以上方案并不是一个典型XA模式。因为XA模式中的分支事务是异步的，而上述例子事务消息方案中的消息预提交预与预扣款操作间是同步的





#### 事务基础

##### 分布式事务

```
对于分布式事务，通俗地说就是，一次操作由若干分支操作组成，这些分支操作分属不同应用，分布在不同的服务器上。分布式事务需要保证这些分支操作要么全部成功，要么全部失败。分布式事务与普通事务一样，就是为了保证操作结果的一致性。
```



##### 事务消息

```
RocketMQ提供了类似X、Open XA的分布式事务功能，通过事务消息达到分布式事务的最终一致。XA是一种分布式事务解决方案，一种分布式事务处理模式。
```



##### 半事务消息

```
暂不能投递的消息，发送方已经成功地将消息发送到了Broker，但是Broker未收到最终的确认指令，此时该消息被标记为“暂不能投递”状态，即不能被消费者看到。处于该种状态下的消息即半事务消息。
```



##### 本地事务状态

```
Producer回调操作执行的结果为本地事务状态，其会发送给TC，而TC会再发送给TM。TM会根据TC发送来的本地事务状态来决定全局事务确认指令。
```

```java
package org.apache.rocketmq.client.producer;
public enum LocalTransactionState {
    COMMIT_MESSAGE,
    ROLLBACK_MESSAGE,
    UNKNOW;
    private LocalTransactionState() {
    }
}
```



##### 消息回查

![image-20220522215749077](https://tva1.sinaimg.cn/large/e6c9d24egy1h2hj0o9c96j20pf0k7jse.jpg)



消息回查，即重新查询本地事务的执行状态。本例就是重新到DB中查看预扣款操作是否执行成功。

 >注意，消息回查不是重新执行回调操作。回调操作是进行预扣款处理的，而消息回查则是查看预扣款操作执行的结果
 >
 >
 >
 >引起消息回查的原因最常见的有两个：
 >
 >1. 回调操作返回UNKONW
 >2. TC没有接收到TM返回的最终全局事务确认指令

##### RocketMQ中的消息回查设置

关于消息回查，有三个常见的属性设置。它们都在Broker加载的配置文件中设置，例如：

* transactionTimeout=20 指定TM在20秒内应将最终确认状态返回给TC，否则引发消息回查。默认为60s
* transactionCheckMax =5，指定最多回查5次，超过后将丢弃消息并记录错误日志。默认15词。
* transactionCheckInterval =10，指定设置的多次消息回查的时间间隔为10s，默认为60s。



#### XA模式三剑客

##### XA协议

```
XA（Unix Transaction） 是一种分布式事务解决方案，一种分布式事务处理模式，是基于XA协议。XA协议由Tuxedo（Transaction for Unix has been Extended for Distribute  Operation  分布式操作扩展之后的Unix事务系统）首先提出的，并由X/Open组织，作为资源管理器与事务管理器的接口标准。
```

```
XA模式中有三个重要组件：**TC、TM、RM**
```



##### TC

```
Transaction Coordinator，事务协调者。维护全局和分支事务的状态，驱动全局事务提交或者回滚。

Broker充当这个角色
```



##### TM

```
Transaction Manager  事务管理器，定义全局事务的范围：开始全局事务、提交或回滚全局事务。它实际是全局事务的发起者

Producer充当着TM
```



##### RM

```
Resource Manager ,资源管理器。管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚

Producer及Broker 均是RM
```



#### XA模式架构

![image-20220523230225634](https://tva1.sinaimg.cn/large/e6c9d24egy1h2iqi7csutj20ix0dpjsi.jpg)

 XA是一个典型2PC，其执行原理如下：

1. TM向TC发起指令，开启一个全局事务

2. 根据业务要求，各个RM会逐个向TC注册分支事务，然后TC会逐个向RM发出预执行指令

3. 各个RM在接收到指令后会在进行本地事务预执行

4. RM将执行结果Report给TC。当然，这个结果可能是成功，也可能是失败。

5. TC在接收到各个RM的Report后会将汇总结果上报给TM，根据汇总结果TM向TC发出确认指令

   * 若所有结果都是成功响应，则向TC发送GlobalCommit 指令
   * 只要结果是失败响应，则向TC发送Global Rollback指令

6. TC在接收指令后再次向RM发送确认指令

   > 事务消息方案并不是一个典型的XA模式。因为XA模式中的分支事务是异步的，而事务方案中的消息预提交与扣款操作间是同步的。



#### 注意

* 事务消息不支持延时消息
* 对于事务消息要做好幂等性减产，因为事务消息可能不止一次被消费（因为存在回滚后在提交的情况）
* **事务消息不支持延时消息和批量消息。**
* 为了避免单个消息被检查太多次而导致半队列消息累积，我们默认将单个消息的检查次数限制为 15 次，但是用户可以通过 Broker 配置文件的 `transactionCheckMax`参数来修改此限制。如果已经检查某条消息超过 N 次的话（ N = `transactionCheckMax` ） 则 Broker 将丢弃此消息，并在默认情况下同时打印错误日志。用户可以通过重写 `AbstractTransactionalMessageCheckListener` 类来修改这个行为。
* 事务消息将在 Broker 配置文件中的参数 transactionTimeout 这样的特定时间长度之后被检查。当发送事务消息时，用户还可以通过设置用户属性 CHECK_IMMUNITY_TIME_IN_SECONDS 来改变这个限制，该参数优先于 `transactionTimeout` 参数。
* **事务性消息可能不止一次被检查或消费。**
* 提交给用户的目标主题消息可能会失败，目前这依日志的记录而定。它的高可用性通过 RocketMQ 本身的高可用性机制来保证，如果希望确保事务消息不丢失、并且事务完整性得到保证，建议使用同步的双重写入机制。
* 事务消息的生产者 ID 不能与其他类型消息的生产者 ID 共享。与其他类型的消息不同，事务消息允许反向查询、MQ服务器能通过它们的生产者 ID 查询到消费者



### 五、批量发送消息

#### 批量发送消息

##### 发送限制

生产者进行消息发送时可以一次发送多条消息，这可以大大提升Producer的发送效率。不过需要注意以下几点：

* 批量发送的消息必须具有相同的Topic
* 批量发送的消息必须具有相同的刷盘策略
* 批量发送的消息不能是延迟消息与事务消息



##### 批量发送大小

默认情况下，一批发送的消息总大小不能超过4MB字节。如果想超过该值，有两种解决方案：

* 方案一：将批量消息进行拆分，拆分为若干不大于4M的消息集合分多次批量发送

* 方案二：在Producer端与Broker端修改属性
  * Producer端需要在发送之前设置的Producer的MaxMessageSize属性
  * Broker端需要修改其加载的配置文件中的maxMessageSize属性



##### 生产者发送的消息大小

![image-20220529124756034](https://tva1.sinaimg.cn/large/e6c9d24egy1h2p6gmxtpxj20ew05yq2x.jpg)

生产者听过send()方法发送的Message，并不是直接将Message序列化后发送到网络上的，而是通过这个Message生成了一个字符串发送出去的。这个字符串由四部分构成：topic、消息Body、消息日志（占20字节），及用于描述消息的一堆属性key-value。这些属性中包含例如生产者地址、生产时间、要发送的QueueId等。最终写入到Broker中消息单元中的消息数据都是来自于这些属性的。

#### 批量消费消息



使用MessageListenerConcurrently监听接口的consumeMessage() 方法的第一个参数为消息列表，但默认情况下，每次只能消费一条消息。若要使其一次可以消费多条消息，则可以通过COnsumer的consumeMessageBatchMaxSize属性来指定。不过，该值不能超过32。因为默认情况下消费者每次可以拉取的消息最多是32条。若要修改一次拉取的最大值，则可以通过修改Consumer的pullBatchSize属性来指定。



##### 存在的问题

consumer的pullBatchSize属性与consumeMessageBatch属性是否设置越大越好？当然不是

* pullBatchSize值设置的越大，Consumer每拉取一次需要的时间就会越长，且在网络上传输出现问题的可能性就越高。**若在拉取过程中若出现了问题，那么本批次所有的消息都需要全部重新拉取**
* consumerMessageBatchMaxSize值设置的越大，Consumer的消息并发消费能力越低，且这批被消费的消息具有相同的效果。因为consumeMessageBatchMaxSize指定的一批消息只会使用一个线程进行处理，**且在处理过程中只要有一个消息处理异常，则这批消息需要全部重新再次消费处理。**



#### 代码举例

该批量发送的需求时，不修改最大发送4M的默认值，但要防止发送的批量消息超过4M的限制。



### 六、消息过滤

消费者在进行消息订阅时，除了可以指定要订阅消息的Topic外，还可以对指定Topic中的消息根据指定条件进行过滤，即可以订阅比Topic更加细粒度的消息类型。

对于指定Topic消息的过滤有两种过滤方式：Tag过滤与SQL过滤

#### Tag过滤

在大多数情况下，TAG是一个简单而有用的设计，其可以来选择您想要的消息。例如：

```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_EXAMPLE");
consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");
```



#### SQL过滤

SQL过滤是一种复杂特定表达式对事先埋入到消息中的用户属性进行筛选过滤的方式。通过SQL过滤，可以实现对消息的复杂过滤。不过，只有使用PUSH模式的消费者才能使用SQL过滤。

SQL过滤表达式支持多种常量类型与运算符。

RocketMQ只定义了一些基本语法来支持这个特性。你也可以很容易地扩展它。

- 数值比较，比如：**>，>=，<，<=，BETWEEN，=；**
- 字符比较，比如：**=，<>，IN；**
- **IS NULL** 或者 **IS NOT NULL；**
- 逻辑符号 **AND，OR，NOT；**

常量支持类型为：

- 数值，比如：**123，3.1415；**
- 字符，比如：**'abc'，必须用单引号包裹起来；**
- **NULL**，特殊的常量
- 布尔值，**TRUE** 或 **FALSE**

默认情况下，Broker没有开启消息的SQL过滤功能，需要在Broker加载的配置文件中添加如下属性：

```properties
# 默认是关闭的
enablePropertyFilter = true
```

在启动Broker时需要指定这个修改过的配置文件。例如对于单机Broker的启动，其修改的配置文件是conf/broker.conf  启动时使用如下命令：

```shell
sh bin/mqbroker -n localhost:9876 -c conf/broker.conf &
```



### 七、消息发送重试机制

#### 说明

对于消息重投，需要注意以下几点：

* 生产者在发送消息时吗，若采用同步或异步发送方式，发送失败会重试，但oneway消息发送方式发送失败是没有重试机制的
* 只有普通消息具有发送重试机制，顺序消息是没有的**（必须在一个Q上）**
* 消息重投机制可以保证消息尽可能发送成功、不丢失，但可能造成消息消费重复。消息重复在RocketMQ中是无法避免的
* 消息重复一般情况下不会发生，当出现消息量大、网络抖动、消息重复就会成为大概率事件
* producer主动重发、consumer负载变化（发生Rebalance，不会导致消息重复，但可能出现重复消费）也会导致重复消息
* 消息重复无法避免，但要避免消息的重复消费
* 避免消息重复消费的解决方案是，为消息天价唯一标识（例如消息key），使消费者对消息进行消费判断来避免重复消费
* 消息发送重试有三种策略可以选择：同步发送失败策略、异步发送失败策略、消息刷盘失败策略



#### 同步发送失败策略

对于普通消息，消息发送默认采用round-robin策略来选择所发送到的队列。如果发送失败，默认重试2次。但在重试时是不会选择上次发送失败的Broker，而是选择其他Broker。当然，若只有一个Broker其也只能发送到该Broker，但其会尽量发送到该Broker上的其他Queue。

```java
 // 设置消息发送失败重试次数
 producer.setRetryTimesWhenSendFailed(5);
 producer.setSendMsgTimeout(5);
```

同时，Broker还具有**失败隔离**功能，使Producer尽量选择未发生过发送失败的Broker作为目标Broker。其可以保证其他消息尽量不发送到问题Broker，为了提升消息发送效率，降低消息发送耗时。

> ***思考：让我们自己实现失败隔离功能，如何来做？***
>
> **方案一**：Producer中维护某JUC的Map集合，其key是发生失败的时间戳，value为broker实例。
>
> Producer中还维护着一个Set集合，其中存放着所有未发生发送异常的Broker实例。选择目标Broker是从该Set集合中选择的。在定义一个定时任务，定期从Map集合中将长期未发生发送异常的Broker清理出去，并添加到Set集合中。
>
> **方案二**：为Producer中的Broker实例添加一个标识，例如是一个AtomicBoolean属性。只要该Broker上发生过发送异常，就将其置为true。选择目标Broker就是选择该属性值为false的Broker。在定义一个定时任务，定期将Broker的该属性置为false。
>
> **方案三**：为Producer中的Broker实例添加一个标识，例如是一个AtomicLong属性。只要该Broker上发生过发送异常，就使其增加1。选择目标Broker就是选择该属性值最小的Broker。若该值相同，采用轮询方式选择。



如果超过重试次数，则抛出异常，由producer去保证消息不丢。当然当生产者出现RemotingException、MQclientException、MQBrokerException时，Producer会自动重投消息。



#### 异步发送失败策略

异步发送失败重试时，异步重试不会选择其他broker，仅在一个broker上做重试，所以该策略无法保证消息不丢。

```java
  // 设置消息发送失败重试次数
  producer.setRetryTimesWhenSendAsyncFailed(0);
```



#### 消息刷盘失败策略

消息刷盘超时（Master或Slave）或slave不可用（slave在做数据同步时向master返回状态不是SEND_OK）时，默认是不会将消息尝试发送到其他Broker的。不过，对于重要消息可以通过在Broker的配置文件设置**retryAnotherBrokerWhenNotStoreOK**属性为true来开启。





### 八、消息发送消费重试机制

#### 顺序消息的消费重试

对于顺序消息，当Consumer消费消息失败后，为了保证消息的顺序性，其会自动不断地进行消息重试，直到消费成功。

```java
// 顺序消费消息失败的消息重试时间间隔，单位毫秒，默认为1000，其取值范围为[10,30000]
 consumer.setSuspendCurrentQueueTimeMillis(100);
```

> **由于对顺序消息的重试是无休止的，不间断的，直至消费成功，所以对于顺序消费，务必要保证应用能够及时监控并处理消费失败的情况，避免消费被永久性阻塞。**

> 注意，顺序消费没有发送失败重试机制，但具有消费失败重试机制



#### 无序消息的消费重试

对于无序消息（普通、延时、事务消息），当Consumer消费消息失败时，可以通过设置返回状态达到消息重试的效果，不过需要注意，无序消息的重试 **只对集群消费方式生效**，广播消费方式不提供失败重试特性。即对于广播消费，消费失败后，失败消息不在重试，继续消费后续消息。



#### 消费重试次数与间隔

对于无序消息集群消费下的重试消费，每条消息默认最多重试16次，但每次重试的间隔时间是不同的，会逐渐变长，每次重试的间隔时间如下：

| 重试次数 | 与上次重试的间隔时间 |
| :------: | :------------------: |
|    1     |         10s          |
|    2     |         30s          |
|    3     |        1分钟         |
|    4     |        2分钟         |
|    5     |        3分钟         |
|    6     |        4分钟         |
|    7     |        5分钟         |
|    8     |        6分钟         |
|    9     |        7分钟         |
|    10    |        8分钟         |
|    11    |        9分钟         |
|    12    |        10分钟        |
|    13    |        20分钟        |
|    14    |        30分钟        |
|    15    |        1小时         |
|    16    |        2小时         |

> 若一条消息在一直消费失败的前提下，将会在正常消费后的**第4小时46分后进行第16次尝试**，若还失败，**则将消息投递到死信队列中。**

> ```java
> consumer.setMaxReconsumeTimes(10);
> ```
>
> 对于修改过的重试次数，将按照以下策略执行：
>
> * 若修改值小于16，则按照指定时间间隔进行重试
> * 若修改值大于16，则超过16次的重试时间间隔均为2小时
>
> 对于Consumer Group，若仅修改了一个Consumer的消费重试次数，则会应用到该Group中所有其他Consumer实例。若出现多个Consumer均做了修改的情况，则采用覆盖方式生效，即最后被修改的值会覆盖前面设置的值。



#### 重试队列

对于需要重试消费的消息，并不是Consumer在等待了指定时长后再次去拉取原来的消息进行消费的，而是将这些需要重试的消息放到一个Topic的队列中，而后进行再次消费的。这个特殊的队列就就是**重试队列**。

当出现需要重试消费的消息时，Broker会为每个消费组都设置一个Topic名词为 **%RETRY%consumerGroup@consumerGroup**的重试队列

> 1）这个重试队列是针对消息才组的，而不是针对每个Topic设置的（一个Topic的消息可以让多个消费组进行消费，所以会为这些消费者组各创建一个重试队列）
>
> 2）只有当出现需要进行重试消费的消息后，才会为该消费者组创建重试队列

![image-20220529210614795](https://tva1.sinaimg.cn/large/e6c9d24egy1h2pkv59h8yj20mh0bo0ty.jpg)

> 注意，消费重试的时间间隔与**延时消费**的**延时等级**十分相似，除了没有延迟等级的前两个时间外，其他的时间都是相同的

Broker对于重试消息的处理是通过延时消息实现的。先将消息保存到SCHEDULE_TOPIC_XXXX延迟队列中，延迟时间到后，会将消息投递到%RETRY%consumerGroup@consumerGroup重试队列中。



#### 消费重试配置方式

 集群消费方式下，消息消费失败后若希望消费重试，则需要在消息监听器接口中明确进行如下下三种方式之一的配置：

* 返回ConsumeConcurrentStatus.RECONSUME_LATER
* 返回Null
* 抛出异常



#### 消费不重试配置方式

 集群消费方式下，消息消费失败后若不希望消费重试，则在捕获异常后同样也返回与消费成功后相同的结果，即ConsumeConcurrentStatus.CONSUMER_SUCCESS ,则不进行消费重试。

 

### 九、死信队列

#### 什么是死信队列

当一条消息初次消费失败，消息队列会自动进行消费重试；达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确的消费该消息，此时，消息队列不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。这个队列就是死信队列（Dead-Letter Queue，DLQ）,而其中的消息则称为死信消息（Dead-Letter Message，DLQ）

> 死信队列是用于处理无法被正常消费的消息的。



#### 死信队列的特征

死信队列具有如下特征：

* 死讯队列中的消息不会再被消费者正常消费，即DLQ对于消费者是不可见的
* 死信存储有效期与正常消息相同，均为3天（commiylog文件的过期时间），3天后会被自动删除
* 死信队列就是一个特殊的Topic，名称为%DLQ%consumerGroup@consumerGroup，即每个消费组都有一个死信队列
* 如果一个消费者组未产生死信消息，则不会为其创建相应的死信队列



#### 死信消息的处理

实际上，当一条消息进入死信队列，就意味着系统中某些地方出现了问题，从而导致了消费者无法正常消费该消息，比如代码中原本就存在bug。因此，对于死信消息，通常需要开发人员进行特殊处理。最关键的步骤是要排查可疑因素，解决代码中可能存在的bug，然后再将原来的死信消息再次进行投递消费。





















