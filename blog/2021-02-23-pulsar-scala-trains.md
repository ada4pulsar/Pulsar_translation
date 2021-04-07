---
title: 开往瑞士的 Pulsar 火车网
categories: "engineering"
image: "/blog/media/scala-trains.png"
topBackgroundImage: "/blog/media/scala-trains.png"
summary: "请用一句话提炼文章内容概要，补充到这里。"
displayDate: "2021-02-23"
tags: "Pulsar，事件驱动, 铁路网"
authorList: ["pavels-sisojevs"]
translator: ["teng-da"]
reviewer: ["sijia","yunze"]
id: "2021-02-23-pulsar-scala-trains"
---


请把 google doc 中文版本内容以 markdown 格式放到这里。
# Event-driven railway network based on Pulsar


# 基于 Pulsar 的事件驱动铁路网

I took this photo while crossing [Landwasser viaduct](https://en.wikipedia.org/wiki/Landwasser_Viaduct) in Filisur, Switzerland. Switzerland is famous for [its railway network](https://en.wikipedia.org/wiki/Rail_transport_in_Switzerland). According to Wikipedia, it is the densest railway network in the world. How about creating a virtual simulator of it? Sounds great!

这张照片拍摄于瑞士的 [Landwasser 高架桥](https://en.wikipedia.org/wiki/Landwasser_Viaduct)。瑞士以其[铁路网络](https://en.wikipedia.org/wiki/Rail_transport_in_Switzerland)闻名于世，根据维基百科，瑞士拥有世界上最密集的铁路网。本文带你一起模拟瑞士的铁路网络。



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


In this article, I would like to introduce you to [Apache Pulsar](https://pulsar.apache.org/) and [Neutron](https://github.com/cr-org/neutron). Apache Pulsar is an open-source distributed pub-sub messaging system originally created at Yahoo and now is part of the Apache Software Foundation. Often it is compared to Apache Kafka. If you are interested in a comparison of these two systems you can find several articles on this topic.

我们会用到 [Apache Pulsar](https://pulsar.apache.org/) 和 [Neutron](https://github.com/cr-org/neutron)。Apache Pulsar 是开源分布式 pub-sub 消息系统，最初由 Yahoo! 开发，目前属于 Apache 软件基金会。数据架构师、数据分析师、程序员等经常对比 Apache Pulsar 和 Apache Kafka，目前已有许多对比二者优劣势的文章。

Also, we will use Neutron. It is a Pulsar client which is based on [FS2](https://fs2.io/) streaming library. Be aware, even though Neutron is developed and used in production by [Chatroulette](https://about.chatroulette.com/), it is still in active development.

Neutron 是基于 [FS2](https://fs2.io/) 流媒体库的 Pulsar 客户端。作为一款成熟的产品， Neutron 已经用于 [Chatroulette](https://about.chatroulette.com/) 的生产，但 Neutron 的开发并未停止。

In my childhood, I always wanted to have a huge toy railway but never had one. Now I can build a virtual simulator on my own.

In this tutorial, we will build an event-driven railway network simulator together.

拥有一套玩具铁路网一直是我童年时的梦想。现在，我可以自己动手搭建一套虚拟铁路网了。

接下来，我们将一起开发一个事件驱动的铁路网络模拟器。

The idea 


# 思路

We will build a railway network MVP consisting of three stations: Geneva, Bern, and Zurich. Geneva and Zurich are connected to Bern and are not connected to each other.

我们要搭建一套包含三个车站的铁路网：日内瓦、伯尔尼和苏黎世。其中日内瓦和苏黎世均与伯尔尼相连，但日内瓦与苏黎世不相连。



<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


Every station will be represented as a node. Connected nodes will communicate through a message broker - Apache Pulsar. Every node consumes events that are emitted by connected nodes. Listeners (consumers) will filter incoming events and will use the ones which relate to a specific city.

每个站点为一个节点，相连节点通过消息 broker——Apache Pulsar 通信。节点消费其相连节点发布的事件。消费者过滤传入事件后消费与特定城市相关的事件。

We should have a way to control the behavior of the simulator. One way of doing this is adding HTTP endpoints, which can be used for manual intervention. By sending an HTTP request, the user will be able to add new trains to the system.

有两种方式可以控制模拟器的行为，一是添加可用于人工干预的 HTTP 端点。用户通过发送 HTTP 请求向系统中添加新列车。

In this tutorial we won’t persist any data, so we won’t use any database or cache system. All the data will be kept in memory. For that, we can use a high-level concurrency mechanism like <code>[Ref](https://zio.dev/docs/datatypes/datatypes_ref)</code>.

我们不持久保存任何数据，无需使用数据库或缓存系统，将所有数据保存在内存中。因此我们可以使用类似于 [Ref](https://zio.dev/docs/datatypes/datatypes_ref) 的高级并发机制。

At the core of our system will be Apache Pulsar. It will be responsible for communication between nodes. After every change in its state system should emit a new event. The event would describe an action that already has happened. This means every event should have a `timestamp`. Also, every event should have a `trainId`, which denotes an identification number of a specific train. At the very beginning there will be two events:

Apache Pulsar 是系统的核心，负责节点间通信。一旦状态发生改变，系统应该发布描述这一动作的新事件。也就是说，每个事件都应该有一个时间戳。此外，每个事件应有一个列车 id，代表特定列车的标识号码。初始时，有两个事件：



*   `Departed` - is emitted when a train has departed from a city.
*   `Arrived` - is emitted when a train has arrived in a city. 

Both events contain generic train information: its identification number, city of departure, destination city, expected time of arrival and event timestamp.



*   出发（Departed）事件——列车出发时发布出发事件。
*   到达（Arrived）事件——列车到达时发布到达事件。

    这两个事件包含关于列车的基本信息：列车标识号码、出发城市、目的地城市、预计到达时间和事件时间戳。


Every city consumes events from connected cities. For example, Zurich consumes Bern events and doesn’t know about events coming from Geneva. Event consumer in Zurich should make sure that whenever Bern emits a `Departed` event with the destination city set as Zurich it should be captured. Every city will have its own topic. In the future, if we will need to optimize this we can split generic “city topic” into a few, more specific topics. For 3 cities there will be 3 topics.

每个城市都消费来自相连城市的事件。例如，苏黎世消费来自伯尔尼的事件，但不关注来自日内瓦的事件。苏黎世的事件消费者应确保能够捕获到由伯尔尼发车并且苏黎世为目的地的事件。每个城市对应一个 topic，3 个城市就对应 3 个 topic。需要优化时，可以把通用的 "城市 topic "分成几个更具体的 topic。

As a glue between our business logic and Apache Pulsar we will use [neutron](https://github.com/cr-org/neutron).

业务逻辑通过 [Neutron](https://github.com/cr-org/neutron) 连接到 Apache Pulsar。

Every consumed topic will be converted to a `fs2` stream which we already know how to handle. If you don’t, I really recommend going through [fs2 guide](https://fs2.io/guide.html), but it is not mandatory to understand the code below.

每个被消费的 topic 都会转换为 fs2 流，如果你不了解如何处理 fs2 流，可以参考 [fs2 指南](https://fs2.io/#/guide)，本文代码不会涉及到这部分内容。

The application will be written using the Tagless Final technique based on `cats` library and using `ZIO` as the runtime effect. This is a controversial decision and I have different thoughts about it but it deserves a separate blog post.

我基于 cats 库的 Tagless Final 技术编写了这一应用程序，并以 ZIO 作为运行时 [effect](https://typelevel.org/cats-effect/)。


# Apache Pulsar in ~10 sentences Apache 


# Pulsar 简介

Apache Pulsar is a distributed messaging and streaming platform. It can be used to build a highly scalable system on top of it. Parts of a system(s) can communicate using messages in up to millions of topics. From a developer perspective, it can be treated as a black box, but I would recommend learning more on how it works under the hood.

Apache Pulsar 是分布式消息和流平台，可用于搭建高扩展性系统。系统内部通过消息进行通信，topic 数量可达数百万个。从开发者的角度来讲，Apache Pulsar 可以看作是一个黑匣子，但我建议多了解它的底层工作原理。为了更好地理解本文中的操作，我先介绍几个概念：

To understand this article you will need to be familiar with a few concepts (simplified):



*   `Topic` - channel for transmitting messages. There are 2 types:
    *   `Persistent` - message data is persistently stored;
    *   `NonPersistent` - message data is never persistently stored and is kept in memory. All in-transit messages will be lost if a Pulsar broker will go down;
*   `Producer` - process that attaches to a topic and publishes messages;
*   `Consumer` - process that attaches to a topic via a subscription and then receives messages;
*   `Subscription` - configuration rule that determines how messages are delivered to consumers. There are 4 subscription types:
    *   `Exclusive` - Single consumer. Every other consumer of a subscription in this mode will raise an error;
    *   `Failover` - Multiple consumers can be attached in this mode, but only one will receive messages;
    *   `Shared` - Multiple consumers will receive messages in a round-robin distribution;
    *   `Key Shared` - Multiple consumers will receive messages distributed by key (one key will always be delivered to the same consumer).

·topic——信息传输的媒介。topic 分为两种：

1.持久化 topic——持久存储消息数据。

2.非持久化 topic——不持久存储消息数据，将消息保存在内存中。如果 Pulsar broker 宕机，所有传输中的消息都会丢失。

·生产者——与 topic 相连，用于发布消息。

·消费者——通过订阅与 topic 相连，用于接收消息。

·订阅——制定向消费者发布消息的配置规则。Pulsar 支持四种订阅类型：

1.独占——单一消费者，如有多个消费者同时订阅则会引发错误；

2.故障转移——多个消费者，但只有一个消费者能收到消息；

3.共享——多个消费者，以轮询方式接收消息；

4.Key_Shared——多个消费者，按 key 分发消息（一个消费者对应一个 key）。

Imagine a system, that is emitting events, which are processed by a `Producer`, that publishes them to a `Topic`, which is listened to by a `Consumer` of another system, that is attached using a `Subscription`.

消息系统发布事件后，由生产者处理这些事件并发布到 topic 上，另一个系统里的消费者通过订阅连接到这个 topic。

Simple as that. Almost. In a fact, it is a slightly more complicated process and if you want to learn more about Apache Pulsar [check out its documentation](https://pulsar.apache.org/docs/en/pulsar-2.0/).

点击[这里](http://pulsar.apache.org/docs/en/standalone/)了解更多关于 Apache Pulsar 的信息。


# The multi-billion business logic 


# 业务逻辑

Earlier we mentioned two events that can happen on the railway - train departure and arrival. It is time to define them:

上文提到铁路网中会发生的两个事件——列车的出发与到达。定义这两个事件的代码如下：


```
```
case class Departed(id: EventId, trainId: TrainId, from: From, to: To, expected: Expected, created: Timestamp) extends Event
case class Arrived(id: EventId, trainId: TrainId, from: From, to: To, expected: Expected, created: Timestamp)  extends Event
```
```


These events provide general information about an action that already happened in the system: unique event id, train id, departure city, destination city, expected arrival time, and actual event timestamp. Later we can add additional information such as platform number, number of cars, etc.

事件需包含系统中已发生动作的基本信息：唯一的事件 id、列车 id、出发城市、目的地城市、预计到达时间和实际事件时间戳。我们以后还可以添加站台号等信息。

For the sake of simplicity of the tutorial let’s limit the amount of data we need for the system to work. Every field in the event is strongly typed as we don’t want to accidentally mix up things (for example, destination and departure cities).

为确保本文简单易懂，我们对本系统工作所需的数据量加以限制。为了便于区分事件中的字段（比如目的地和出发城市），所有字段都使用强类型。

As we don’t have an automatic system that would detect train arrival or departure, we will have to control our network manually. We could imagine a real person (a train dispatcher) who controls the network. This person would press buttons on a shiny panel full of buttons and lights. We won’t have a shiny UI, but we can build an API for it. At the core of this API would be 2 simple commands:

由于没有可以自动检测火车到达或出发的系统，我们需要手动控制铁路网。假设有一名火车调度员在通过按钮和仪表盘来控制铁路网络，我们虽然没有炫酷的 UI，但可以搭建 API，API 的核心是两个简单的命令，用于触发车站的业务逻辑：


```
```
case class Arrival(trainId: TrainId, time: Actual)
case class Departure(id: TrainId, to: To, time: Expected, actual: Actual)
```
```


These commands would trigger the business logic of a train station.


## **Train departure **


## **列车出发**

We start by creating a train departure. This is the first command that we should be able to send using a simple cURL:

让我们从创建火车出发开始吧！这个命令比较简单，可以通过 cURL 发送:


```
```
curl --request POST \
  --url http://localhost:8081/departure \
  --header 'content-type: application/json' \
  --data '{
	"id": "153",
	"to": "Zurich",
	"time": "2020-12-03T10:15:30.00Z",
	"actual": "2020-12-03T10:15:30.00Z"
}'
```
```


This command is assuming Bern service node is running on port 8081. Every node is running an HTTP server and should be able to handle this request. As the HTTP server, we will use `Http4s` library. Our first route definition looks like this:

上述命令假设伯尔尼服务节点在 8081 端口运行，每个节点都运行 HTTP 服务器，也都能够处理这一请求。我们使用 Http4s 库作为 HTTP 服务器，第一个线路定义如下：


```
```
case req @ POST -> Root / "departure" =>
  req
    .asJsonDecode[Departure]
    .flatMap(departures.register)
    .flatMap(_.fold(handleDepartureErrors, _ => Ok()))
```
```


Here we call `Departures` service, which we haven’t defined yet. Let’s do it now. The only thing service should do is to register a departing train:

调用 Departures 服务仅需注册一列出发的火车： 


```
```
trait Departures[F[_]] {
  def register(departure: Departure): F[Either[DepartureError, Departed]]
}
```
```


There are different ways to do data validation in Scala. I’ve picked the most straightforward and explicit way - returning an `Either` with a custom error. If registering the train succeeds we return a `Departed` event. If not, there is an error that should be handled by the caller.

Scala 支持多种验证数据的方式，我选择最直接的一种——返回带有自定义错误的 Either。如果火车注册成功，则返回 Departed 事件；否则，返回错误。

For simplicity reasons let’s call the message producer inside the `Departures` service implementation. Wait, we didn’t implement it yet? No need to wait, let’s do it now. Inside the `Departures` companion object we create function `make`:

为确保本文简单易懂，我们会在 Departures 服务执行过程中调用消息生产者。首先需执行 Departures 服务，即在 Departures 伴生对象中创建 make 函数 ：


```
```
object Departures {
  def make[F[_]: Monad: UUIDGen: Logger](
      city: City,
      connectedTo: List[City],
      producer: Producer[F, Event]
  ): Departures[F] = new Departures[F] {
    def register(departure: Departure): F[Either[DepartureError, Departed]] = ???
  }
}
```
```


To implement `Departures` interface we set boundaries for our effect `F`: it should have instances of `UUIDGen` and `Logger`. In this application I’ve created dummy `UUIDGen` and `Logger` interfaces, don’t use them in your multi-billion startup - you can find something better.

为实现 Departures 接口，我们要给 effect F 设置边界：需有 UUIDGen 和 Logger 实例。我已经在程序中创建了虚拟的 UUIDGen 和 Logger 接口。

Also `F` should have a `Monad` instance to chain function calls.

F 还应有 Monad 实例，用于连接函数调用。

Let’s start by implementing a validation logic that will check if `Departure` is valid. We will have only 1 check - if the destination city is in the list of connected cities.

首先执行验证逻辑，检查出发事件是否有效。我们只需检查目的地城市是否在相连城市列表中：


```
```
def validated(departure: Departure)(f: F[Departed]): F[Either[DepartureError, Departed]] = {
  val destination = departure.to.city

  connectedTo.find(_ === destination) match {
    case None =>
      val e: DepartureError = DepartureError.UnexpectedDestination(destination)
      F.error(s"Tried to departure to an unexpected destination: $departure")
       .as(e.asLeft)
    case _ =>
      f.map(_.asRight)
  }
}
```
```


If the destination city is not on the list, we log an error message and return the error as the result. Otherwise, we create a `Departed` event and return it as the result.

如果目的地城市不在列表中，则生成错误信息日志并返回错误。否则创建 Departed 事件并将其作为结果返回。

Let’s see how `register` function can be implemented:

接下来需要实现注册功能，示例代码如下：


```
```
def register(departure: Departure): F[Either[DepartureError, Departed]] =
  validated(departure) {
    F.newEventId
      .map {
        Departed(
          _,
          departure.id,
          From(city),
          departure.to,
          departure.time,
          departure.actual.toTimestamp
        )
      }
      .flatTap(producer.send_)
  }
```
```


We start by validating the destination city. If it is valid then we generate a `newEventId` which we use to create a new `Departed` event. This event is published to the `city` topic in Pulsar using `producer` that we passed to `make` function. Simple! You can find the final version of `Departures` [here](https://github.com/psisoyev/train-station/blob/ec3841784841ebc03c6d1cdc3347b04065e81d1c/service/src/main/scala/com/psisoyev/train/station/departure/Departures.scala#L13).

先验证目的地城市，若有效，生成一个 newEventId，用于创建新的 Departed 事件，该事件将通过传递给 make 函数的生产者发布到 Pulsar 的城市 topic。点击[这里](https://github.com/psisoyev/train-station/blob/ec3841784841ebc03c6d1cdc3347b04065e81d1c/service/src/main/scala/com/psisoyev/train/station/departure/Departures.scala#L13) 查看 Departures 事件的最终版本。


## **Expecting departed trains **


## **预计出发列车**

We have learned how to spawn trains in our system. If a train has departed from Zurich to Bern, then Bern should be notified about it.

我们已经了解了如何生成列车。如果一列火车从苏黎世开往伯尔尼，那么伯尔尼会收到相应通知。

Bern is listening for Zurich events and as soon as there is a `Departed` event with Bern set as the destination, it should add it to the expected train list. Let’s leave message consuming for dessert and now will focus on business logic. We define a `DepartureTracker` that expects a `Departed` event:

伯尔尼收听来自苏黎世的事件，一旦有把伯尔尼设为目的地的 Departed 事件，就将其加入预期列车表中。现在我们只关注业务逻辑，后文会再讨论消息消费。为预期出发事件定义 DepartureTracker，示例代码如下：


```
```
trait DepartureTracker[F[_]] {
  def save(e: Departed): F[Unit]
}
```
```


This service will be a sink in our the `Departed` event flow, so we don’t care about the return type and we don’t expect any validation errors. As we did with `Departures`, we create a companion object, where we define a `make` function:

该服务会成为 Departed 事件流中的 sink，所以我们不关注返回类型，也不希望出现任何验证错误。和上文 Departures 服务一样，先创建伴生对象，定义 make 函数：


```
```
def make[F[_]: Applicative: Logger](
    city: City,
    expectedTrains: ExpectedTrains[F]
  ): DepartureTracker[F] = new DepartureTracker[F] {
    def save(e: Departed): F[Unit] =
      val updateExpectedTrains =
        expectedTrains.update(e.trainId, ExpectedTrain(e.from, e.expected)) *>
          F.info(s"$city is expecting ${e.trainId} from ${e.from} at ${e.expected}")


      updateExpectedTrains.whenA(e.to.city === city)
  }
```
```


We have a dependency on `ExpectedTrains` service. This service is the storage of incoming trains. We will implement it shortly. Here we have implemented `save` function, which works only when the destination city of the incoming train is the expected one. For example, both Geneva and Zurich are consuming events from Bern. When Bern emits a `Departed` event, one city will just ignore the message.

The other city, which is the destination city, will update the expected train list and create a log message.

我们依赖于 ExpectedTrains 服务。ExpectedTrain 是存储进站列车的服务，我们很快就能实现该服务。我们实现了 save 函数，只有进站列车的目的地城市与预期城市相符时，该函数才会执行。例如，日内瓦和苏黎世均消费来自伯尔尼的事件。伯尔尼发出 Departed 事件时，其中一个城市会忽略此消息，而另一个城市，即目的地城市，会更新预期列车表，创建日志消息。

Our expected train storage has a minimal set of functionality:

预期列车存储中至少包含以下函数：


```
```
trait ExpectedTrains[F[_]] {
  def get(id: TrainId): F[Option[ExpectedTrain]]
  def remove(id: TrainId): F[Unit]
  def update(id: TrainId, expectedTrain: ExpectedTrain): F[Unit]
}
```
```


Even if we will try to remove a train that doesn’t exist in our system, we will treat it as a non-failure. In some business cases it might be wrong and a sign of system malfunctioning, but in this particular case we will ignore that kind of errors. For our MVP we will store data in memory without persisting it anywhere:

即使我们尝试删除不存在于系统中的列车，也不会操作失败。在某些业务情况下可能会出现系统故障的错误，但在这种特殊情况下，我们会忽略这一错误。整个测试过程中，数据一直存储在内存中，不持久保存。


```
```
def make[F[_]: Functor](
    ref: Ref[F, Map[TrainId, ExpectedTrain]]
  ): ExpectedTrains[F] = new ExpectedTrains[F] {
    def get(id: TrainId): F[Option[ExpectedTrain]] = 
      ref.get.map(_.get(id))
    def remove(id: TrainId): F[Unit] = 
      ref.update(_.removed(id))
    def update(id: TrainId, train: ExpectedTrain): F[Unit] = 
      ref.update(_.updated(id, train))
  }
```
```


We use <code>[Ref](https://zio.dev/docs/datatypes/datatypes_ref)</code> as our high-level concurrency mechanism.

我们在这一应用程序中使用 [Ref](https://zio.dev/docs/datatypes/datatypes_ref) 作为高级并发机制。


## **Train arrival **


## **列车到达**

The final part of the business logic trilogy is train arrival. Similarly to train departure, we create an HTTP endpoint, which we can call using a simple cURL POST request:

业务逻辑三部曲的最后一部分是列车到达。与列车出发类似，先创建一个 HTTP 端点，可以用简单的 cURL POST 请求来调用：


```
```
curl --request POST \
  --url http://localhost:8081/arrival \
  --header 'Content-Type: application/json' \
  --data '{
	"trainId": "123",
	"time": "2020-12-03T10:15:30.00Z"
}'
```
```


This request will be handled by Http4s routes:

再由 Http4s 路线处理请求：


```
```
case req @ POST -> Root / "arrival" =>
  req
    .asJsonDecode[Arrival]
    .flatMap(arrivals.register)
    .flatMap(_.fold(handleArrivalErrors, _ => Ok()))
```
```


We call `register` method on a twin service of `Departures` service we’ve seen earlier - `Arrivals`. `Arrivals` services also has only one method:

Arrivals 服务类似于上文介绍的 Departures 服务。Arrivals 服务中也只有一个方法，即 register 方法：


```
```
trait Arrivals[F[_]] {
  def register(arrival: Arrival): F[Either[ArrivalError, Arrived]]
}
```
```


Again we start with validation of the request:

然后需要验证请求，示例代码如下：


```
```
def validated(arrival: Arrival)(f: ExpectedTrain => F[Arrived]): F[Either[ArrivalError, Arrived]] =
  expectedTrains
    .get(arrival.trainId)
    .flatMap {
      case None =>
        val e: ArrivalError = ArrivalError.UnexpectedTrain(arrival.trainId)
        F.error(s"Tried to create arrival of an unexpected train: $arrival")
         .as(e.asLeft)
      case Some(train) =>
        f(train).map(_.asRight)
    }
```
```


Here we check if the arrived train was expected and if it is, then we create an `Arrived` event. Otherwise, we create an error and log it. If you take a look at the implementation you will notice similarities to the other `register` method we’ve seen earlier:

检查到达的列车是否与预期相符，若相符，则创建 Arrived 事件；否则，生成错误日志。列车到达事件中 register 方法的实现中与之前 register 方法的实现类似：


```
```
def register(arrival: Arrival): F[Either[ArrivalError, Arrived]] =
  validated(arrival) { train =>
    F.newEventId
      .map {
        Arrived(
          _,
          arrival.trainId,
          train.from,
          To(city),
          train.time,
          arrival.time.toTimestamp
        )
      }
      .flatTap(a => expectedTrains.remove(a.trainId))
      .flatTap(producer.send_)
  }
```
```


Comparing to `Departures` the difference is that we not only publish the new event but also have another side-effect - we remove the arrived train from the list of expected trains.

与 Departures 相比，到达事件不仅发布了新事件，还把到达列车从预计出发列车列表中删除。

That is all the business logic needed for the MVP. This logic is covered by unit tests and they are [available on GitHub](https://github.com/psisoyev/train-station/tree/ec3841784841ebc03c6d1cdc3347b04065e81d1c/service/src/test/scala/com/psisoyev/train/station). Unit tests are implemented using ZIO Test. If you want to learn more about it, you can check one of my previous articles - [Effective testing with ZIO Test](https://scala.monster/zio-test/).

以上为全部业务逻辑，代码已经通过单元测试（使用 [ZIO Test](https://scala.monster/zio-test/) 实现），可参考 [GitHub 文件](https://github.com/psisoyev/train-station/tree/ec3841784841ebc03c6d1cdc3347b04065e81d1c/service/src/test/scala/com/psisoyev/train/station) 。


# The dessert 


# 消息消费

Remember I said we’ll leave message consuming for dessert? The time has come!

In this section, we will wire all logical services together.

这一节主要讲消息消费，也会把所有逻辑服务连在一起。


## **Building resources **


## **创建资源**

Let’s start by creating the required resources. A city node needs 4 things: configuration, event producer, event consumers, and a `Ref` where we will store `ExpectedTrains`. We can combine them in one case class and create outside of `Main` class:

首先创建所需资源。一个城市节点包含四个组件：配置、事件生产者、事件消费者，以及存储 ExpectedTrains 的 Ref。我们可以把这四种资源在一个 case class 中组合起来，在 Main 类外创建：


```
```
final case class Resources[F[_], E](
  config: Config,
  producer: Producer[F, E],
  consumers: List[Consumer[F, E]],
  trainRef: Ref[F, Map[TrainId, ExpectedTrain]]
)
```
```


`Config` is read from environment variables. For this purpose, we will use a library called [ciris](https://github.com/vlovgr/ciris). We won’t focus on it too much, as the configuration is boring. You can find the final implementation of `Config` on [GitHub](https://github.com/psisoyev/train-station/blob/ec3841784841ebc03c6d1cdc3347b04065e81d1c/server/src/main/scala/com/psisoyev/train/station/Config.scala#L13).

Producers and consumers are much more interesting to see. To create them we will use library called [neutron](https://github.com/cr-org/neutron/) developed by [Chatroulette](https://about.chatroulette.com/).

我们使用 [ciris](https://github.com/vlovgr/ciris) 库从环境变量中读取 Config。关于配置，可以参考 [GitHub 文件](https://github.com/psisoyev/train-station/blob/ec3841784841ebc03c6d1cdc3347b04065e81d1c/server/src/main/scala/com/psisoyev/train/station/Config.scala#L13)。我们使用 [Chatroulette](https://about.chatroulette.com/) 开发的 [Neutron](https://github.com/cr-org/neutron/) 库来创建生产者和消费者。

First, we need to establish a connection with Apache Pulsar cluster. For this, we create an instance of `Pulsar` object:

首先，创建一个 Pulsar 对象实例，用于与 Apache Pulsar 集群建立连接：


```
```
Pulsar.create[F](config.pulsar.serviceUrl)
```
```


As we can see, it requires only a `serviceUrl`. In return, we will get a `Resource[F, PulsarClient]`. This resource can be used to create producers and consumers. Before we create a producer, we need to create a `Topic` object, which contains topic configuration:

以上操作仅需 serviceUrl，我们会得到 Resource[F, PulsarClient]，可以用来创建生产者和消费者。创建生产者之前，应该先创建包含 topic 配置的 topic 对象：


```
```
def topic(config: PulsarConfig, city: City) =
  Topic(
    Topic.Name(city.value.toLowerCase),
    config
  ).withType(Topic.Type.Persistent)
```
```


The topic name simply will be a city name and it will be a `Persistent` topic. So that we don’t lose any unacknowledged messages. Also, as a part of the configuration, we pass `namespace` and `tenant`. You can learn more about these concepts [in the official documentation](https://pulsar.apache.org/docs/en/concepts-multi-tenancy/).

Topic 名就是城市名，而且是持久化 topic，这样，任何未确认的消息都不会丢失。另外，作为配置的一部分，我们传递了命名空间和租户。关于命名空间和租户的更多信息，可以查阅 [Pulsar 文档](https://pulsar.apache.org/docs/en/next/standalone/)。

Creating a producer is a simple one-liner:

创建生产者操作只是简单的一行：


```
```
def producer(client: Pulsar.T, config: Config): Resource[F, Producer[F, E]] =
  Producer.create[F, E](client, topic(config.pulsar, config.city))
```
```


There are several ways of creating a producer, we’ll use the simplest one. It requires only Pulsar client we created earlier and a topic.

创建生产者的方法有很多，我们选择最简单的一种，只需使用之前创建的 Pulsar 客户端和一个 topic。

Creating a consumer requires slightly more actions, as we also have to create a `Subscription`:

创建消费者所需操作稍多，因为还要创建订阅：


```
```
def consumer(client: PulsarClient, config: Config, city: City): Resource[F, Consumer[F, E]] = {
  val name         = s"${city.value}-${config.city.value}"
  val subscription =
          Subscription
            .Builder
            .withName(Subscription.Name(name))
            .withType(Subscription.Type.Failover)
            .build

  Consumer.create[F, E](client, topic(config.pulsar, city), subscription)
}
```
```


We create a subscription with a name corresponding to a connected city name plus the city we’d like to run. By default, we will use the `Failover` subscription type as it allows us to run 2 instances in parallel, just in case one will go down.

创建订阅，设置订阅名称为相连的城市名称与火车经停城市名组合。默认使用 Failover 订阅类型，并行运行 2 个实例（以防其中一个实例宕机）。

Together with the required `Ref` we can finally build our `Resources`:

加上所需 Ref，我们终于可以创建 Resources 了：


```
```
for {
  config    <- Resource.liftF(Config.load[F])
  client    <- Pulsar.create[F](config.pulsar.url)
  producer  <- producer(client, config)
  consumers <- config.connectedTo.traverse(consumer(client, config, _))
  trainRef  <- Resource.liftF(Ref.of[F, Map[TrainId, ExpectedTrain]](Map.empty))
} yield Resources(config, producer, consumers, trainRef)
```
```


Note, that we create a list of consumers using `traverse` method on `connectedTo` list of cities. As always, you can find the final result [on GitHub](https://github.com/psisoyev/train-station/blob/b7447c40f88e19020c33f799bcbb9c5b94a7d5ac/server/src/main/scala/com/psisoyev/train/station/Resources.scala#L11).

请注意，我们使用 traverse 方法在 connectedTo 城市列表中创建了一份消费者列表，点击 [GitHub 文件](https://github.com/psisoyev/train-station/blob/b7447c40f88e19020c33f799bcbb9c5b94a7d5ac/server/src/main/scala/com/psisoyev/train/station/Resources.scala#L11)查看最终结果。


## **Starting the engine **


## **启动引擎**

We will use `zio.Task` as the effect type. It contains the least amount of type parameters so should be easier to understand for those, who are not familiar with ZIO.However, if you want to see a few more type parameters in action, you can read my [introduction to ZIO](https://scala.monster/welcome-zio/).

我们在应用程序中使用 zio.Task 作为 effect 类型。zio.Task 包含的类型参数最少，对于不熟悉 ZIO 的人来说，zio.Task 更易理解。如果你想了解更多类型参数，可以参考[ZIO简介](https://scala.monster/welcome-zio/)。

First, we create the `Resources` class we’ve defined earlier:

首先，创建之前定义过的 Resources 类：


```
```
Resources
  .make[Task, Event]
  .use {
    case Resources(config, producer, consumers, trainRef) => ???
  }
```
```


Nothing has changed here - same 4 parameters as before. We start with initializing services and creating `routes` for the HTTP server:

依然是 4 个参数。先初始化服务，为 HTTP 服务器创建路线：


```
```
val expectedTrains   = ExpectedTrains.make[Task](trainRef)
val arrivals         = Arrivals.make[Task](config.city, producer, expectedTrains)
val departures       = Departures.make[Task](config.city, config.connectedTo, producer)
val departureTracker = DepartureTracker.make[Task](config.city, expectedTrains)

val routes = new StationRoutes[F](arrivals, departures).routes.orNotFound
```
```


Then we create the HTTP server:

创建 HTTP 服务器：


```
```
val httpServer = Task.concurrentEffectWith { implicit CE =>
  BlazeServerBuilder[Task](ec)
    .bindHttp(config.httpPort.value, "0.0.0.0")
    .withHttpApp(routes)
    .serve
    .compile
    .drain
}
```
```


Nothing new here for those, who are already familiar with Http4s. Who is not familiar with Http4s, I encourage you to [read the docs](https://http4s.org/). The next step is to start consuming incoming messages and to build a stream out of them:

如果你很了解 Http4s，那么以上操作应该不难理解。若不了解，点击[这里]([https://http4s.org/](https://http4s.org/))查看相关文档。开始消费传入消息，并创建一个流：


```
```
val departureListener =
  Stream
    .emits(consumers)
    .map(_.autoSubscribe)
    .parJoinUnbounded
    .collect { case e: Departed => e }
    .evalMap(departureTracker.save)
    .compile
    .drain
```
```


Let’s take a closer look at what is going on here. We are using FS2 library to create a stream of events. First, we create a stream of consumers and call `autoSubscribe` method on every consumer. This will start a subscription on the topic. Then we merge all streams into one stream using `parJoinUnbounded`. After that, we remove every other message except `Departed` using `collect` method. The final step is to call `save` method on our `departureTracker` which we implemented earlier. Then the stream gets compiled and drained.

简而言之，我们使用 FS2 库创建了事件流。首先，创建消费者流，对每个消费者调用 autoSubscribe 方法，用于订阅 topic，再通过 parJoinUnbounded 把所有流合在一起，然后，用 collect 方法删除 Departed 以外的所有消息。最后，在之前实现的 departureTracker 上调用 save 方法，编译并排出流。

Now we have 2 final streams: HTTP server and incoming Pulsar messages. At this point we are already handling all the messages and simply need to run the streams, so we just zip them in parallel and drop the result:

现在有两个最终流：HTTP 服务器和 Pulsar 的传入消息。此时我们已经处理完了所有消息，只需运行流，即并行压缩并丢弃结果：


```
```
departureListener
  .zipPar(httpServer)
  .unit
```
```


That is it. Our `Main` class consists of a few simple blocks of code which we can easily read and maintain.

组成 Main 类的代码块都比较简单，读取和维护也相对容易。


# Summary 


# 结语

Thank you for reading up to this point. In this article, I’ve given you an example of an event-driven system. Together we’ve built a Swiss railway network simulator MVP. Of course, some decisions and choices can be challenged. You must remember that this is not a tutorial on how you _should_ build your next multi-billion startup, but how you _can_ build one.

本文给出了事件驱动系统的例子，按步骤梳理了业务逻辑，模拟了瑞士铁路网。你可以在本文示例代码的基础上进行修改和拓展。

We’ve seen some capabilities of Apache Pulsar, but I promise there is much more than that. The system has really surprised me with its simplicity and capabilities. We’ve built a simple but distributed system consisting of several nodes that are communicating using messaging on top of Apache Pulsar. All of this on the functional stack using Tagless Final approach on top of `cats` library, where `ZIO` `Task` is used as a main effect type.

本文中使用到了 Apache Pulsar 的部分功能，但 Pulsar 不止于此，它操作简易，功能强大。我们搭建了一个简单的分布式系统，由几个节点组成，这些节点在 Apache Pulsar 上使用消息传递进行通信。本应用程序使用基于 cats 库的 Tagless Final 技术编写，其中 ZIO Task 为主要的 effect 类型。

Also, we’ve tried [Neutron](https://github.com/cr-org/neutron/) which is still in active development but is used in production by [Chatroulette](https://about.chatroulette.com/). 

此外，我们还尝试了 [Neutron](https://github.com/cr-org/neutron/)，虽然 Neutron 已用于 [Chatroulette](https://about.chatroulette.com/) 生产，但仍在开发中。

The final version of the app is available on [GitHub](https://github.com/psisoyev/train-station/). You can find instructions on how to run it in the readme of the project.

点击[这里]([https://github.com/psisoyev/train-station/](https://github.com/psisoyev/train-station/))查看本程序的最终版本，操作指南可见 readme 部分。

Let me know if you have any questions or suggestions in the comment section! Feedback is also very welcome, thank you! Cheers!

