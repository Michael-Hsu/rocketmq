# 基本概念
----
## 1 消息模型（Message Model）

RocketMQ主要由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。
Broker 在实际部署过程中对应一台服务器，每个 Broker 可以存储多个Topic的消息，每个Topic的消息也可以分片存储于不同的 Broker（Broker和Topic是多对多关系）。
Message Queue 用于存储消息的物理地址（相当于是一个索引，消息顺序写入CommitLog中），每个Topic中的消息地址存储于多个 Message Queue 中。
**ConsumerGroup 由多个Consumer 实例构成.**


## 2 消息生产者（Producer）
 负责生产消息，一般由业务系统负责生产消息。一个消息生产者会把业务应用系统里产生的消息发送到broker服务器。RocketMQ提供多种发送方式，同步发送、异步发送、顺序发送、单向发送。
 同步和异步方式均需要Broker返回确认信息，单向发送不需要。单向发送由于不能让Producer感知到消息是否发送成功，不可靠。为保证可靠性，多使用同步发送和异步发送方式，当ack失败时
，producer可以重新发送。
 
## 3 消息消费者（Consumer）
 负责消费消息，一般是后台系统负责异步消费。
 一个消息消费者会从Broker服务器拉取消息、并将其提供给应用程序。从用户应用的角度而言提供了两种消费形式：拉取式消费、推动式消费。
 **推荐使用拉取式消费，推动式消费会导致Consumer消费压力过大。**
 
## 4 主题（Topic）
  表示一类消息的集合，每个主题包含若干条消息，每条消息只能属于一个主题，是RocketMQ进行消息订阅的基本单位。
  Topic与Msg是一对多的关系。
  
## 5 代理服务器（Broker Server）
消息中转角色，负责存储消息、转发消息。代理服务器在RocketMQ系统中负责接收从生产者发送来的消息并存储、同时为消费者的拉取请求作准备。代理服务器也存储消息相关的元数据，包括消费者组、消费进度偏移和主题和队列消息等。

## 6 名字服务（Name Server）
 名称服务充当路由消息的提供者。生产者或消费者能够通过名字服务查找各主题相应的Broker IP列表。
 **多个Namesrv实例组成集群，但相互独立，没有信息交换。（没有Master-Slave之分，不同于zk，没有复杂的选主）**
 
## 7 拉取式消费（Pull Consumer）
  Consumer消费的一种类型，应用通常主动调用Consumer的拉消息方法从Broker服务器拉消息、主动权由应用控制。一旦获取了批量消息，应用就会启动消费过程。
  MQ本身就是做异步的，这种方式是可以允许有时间差的，但要考虑拉取的周期和策略是什么？
  
## 8 推动式消费（Push Consumer）
 Consumer消费的一种类型，该模式下Broker收到数据后会主动推送给消费端，**该消费模式一般实时性较高**。
 
## 9 生产者组（Producer Group）
  同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。
  
## 10 消费者组（Consumer Group）
  同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。RocketMQ 支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。
  **RocketMQ中默认采用集群消费模式**
  
## 11 集群消费（Clustering）
集群消费模式下,相同Consumer Group的每个Consumer实例**平均分摊消息**。一条消息被ConsumerGroup中的某个消息消费完后，不会在被其他Consumer消费。

## 12 广播消费（Broadcasting）
广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。一条消息被多个Consumer消费，即使这些Consumer属于同一ConsumerGroup，消息也会被ConsumerGroup中的
每个消费者消费一次。广播消费中，ConsumerGroup的概念在消息划分方面无意义。
适用场景：每条消息需要被集群下的每个消费者处理的场景。
举例：Consumer所在的机器使用了本地缓存，收到消息后需要更新缓存，这时就需要使用广播模式，不能使用集群模式。
ps:这是一个不太好的应用场景，因为RocketMQ广播消费模式下一旦Consumer消费失败并不会进行消息重投。一般像这种就应使用redis这样的分布式缓存，而不是使用本地缓存。

## 13 普通顺序消息（Normal Ordered Message）
普通顺序消费模式下，消费者通过同一个消费队列收到的消息是有顺序的，不同消息队列收到的消息则可能是无顺序的。

## 14 严格顺序消息（Strictly Ordered Message）
严格顺序消息模式下，消费者收到的所有消息均是有顺序的。

## 15 消息（Message）
消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。RocketMQ中每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能。
## 16 标签（Tag）
 为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。
 
 中台业务线A承载了B、C、D各个业务线，使用了统一的topic便于消息处理，但为了标识各个业务线便于拆分操作，会使用tagB、tagC和
 tagD进行二次标识，而tagB、tagC和tagD是由A业务线提前统一定义好的，可以使用对应的业务标识。

