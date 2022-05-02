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



## RocketMq的安装和启动

### 单机安装与启动







