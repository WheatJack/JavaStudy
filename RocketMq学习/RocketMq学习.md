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























### 三、indexFile



### 四、消息的消费



### 五、订阅关系的一致性



### 六、offset管理



### 七、消费幂等性



### 八、消息堆积与消费延迟



### 九、消息的清理





















## RocketMQ应用



























