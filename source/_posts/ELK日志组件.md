# 1 引言

## 1.1 编写目的

目的是帮助研发、测试掌握和了解设计日志中心的开发目的以及开发方式。

## 1.2 背景

什么是日志？ 日志就是程序产生的，遵循一定格式（通常包含时间戳）的文本数据。通常日志由服务器生成，输出到不同的文件中，一般会有系统日志、 应用日志、安全日志。这些日志分散地存储在不同的机器上。通常当系统发生故障时，工程师需要登录到各个服务器上，使用 grep / sed / awk 等 Linux 脚本工具去日志里查找故障原因。在没有日志系统的情况下，首先需要定位处理请求的服务器，如果这台服务器部署了多个实例，则需要去每个应用实例的日志目录下去找日志文件。每个应用实例还会设置日志滚动策略（如：每天生成一个文件），还有日志压缩归档策略等。

因此，日志数据在以下几方面具有非常重要的作用：

- **数据查找**：通过检索日志信息，定位相应的 bug ，找出解决方案
- **服务诊断**：通过对日志信息进行统计、分析，了解服务器的负荷和服务运行状态
- **数据分析**：可以做进一步的数据分析

针对这些问题，为了提供分布式的实时日志搜集和分析的监控系统，我们采用了业界通用的日志数据管理解决方案——即基于ELK的日志收集中心。该模块将会的是公司微服务平台组成中重要一环，它能给开发者、测试者、运维者、以及用户提供更加便利的途径去查看项目的异常信息，还能及时的发出异常告警信息来提示运维者及时的进行系统的维护。



## 1.3 术语定义

| 术语          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| ELK           | ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash, Kibana = **搜索+过滤+可视化** |
| Flilebeat     | 轻量级的日志收集器                                           |
| ElasticSearch | 开源分布式搜索引擎，提供搜集、分析、存储数据三大功能         |
| Logstash      | 日志的搜集、分析、过滤日志的工具，支持大量的数据获取方式     |
| Kibana        | 用于**可视化**，能提供友好的数据分析界面                     |
| lucene        | 一套信息**检索工具包**（包含索引结构、排序、搜索规则，不包括搜索引擎）， Elasticsearch基于luncene进行封装增强 |



## 1.4 参考资料



* https://www.elastic.co/guide/en/logstash/6.1/index.html

* https://docs.fluentd.org

* https://blog.51cto.com/13527416/2132270

* https://elasticsearch.cn/article/190

* https://www.elastic.co/guide/en/beats/filebeat/6.1/filebeat-overview.html

- 查看[ELK 官方网站](https://www.elastic.co/)，了解 ELK+Beats 等产品的介绍；
- 搜索[ELK 论坛](https://discuss.elastic.co/)，寻找关于 ELK 在实际使用中的问题及解答；
- 查看文章[部署和扩展 ELK](https://www.elastic.co/guide/en/logstash/current/deploying-and-scaling.html)，了解更多关于如何扩展 ELK 架构的知识；
- 参考[使用 Nginx 实现 Kibana 登陆认证](http://www.cnblogs.com/yjmyzz/p/filebeat-turorial-and-kibana-login-setting-with-nginx.html)博客，学习如何配置 Nginx 作为 Kibana 的反向代理，且实现登陆认证；
- 查阅[Filebeat 文档](https://www.elastic.co/guide/en/beats/filebeat/1.2/filebeat-overview.html)，系统全面地学习 Filebeat 的安装配置细节；
- 查阅[Elasticsearch 文档](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/getting-started.html)，系统全面地学习 Elasticsearch 的安装配置细节；
- 查阅[Logstash 文档](https://www.elastic.co/guide/en/logstash/2.3/installing-logstash.html)，系统全面地学习 Logstash 的安装配置细节；
- 查阅[Kibana 文档](https://www.elastic.co/guide/en/kibana/4.5/introduction.html)，系统全面地学习 Kibana 的安装配置细节。



# 2 总体设计

## 2.1 方案选型

一个完整的日志中心一般包含一下几个特点：

​	● 收集－能够采集多种来源的日志数据

​	● 传输－能够稳定的把日志数据传输到中央系统

​	● 存储－如何存储日志数据

​	● 分析－可以支持 UI 分析

​	● 告警－能够提供错误报告，监控机制



由此产生的产品或者方案如下：

* Scribe – Scribe是由Facebook开源的，主打可扩展性和可靠 性的日志聚合服务，由C++编写，使用Thrift通信协议，但是 实际上可以支持任何语言。

* Flume – Flume是apache的项目，旨在收集，聚合，移动大 量的日志数据，最终将数据存放到HDFS上。

* Chukwa – Chukwa是另外一个apache项目用来将日志放到HDFS中。

* fluent – fluent 和logstash类似，也支持各种输入和输出，虽然不支持任何存储层，但是可配置。另外，它的设计原则是方便安装和低开销。

* kafka – 由LinkIn开发的用来处理他们的活动流（activity stream），现在是apache的孵化项目。虽然kafka可用作为日志 收集器，不过不是他的主要使用场景。需要配置Zookeeper来 管理集群。

* `ELK` - ELK相对于其他单一产品或系统提供了一整套解决方案，它是三个软件产品的首字母缩写，Elasticsearch、Logstash 和 Kibana。这三款软件都是开源软件，通常是配合使用，而且又先后归于 [http://Elastic.co](https://www.elastic.co/cn/)公司名下，故被简称为ELK协议栈。

  

### 介绍

#### ![ElasticSearch](https://gitee.com/JungwooJava/cloudimg/raw/master/ElasticSearch.png)Elasticsearch

Elasticsearch 是一个实时的分布式搜索和分析引擎，它可以用于全文搜索，结构化搜索以及分析。	它是一个建立在全文搜索引擎 Apache Lucene 基础上的搜索引擎，使用 Java 语言编写。主要特点如	下：

​		❖ 实时分析

​		❖ 分布式实时文件存储，并将每一个字段都编入索引

​		❖ 文档导向，所有的对象全部是文档

​		❖ 高可用性，易扩展，支持集群（Cluster）、分片和复制（Shards 和 Replicas）

​		❖ 接口友好，支持 JSON



```text
Elasticsearch最关键的就是提供强大的索引能力。

Elasticsearch索引的精髓：一切设计都是为了提高搜索的性能

Elasticsearch优势
1.横向可扩展性:只需要增加服务器，做一点儿配置，启动一下Elasticsearch就可以并入集群。
2.分片机制提供更好的分布性:同一个索引分成多个分片, 分而治之的方式可提升处理效率。
3.高可用:提供复制( replica) 机制，一个分片可以设置多个复制，使得某台服务器在宕机的情况下，集群仍旧可以照常运行，并会把服务器宕机丢失的数据信息复制恢复到其他可用节点上。
4.使用简单:共需一条命令就可以下载文件，然后很快就能搭建一一个站内搜索引擎。

Elasticsearch存储结构
Elasticsearch是文件存储，Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档
```



#### ![Logstash](https://gitee.com/JungwooJava/cloudimg/raw/master/Logstash.png)Logstash

Logstash是开源数据收集引擎，具有实时管道功能。使用 JRuby 语言编写。其作者是世界著名的运维	工程师乔丹·西塞 (Jordan Sissel)。主要特点如下：

❖ 几乎可以访问任何数据 

❖ 可以和多种外部应用结合

❖ 支持弹性扩展

它由三个主要部分组成：

● Shipper－发送日志数据

● Broker－收集数据，缺省内置 Redis

● Indexer－数据写入



##### ✅**优势**

Logstash 主要的优点就是它的灵活性，主要因为它有很多插件，详细的文档以及直白的配置格式让它可以在多种场景下应用。我们基本上可以在网上找到很多资源，几乎可以处理任何问题。



##### **❎劣势**

Logstash 致命的问题是它的`性能`以及`资源消耗`(默认的堆大小是 1GB)。尽管它的性能在近几年已经有很大提升，与它的替代者们相比还是要慢很多的。这里有 Logstash 与 rsyslog 性能对比以及Logstash 与 filebeat 的性能对比。它在大数据量的情况下会是个问题。

另一个问题是它目前`不支持缓存`，目前的典型替代方案是将 Redis 或 Kafka 作为中心缓冲池。

​	

##### 🌐**典型应用场景**

因为 Logstash 自身的灵活性以及网络上丰富的资料，Logstash 适用于原型验证阶段使用，或者解析非常复杂的时候。在不考虑服务器资源的情况下，如果服务器的性能足够好，我们也可以为每台服务器安装 Logstash 。我们也不需要使用缓冲，因为文件自身就有缓冲的行为，而 Logstash 也会记住上次处理的位置。

如果服务器性能较差，并不推荐为每个服务器安装 Logstash ，这样就需要一个轻量的日志传输工具，将数据从服务器端经由一个或多个 Logstash 中心服务器传输到 Elasticsearch。

随着日志项目的推进，可能会因为性能或代价的问题，需要调整日志传输的方式(log shipper)。当判断 Logstash 的性能是否足够好时，重要的是对吞吐量的需求有着准确的估计，这也决定了需要为 Logstash 投入多少硬件资源。



#### Filebeat

由于 Logstash 在数据收集上并不出色，而且作为 Agent，其性能并不达标。基于此，Elastic 发布了 beats 系列轻量级采集组件。作为 Beats 家族的一员，Filebeat 是一个轻量级的日志传输工具，Filebeat 可以将日志推送到中心 Logstash。



##### ✅**优势**

Filebeat 只是一个二进制文件没有任何依赖。它占用资源极少，尽管它还十分年轻，正式因为它简单，所以几乎没有什么可以出错的地方，所以它的可靠性还是很高的。它也为我们提供了很多可以调节的点，例如：它以何种方式搜索新的文件，以及当文件有一段时间没有发生变化时，何时选择关闭文件句柄。



##### ❎**劣势**

Filebeat 的应用范围十分有限，所以在某些场景下我们会碰到问题。例如，如果使用 Logstash 作为下游管道，我们同样会遇到性能问题。正因为如此，Filebeat 的范围在扩大。开始时，它只能将日志发送到 Logstash 和 Elasticsearch，而现在它可以将日志发送给 Kafka 和 Redis，在 5.x 版本中，它还具备过滤的能力。



##### 🌐**典型应用场景**

Filebeat 在解决某些特定的问题时：日志存于文件，我们希望将日志直接传输存储到 Elasticsearch。这仅在我们只是抓去(grep)它们或者日志是存于 JSON 格式(Filebeat 可以解析 JSON)。或者如果打算使用 Elasticsearch 的 Ingest 功能对日志进行解析和丰富。

将日志发送到 Kafka/Redis。所以另外一个传输工具(例如，Logstash 或自定义的 Kafka 消费者)可以进一步丰富和转发。这里假设选择的下游传输工具能够满足我们对功能和性能的要求。

> 为什么用 Filebeat ，而不用原来的 Logstash 呢？

至此，原因比较明了，资源消耗比较大。

由于 Logstash 是跑在 JVM 上面，资源消耗比较大，后来作者用 GO 写了一个功能较少但是资源消耗也小的轻量级的 Agent 叫 Logstash-forwarder。

后来作者加入 elastic.co 公司， Logstash-forwarder 的开发工作给公司内部 GO 团队来搞，最后命名为 Filebeat。

Filebeat 需要部署在每台应用服务器上，可以通过 Salt 来推送并安装配置。



#### **Fluentd**

Fluentd 创建的初衷主要是尽可能的使用 JSON 作为日志输出，所以传输工具及其下游的传输线不需要猜测子字符串里面各个字段的类型。这样，它为几乎所有的语言都提供库，这也意味着，我们可以将它插入到我们自定义的程序中。



##### **✅优势**

和多数 Logstash 插件一样，Fluentd 插件是用 Ruby 语言开发的非常易于编写维护。所以它数量很多，几乎所有的源和目标存储都有插件(各个插件的成熟度也不太一样)。这也意味这我们可以用 Fluentd 来串联所有的东西。



##### **❎**劣势

因为在多数应用场景下，我们会通过 Fluentd 得到结构化的数据，它的灵活性并不好。但是我们仍然可以通过正则表达式，来解析非结构化的数据。尽管，性能在大多数场景下都很好，但和 syslog-ng一样，它的缓冲只存在与输出端，单线程核心以及 Ruby GIL 实现的插件意味着它大的节点下性能是受限的，不过它的资源消耗在大多数场景下是可以接受的。对于小的或者嵌入式的设备，可能需要看看 Fluent Bit，它和 Fluentd 的关系与 Filebeat 和 Logstash 之间的关系类似。



##### 💠Logstash对比Fluentd

![img](https://pic4.zhimg.com/80/v2-48e462b77b576e6577466a3ab3d1133f_720w.jpg)



##### **🌐典型应用场景**

Fluentd 在日志的数据源和目标存储各种各样时非常合适，因为它有很多插件。而且，如果大多数数据源都是自定义的应用，所以可以发现用 Fluentd 的库要比将日志库与其他传输工具结合起来要容易很多。特别是在我们的应用是多种语言编写的时候，即我们使用了多种日志库，日志的行为也不太一样。



#### **Logagent**

Logagent 是 Sematext 提供的传输工具，它用来将日志传输到 Logsene(一个基于 SaaS 平台的 Elasticsearch API)，因为 Logsene 会暴露 Elasticsearch API，所以 Logagent 可以很容易将数据推送到 Elasticsearch 。

![详解日志采集工具--Logstash、Filebeat、Fluentd、Logagent对比](https://s4.51cto.com/oss/201904/25/8918e955c0b67259b9c19bda039718a4.jpeg)



##### ✅**优势**

可以获取 /var/log 下的所有信息，解析各种格式(Elasticsearch，Solr，MongoDB，Apache HTTPD等等)，它可以掩盖敏感的数据信息，例如，个人验证信息，出生年月日，信用卡号码，等等。它还可以基于 IP 做 GeoIP 丰富地理位置信息(例如，access logs)。同样，它轻量又快速，可以将其置入任何日志块中。在新的 2.0 版本中，它以第三方 node.js 模块化方式增加了支持对输入输出的处理插件。重要的是 Logagent 有本地缓冲，所以不像 Logstash ，在数据传输目的地不可用时会丢失日志。



##### **❎劣势**

尽管 Logagent 有些比较有意思的功能(例如，接收 Heroku 或 CloudFoundry 日志)，但是它并没有 Logstash 灵活。



##### 🌐**典型应用场景**

Logagent 作为一个可以做所有事情的传输工具是值得选择的(提取、解析、缓冲和传输)。



#### **logtail**



阿里云日志服务的生产者，目前在阿里集团内部机器上运行，经过3年多时间的考验，目前为阿里公有云用户提供日志收集服务。



![详解日志采集工具--Logstash、Filebeat、Fluentd、Logagent对比](https://s3.51cto.com/oss/201904/25/a077ff1207ee753462e50078bd908c62.jpeg)

采用C++语言实现，对稳定性、资源控制、管理等下过很大的功夫，性能良好。相比于logstash、fluentd的社区支持，logtail功能较为单一，专注日志收集功能。



##### **✅优势**

logtail占用机器cpu、内存资源最少，结合阿里云日志服务的E2E体验良好。



##### **❎劣势**

logtail目前对特定日志类型解析的支持较弱，后续需要把这一块补起来。



#### **rsyslog**

绝大多数 Linux 发布版本默认的 syslog 守护进程，rsyslog 可以做的不仅仅是将日志从 syslog socket 读取并写入 /var/log/messages 。它可以提取文件、解析、缓冲(磁盘和内存)以及将它们传输到多个目的地，包括 Elasticsearch 。可以从此处找到如何处理 Apache 以及系统日志。

##### ✅**优势**

rsyslog 是经测试过的最快的传输工具。如果只是将它作为一个简单的 router/shipper 使用，几乎所有的机器都会受带宽的限制，但是它非常擅长处理解析多个规则。它基于语法的模块(mmnormalize)无论规则数目如何增加，它的处理速度始终是线性增长的。这也就意味着，如果当规则在 20-30 条时，如解析 Cisco 日志时，它的性能可以大大超过基于正则式解析的 grok ，达到 100 倍(当然，这也取决于 grok 的实现以及 liblognorm 的版本)。

它同时也是我们能找到的最轻的解析器，当然这也取决于我们配置的缓冲。

##### ❎**劣势**

rsyslog 的配置工作需要更大的代价(这里有一些例子)，这让两件事情非常困难：

文档难以搜索和阅读，特别是那些对术语比较陌生的开发者。

5.x 以上的版本格式不太一样(它扩展了 syslogd 的配置格式，同时也仍然支持旧的格式)，尽管新的格式可以兼容旧格式，但是新的特性(例如，Elasticsearch 的输出)只在新的配置下才有效，然后旧的插件(例如，Postgres 输出)只在旧格式下支持。

尽管在配置稳定的情况下，rsyslog 是可靠的(它自身也提供多种配置方式，最终都可以获得相同的结果)，它还是存在一些 bug 。

##### 🌐**典型应用场景**

rsyslog 适合那些非常轻的应用(应用，小VM，Docker容器)。如果需要在另一个传输工具(例如，Logstash)中进行处理，可以直接通过 TCP 转发 JSON ，或者连接 Kafka/Redis 缓存。

rsyslog 还适合我们对性能有着非常严格的要求时，特别是在有多个解析规则时。那么这就值得为之投入更多的时间研究它的配置。



#### ![kibana](https://gitee.com/JungwooJava/cloudimg/raw/master/kibana.png)Kibana

Kibana 是一款基于 Apache 开源协议，使用 JavaScript 语言编写，为 Elasticsearch 提供分析和可视	化的 Web 平台。它可以在 Elasticsearch 的索引中查找，交互数据，并生成各种维度的表图。



##### ✅优势

1. 能根据关键字查询日志详情
2. 能监控系统的运行状况
3. 能进行统计分析，比如接口的调用次数、执行时间、成功率等
4. 能基于日志的数据挖掘
5. 方便开发人员能随时对日志进行查阅分析（开发登录生产系统环境直接查看很麻烦）
6. 日志数据分散在多个系统，难以查找
7. 日志数据刷新过快，定位速度慢
8. 有时候一个调用会涉及多个系统，定位难度大
9. 增加日志数据的实时性



### 总结

ELK （ElasticSearch、Logstash、Kibana）三款软件之间互相配合使用，完美衔接，高效的满足了很多场合的应用，并且被很多用户所采纳，诸如路透社，脸书（Facebook），StackOverFlow 等等。

基于 ELK stack 的日志解决方案的`优势`主要体现于

- 可扩展性：采用高可扩展性的分布式系统架构设计，可以支持每日 TB 级别的新增数据。
- 使用简单：通过用户图形界面实现各种统计分析功能，简单易用，上手快
- 快速响应：从日志产生到查询可见，能达到秒级完成数据的采集、处理和搜索统计。
- 界面炫丽：Kibana 界面上，只需要点击鼠标，就可以完成搜索、聚合功能，生成炫丽的仪表板

基于以上分析，选择ELK方案作为大型系统的日志方案。



## 2.2 功能描述

`ELK`日志中心即Elasticsearch、Logstash、Kibana，组合起来搭建的线上日志系统，本文主要使用ELK来收集SpringBoot应用产生的日志。

![elk+beat](https://gitee.com/JungwooJava/cloudimg/raw/master/elk+beat.png)

- Elasticsearch 

  用于存储收集到的日志信息

- Filebeat

  轻量级的日志采集器，用于采集日志

- Logstash

  在Logstash中filters是其比较重要的功能，它能将每一天的日志信息，按照特定的格式进行解析，分成K-V的模式，这样就能很方便的对日志进行分析，存储，索引

  Logstash也可用于日志收集，但由于其内存消耗比较大，一般采用Filebeat收集日志，Logstash过滤日志

- Kibana

  通过Web端的可视化界面来查看日志。
  
  

日志中心的日志，按照场景来划分，包含四类日志：

* 调试日志

  最全日志，包含了应用中所有`DEBUG`级别以上的日志，仅在开发、测试环境中开启收集；

* 错误日志

  只包含应用中所有`ERROR`级别的日志，所有环境只都开启收集；

* 业务日志

  在应用`对应包下`所有DEBUG级别以上打印的日志，可用于查看我们自己在应用中打印的业务日志；

* 操作日志

  每个接口的`访问记录`，可以用来查看接口执行效率，获取接口访问参数。




## 2.3 系统架构

微服务平台系统整体架构如下：

![微服务架构](https://gitee.com/JungwooJava/cloudimg/raw/master/%E5%BE%AE%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84.png)



日志平台架构

<img src="https://gitee.com/JungwooJava/cloudimg/raw/master/%E6%97%A5%E5%BF%97%E5%B9%B3%E5%8F%B0%E6%9E%B6%E6%9E%84.png" alt="日志平台架构" style="zoom:200%;" />

### 方案一：**Filebeat+ELK**

一般情况下平台各服务模块的日志都存储在本地文件中， [http://Elastic.co](https://link.zhihu.com/?target=http%3A//Elastic.co)公司提供了Filebeat模块，将Filebeat部署到相应的服务器上，然后读取日志文件传给Logstash或者Elasticsearch。系统架构如下图所示：

![Filebeat+ELK](https://gitee.com/JungwooJava/cloudimg/raw/master/Filebeat+ELK.png)



![ELK & EFK部署图](https://gitee.com/JungwooJava/cloudimg/raw/master/ELK%20&%20EFK%E9%83%A8%E7%BD%B2%E5%9B%BE.png)

还有一种方案，就是直接让服务模块按照标准的协议通过网络将日志发送给Logstash，这样可以减少服务模块所在主机的磁盘IO，提高性能。另外，不需要在服务主机上安装Filebeat模块，但需要修改应用程序的日志处理方式。缺点是，需要服务器性能佳，因为在每台服务器上安装Logstash比较吃内存。



### 方案二：**EFK**

即`F`指的是`Fluentd`，它具有Logstash类似的日志收集功能，但是内存占用连Logstash的十分之一都不到，性能优越、非常轻巧。

![EFK](https://gitee.com/JungwooJava/cloudimg/raw/master/ELK%20&%20EFK%E9%83%A8%E7%BD%B2%E5%9B%BE%20(2).png)



### 方案三：**flume+kafka+ELK**

![img](https://gitee.com/JungwooJava/cloudimg/raw/master/v2-249ba5f5dac8b2a1fcad55a99e67782a_720w.jpg)



### 方案四：**Filebeat+Kafka+ELK**

如果需要应对逐渐增大的日志量级，可加入消息队列作为缓冲层

Filebeat  ---> Logstash ---> queue (Redis, Kafka, etc.) ---> Logstash ---> Elasticsearch

Filebeat 5.0支持直接输出消息给kafka了

整体架构主要分为 4 个模块，分别提供不同的功能

**Filebeat**：轻量级数据收集引擎。基于原先 Logstash-fowarder 的源码改造出来。换句话说：Filebeat就是新版的 Logstash-fowarder，也会是 ELK Stack 在 Agent 的第一选择。

**Kafka**: 数据缓冲队列。作为消息队列解耦了处理过程，同时提高了可扩展性。具有峰值处理能力，使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

**Logstash** ：数据收集处理引擎。支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储以供后续使用。

**Elasticsearch** ：分布式搜索引擎。具有高可伸缩、高可靠、易管理等特点。可以用于全文检索、结构化检索和分析，并能将这三者结合起来。Elasticsearch 基于 Lucene 开发，现在使用最广的开源搜索引擎之一，Wikipedia 、StackOverflow、Github 等都基于它来构建自己的搜索引擎。

**Kibana** ：可视化化平台。它能够搜索、展示存储在 Elasticsearch 中索引数据。使用它可以很方便的用图表、表格、地图展示和分析数据。

![亿级 ELK 日志平台构建实践](https://s4.51cto.com/images/blog/201805/16/fdfd8706744a67866b2e34368da3f2f2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)





![beat+ka+elk (2)](https://gitee.com/JungwooJava/cloudimg/raw/master/beat+ka+elk%20(2).png)



<img src="https://gitee.com/JungwooJava/cloudimg/raw/master/beat+ka+elk.png" alt="beat+ka+elk" style="zoom:125%;" />



## 2.4 技术选型

| 软件                | 描述               | 官网                                     |
| ------------------- | ------------------ | ---------------------------------------- |
| ElasticSearch-6.1.0 | 搜索引擎           | https://github.com/elastic/elasticsearch |
| SpringBoot          | SpringBoot         | https://spring.io/projects/spring-boot   |
| Kibana-6.1.0        | 日志可视化查看工具 | https://github.com/elastic/kibana        |
| Filebeat-6.1.0      | 日志收集工具       | https://github.com/elastic/filebeat      |
| Logstash-6.1.0      | 日志收集工具       | https://github.com/elastic/logstash      |





## 2.5 处理流程

- Filebeat 从各个节点中提取日志信息
- Logstash 过滤处理日志信息
- Logstash 将日志转发到 Elasticsearch 进行索引和保存
- Kibana 负责分析和可视化日志信息

![ELK & EFK部署图](C:\Users\陈欣\Documents\调研\ELK日志系统\素材\ELK & EFK部署图.png)



## 2.6 注意事项

* ES内存栈大小不超过50%系统内存，并且不超过32G

* Index.number_of_replicas

  减少Elasticsearch的index日志收集时的复制数，以减少IO，如果后期需要存储备份，则再将复制数增大，Elasticsearch会自动将数据进行重新整理备份。

* Index.refresh_interval

  数据从缓存刷新到磁盘的间隔，间隔越大,磁盘IO越少。

*  Index.translog.durability

  默认情况下，Elasticsearch 每5 秒，或每次请求操作结束前，会强制刷新translog 日志到磁盘上。后者是Elasticsearch 2.0 新加入的特性。为了保证不丢数据，每次index、bulk、delete、update 完成的时候，一定触发刷新translog 到磁盘上，才给请求返回200 OK。这个改变在提高数据安全性的同时当然也降低了一点性能。如果你不在意这点可能性，还是希望性能优先，可以在index template 里设置如下参数："index.translog.durability" : "async"

* index.merge.scheduler.max_thread_count

  merge线程的数量，线程数越少，磁盘IO越少。设置为1即可。

* 硬件方面尽可能的：

  ★ 应用SSD：提高I/O吞吐

  ★ 增大CPU，内存：提高查询性能

* 所有机器关闭防火墙，selinux

  



# 3 模块设计

## 3.1 通过日志组件来收集

这里是指通过logback、log4j等日志组件来输出文件，然后再通过文件输出到 Logstash、Kibana 等日志组件中，通过这些日志组件来进行可视化统计与分析，这里需要统一关键日志输出格式方便日后统计搜索。

优点

- 操作简单，收集方便
- 减少业务依赖
- 粒度细



缺点

- 依赖于 Logstash、Kibana
- 只能满足简单的日志操作，详细点或者个性化需求操作起来比较复杂



示例

* 引入 logstash-logback支持包

  ````xml
  <dependency>
      <groupId>net.logstash.logback</groupId>
      <artifactId>logstash-logback-encoder</artifactId>
      <version>6.6</version>
  </dependency>
  <!-- Your project must also directly depend on either logback-classic or logback-access.  For example: -->
  <dependency>
      <groupId>ch.qos.logback</groupId>
      <artifactId>logback-classic</artifactId>
      <version>1.2.3</version>
  </dependency>
  ````

* Java 版本要求

  | logstash-logback-encoder | 最小Java 版本支持 |
  | :----------------------- | ----------------- |
  | >= 6.0                   | 1.8               |
  | 5.x                      | 1.7               |
  | <= 4.x                   | 1.6               |

  

* 通过添加配置文件logback-spring.xml让logback的日志输出到logstash

```xml
<!--应用名称-->   
<property name="APP_NAME" value="act-log"/>

<!--日志文件保存路径-->
<property name="LOG_FILE_PATH" 	 value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}" />

<contextName>${APP_NAME}</contextName>

<!--每天记录日志到文件appender-->    
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">         		<!-- 按天轮转 -->
    	<fileNamePattern>${LOG_FILE_PATH}/${APP_NAME}-%d{yyyy-MM-dd}.log</fileNamePattern>
         <!--设置日志文件大小，超过就重新生成文件，默认10M-->
         <maxFileSize>${LOG_FILE_MAX_SIZE:-10MB}</maxFileSize>
          <!-- 保存 30 天的历史记录  最大大小为 3GB-->
		<maxHistory>30</maxHistory>
        <totalSizeCap>3GB</totalSizeCap>
	</rollingPolicy>
	<encoder>      
		<pattern>${FILE_LOG_PATTERN}</pattern>       
    </encoder>
</appender>
  
<!--输出到logstash的appender-->
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
	<!--可以访问的logstash日志收集端口-->
	<destination>172.31.133.12:4560</destination>
	<encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>

```

* Logstash配置

  ```php
  input {
    tcp {
      mode => "server"
      host => "0.0.0.0"
      port => 4560
      codec => json_lines
      type => "debug"
    }
    tcp {
      mode => "server"
      host => "0.0.0.0"
      port => 4561
      codec => json_lines
      type => "error"
    }
    tcp {
      mode => "server"
      host => "0.0.0.0"
      port => 4562
      codec => json_lines
      type => "business"
    }
    tcp {
      mode => "server"
      host => "0.0.0.0"
      port => 4563
      codec => json_lines
      type => "record"
    }
  }
  filter{
    if [type] == "record" {
      mutate {
        remove_field => "port"
        remove_field => "host"
        remove_field => "@version"
      }
      json {
        source => "message"
        remove_field => ["message"]
      }
    }
  }
  output {
    elasticsearch {
      hosts => ["es:19200"]
      action => "index"
      codec => json
      index => "act-log-%{type}-%{+YYYY.MM.dd}"
      template_name => "act-log"
    }
  }
  ```
  
* 配置要点

  - input：使用不同端口收集不同类型的日志，从4560~4563开启四个端口；
  - filter：对于记录类型的日志，直接将JSON格式的message转化到source中去，便于搜索查看；
  - output：按类型、时间自定义索引格式。



## 3.2 SpringBoot使用AOP来拦截controller

通过在controller层建一个切面来实现接口访问的统一日志记录，通过controller中的方法名是否包含insert、update、delete等关键字来记录业务日志，日志直接输入到ES。



优点

- 操作简单，收集方便



缺点

- 只能记录简单日志
- 不同的人命名习惯不一样，日志可能不准确



示例

> 定义日志切面，在环绕通知中获取日志需要的信息，并应用到controller层中所有的public方法中去

```java
package com.macro.mall.tiny.component;

import cn.hutool.core.util.StrUtil;
import cn.hutool.core.util.URLUtil;
import cn.hutool.json.JSONUtil;
import com.macro.mall.tiny.dto.WebLog;
import io.swagger.annotations.ApiOperation;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.lang.reflect.Parameter;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 统一日志处理切面
 */
@Aspect
@Component
@Order(1)
public class WebLogAspect {
    private static final Logger LOGGER = LoggerFactory.getLogger(WebLogAspect.class);

    @Pointcut("execution(public * com.act.idc.controller.*.*(..))")
    public void webLog() {
    }

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable {
    }

    @AfterReturning(value = "webLog()", returning = "ret")
    public void doAfterReturning(Object ret) throws Throwable {
    }

    @Around("webLog()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        //获取当前请求对象
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        //记录请求信息
        WebLog webLog = new WebLog();
        Object result = joinPoint.proceed();
        Signature signature = joinPoint.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        Method method = methodSignature.getMethod();
        if (method.isAnnotationPresent(ApiOperation.class)) {
            ApiOperation apiOperation = method.getAnnotation(ApiOperation.class);
            webLog.setDescription(apiOperation.value());
        }
        long endTime = System.currentTimeMillis();
        String urlStr = request.getRequestURL().toString();
        webLog.setBasePath(StrUtil.removeSuffix(urlStr, URLUtil.url(urlStr).getPath()));
        webLog.setIp(request.getRemoteUser());
        webLog.setMethod(request.getMethod());
        webLog.setParameter(getParameter(method, joinPoint.getArgs()));
        webLog.setResult(result);
        webLog.setSpendTime((int) (endTime - startTime));
        webLog.setStartTime(startTime);
        webLog.setUri(request.getRequestURI());
        webLog.setUrl(request.getRequestURL().toString());
        LOGGER.info("{}", JSONUtil.parse(webLog));
        return result;
    }

    /**
     * 根据方法和传入的参数获取请求参数
     */
    private Object getParameter(Method method, Object[] args) {
        List<Object> argList = new ArrayList<>();
        Parameter[] parameters = method.getParameters();
        for (int i = 0; i < parameters.length; i++) {
            //将RequestBody注解修饰的参数作为请求参数
            RequestBody requestBody = parameters[i].getAnnotation(RequestBody.class);
            if (requestBody != null) {
                argList.add(args[i]);
            }
            //将RequestParam注解修饰的参数作为请求参数
            RequestParam requestParam = parameters[i].getAnnotation(RequestParam.class);
            if (requestParam != null) {
                Map<String, Object> map = new HashMap<>();
                String key = parameters[i].getName();
                if (!StringUtils.isEmpty(requestParam.value())) {
                    key = requestParam.value();
                }
                map.put(key, args[i]);
                argList.add(map);
            }
        }
        if (argList.size() == 0) {
            return null;
        } else if (argList.size() == 1) {
            return argList.get(0);
        } else {
            return argList;
        }
    }
}

```



## 3.3 使用注解来，进行稍微精准的业务日志记录

优点

这个方案粒度可大可小，代码侵入性也比较小，可操作性比较强，如果需要获取参数信息或者返回值信息，可以通过注解配置获取到，可以集合fastjson中的jpath来获取参数值



缺点

具有少量的代码侵入性（注解）



实际开发经验

实际开发过程中，往往此操作日志注解中的文字描述与swagger Api中的有所重复，可直接采取方式二统一拦截，获取Swagger Api中的注解描述即可。



示例

 ```java
/**
 *
 * <p>Title: OperateLogInterceptor
 * <p>Description: 操作记录拦截器
 */
@Component
@Aspect
@Log4j2
public class OperateLogInterceptor {
    /** 字段缓存 */
    private static Cache<Class<?>, Field[]> FIELDS_CACHE= CacheBuilder.newBuilder().build();
    /**
     * 注解内容缓存
     */
    private static Cache<Class<?>, Field> ANNOTATION_FIELDS_CACHE= CacheBuilder.newBuilder().build();

    /**
     * 定义织入切面的范围，目前只会织入controller的请求部分
     */
    @Pointcut("execution(* com.act..*.controller..*.*(..)) && @annotation(com.act.util.operatelog.OperateLog)")
    private void controllerAspect() {}// 定义一个切入点

    /**
     * 在请求完毕后，保存日志表
     * @param pjd 切入点
     * @throws Throwable 操作日志异常
     */
    @Around("controllerAspect()")
    public Object doAroundMethod(ProceedingJoinPoint pjd) throws Throwable {
        String opDesc = getOpDesc(pjd);
        // 其中查询系列的日志记录根据开关控制是否入库，默认不需要入库
        if (OperateLogConstant.PAGING.equals(opDesc) ||
                OperateLogConstant.INIT.equals(opDesc) ||
                OperateLogConstant.INIT_EDIT.equals(opDesc) ||
                OperateLogConstant.VIEW.equals(opDesc) ){
            String flag = "0";
			/*if (flag.equals(EhcacheUtil.getConfigByConfigId(ConfigConstant.OPERATE_LOG_VIEW_SWITCH))){
				return pjd.proceed();
			}*/
        }
        HttpServletRequest request = getHttpServletRequest();
        String userId = BaseContextHandler.getUserID()!=null?BaseContextHandler.getUserID():"auto";
        String ip = IpUtil.getClientIP(request);
        // 获取请求参数
        boolean argInsert = false;
        Object[] params = pjd.getArgs();
        StringBuilder paramStr = new StringBuilder();
        for (Object param : params) {
            if (param instanceof HttpServletRequest || param instanceof HttpServletResponse || param instanceof HttpSession){
                continue;
            }
            try {
                paramStr.append(JSONUtil.toJsonStr(param)).append(" ");
                if(paramStr.length()>=1024){
                    break;
                }
            } catch (Exception e) {
                log.error("获取参数异常",e);
            }
        }
        Object returnVal = null;
        TOperateLog operateLog = null;
        Long startTime=System.currentTimeMillis();
        try {
            operateLog = getBeforeLog(pjd,userId,ip,paramStr.toString(),argInsert);
            returnVal = pjd.proceed();
            operateLog = getAfterLog(pjd, operateLog, returnVal);
        } catch (Throwable ex) {
            operateLog = getExceptionLog(pjd, operateLog, ex);
            log.error(ex.getMessage());
            throw ex;
        } finally {
            if (operateLog != null) {
                // 执行时间
                operateLog.setOpExecTimeMillis((int) (System.currentTimeMillis() - startTime));
                operateLog.insert();
            }
        }
        return returnVal;
    }

    /**
     * <p>title: getBeforeLog
     * <p>Description:操作方法前获取日志信息，主要是注解类和IP等
     * @param pjd 切面操作类
     * @return 日志实体对象
     */
    private TOperateLog getBeforeLog(JoinPoint pjd, String userId, String ip, String paramStr, boolean 		 argInsert) {
        TOperateLog operateLog = new TOperateLog();
        // 操作人
        operateLog.setOpOperater(userId==null?"系统":userId);
        // 操作时间
        operateLog.setOpTime(LocalDateTime.now());
        //默认写的值，对于操作类型，打算废除
        operateLog.setOpType("01");
        // 操作描述
        operateLog.setOpDesc(getOpDesc(pjd));
        // 操作结果
        operateLog.setOpResult(true);
        // 操作IP
        operateLog.setOpIp(ip);
        // 操作对象
        operateLog.setOpObject(getOpObject(pjd)+(argInsert?"-新增":""));
        // 操作参数
        operateLog.setOpParams(paramStr.length()>=1024? paramStr.substring(0,1024): paramStr);
        return operateLog;
    }

    /**
     * 设置执行完成后对应的日志内容
     *
     * @param pjp
     * @param operateLog
     * @param returnVal
     * @return
     */
    public TOperateLog getAfterLog(ProceedingJoinPoint pjp, TOperateLog operateLog, Object returnVal) {
        if (returnVal == null || !(returnVal instanceof RestResponse)) {
            operateLog.setOpResponse("");
            operateLog.setOpResult(true);
        } else {
            RestResponse rs = (RestResponse) returnVal;
            if(rs.getCode()!=200){
                operateLog.setOpResult(false);
            }
            operateLog.setOpResponse(rs.getMessage());
        }
        operateLog.setOpResponse(getRightLengthString(operateLog.getOpResponse(),1024));
        return operateLog;
    }


    /**
     * 设置异常发生时对应的日志内容
     *
     * @param pjp
     * @param operateLog
     * @param ex
     * @return
     */
    public TOperateLog getExceptionLog(ProceedingJoinPoint pjp, TOperateLog operateLog, Throwable ex) {
        operateLog.setOpResult(false);
        operateLog.setOpResponse(getRightLengthString(ex.getMessage(), 500));
        return operateLog;
    }

    /**
     * 根据给定的长度截取支付查
     *
     * @param str
     * @param length
     * @return
     */
    private String getRightLengthString(String str, int length) {
        return StringUtils.hasLength(str) && str.length() > length ? str.substring(0, length) : str;
    }

    /**
     * <p>Title: getHttpServletRequest
     * <p>Description: 获取request
     * @author FMJ
     * @date 2018/3/19 8:44
     * @return request对象
     */
    private HttpServletRequest getHttpServletRequest() {
        RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        return sra.getRequest();
    }

    /**
     * <p>title: getAnnotation
     * <p>Description:获取参数注解
     * @param joinPoint 切面
     * @return 注解
     */
    private OperateLog getAnnotation(JoinPoint joinPoint){
        try {
            String targetName = joinPoint.getTarget().getClass().getName();
            String methodName = joinPoint.getSignature().getName();
            Object[] arguments = joinPoint.getArgs();
            Class targetClass = Class.forName(targetName);
            Method[] methods = targetClass.getMethods();

            for (Method method : methods) {
                if (method.getName().equals(methodName)) {
                    Class[] clazzs = method.getParameterTypes();
                    if (clazzs.length == arguments.length) {
                        return method.getAnnotation(OperateLog.class);
                    }
                }
            }
        } catch (ClassNotFoundException e) {
            log.error("获取注解异常：",e);
        }
        return null;
    }

    /**
     * <p>Title: getOpDesc
     * <p>Description: 获取操作描述
     * @param joinPoint 切入点
     * @return 参数
     * @throws Exception 异常
     */
    private String getOpDesc(JoinPoint joinPoint) {
        String opDesc = "";
        OperateLog operateLog=getAnnotation(joinPoint);
        if(operateLog!=null){
            opDesc=operateLog.opDesc();
        }
        return opDesc;
    }

    /**
     * <p>Title: getOpObject
     * <p>Description: 操作对象
     * @param joinPoint 切入点
     * @return 操作对象
     */
    private String getOpObject(JoinPoint joinPoint) {
        String opObject = "";
        OperateLog operateLog=getAnnotation(joinPoint);
        if(operateLog!=null){
            opObject=operateLog.opObject();
        }
        return opObject;
    }

    private Field[] getFileds(Class<?> beanClass){
        Field[] allFields = FIELDS_CACHE.getIfPresent(beanClass);
        if (null != allFields) {
            return allFields;
        }
        Field[] fields = beanClass.getSuperclass().getDeclaredFields();
        FIELDS_CACHE.put(beanClass,fields);
        return fields;
    }

    private Field getAnnotationField(Class<?> beanClass,Class annotationCls){
        Field ifPresent = ANNOTATION_FIELDS_CACHE.getIfPresent(beanClass);
        if(ifPresent!=null){
            return ifPresent;
        }
        Field[] fields = getFileds(beanClass);
        for(Field field:fields){
            field.setAccessible(true);
            Annotation fieldAnnotation = field.getAnnotation(annotationCls);
            if(fieldAnnotation!=null){
                ANNOTATION_FIELDS_CACHE.put(beanClass,field);
                return field;
            }
        }
        return null;
    }
}
 ```



```java
@OperateLog(opObject = OperateLogConstant.SAVE_OR_UPDATE, opDesc = "保存修改信息")
```



## 3.4 针对复杂场景或者审计需求手动记录，侵入性强

如果有些业务公用了方法，需要更小的粒度，或者需要记录业务数据变更记录，这时就只能选择侵入式较强的方式来记录日志了，直接在业务代码中记录日志

```java
logClient.logObject(LogObjectType.XXX,id,UserContext.getUserName(),"编辑xxx","xxx变更为xxxxx");
```



## 3.5 记录Sql日志

则可以使用Mybatis拦截器来进行，如果没有使用Mybatis，则可以做一个通用的Preparestatement以及statement代理。

当然现在已经有很多监控组件可以满足这个需求，比如说jeager、javamelody、druid等

一般情况下我们会采用多种方式来记录业务日志，这个都是根据具体需求来进行评估用哪几种方式可以更好的达到产品需求

## 3.6 非规范的日志采集展示流程

需要给出非规范日志的实例，定制化开发配置，转化为标准日志。



# 4 日志数据输入源

## 4.1 按照场景划分

* 调试日志

  最全日志，包含了应用中所有`DEBUG`级别以上的日志，仅在开发、测试环境中开启收集；

* 错误日志

  只包含应用中所有`ERROR`级别的日志，所有环境只都开启收集；

* 业务日志

  在应用`对应包下`所有DEBUG级别以上打印的日志，可用于查看我们自己在应用中打印的业务日志；

* 操作日志

  每个接口的`访问记录`，可以用来查看接口执行效率，获取接口访问参数。

## 4.2 按照业务场景划分

1. 框架/容器日志
   1. Nginx
   2.  Nacos
   3. Tomcat日志

2. 微服务业务系统：
   1. 操作日志：包含的字段如下——（traceId、host、user、time、message、level、params、response、code、menuname、apiname、url、exectime、systemcode）
   2. 系统日志：系统运行日志
   3. 现有业务系统日志：sdk包引入+ 输出规范
   4. 后台：cu、eu













# 5 日志输出规范

## 5.1 ES日志数据规范

| 字段    | 类型      | 长度 | 是否为null | 描述     |
| ------- | --------- | ---- | ---------- | -------- |
| time    | timestamp | 11   | not null   | 生成时间 |
| source  | keyword   | 255  | not null   | 日志来源 |
| message | keyword   |      | not null   | 日志信息 |
|         |           |      |            |          |
|         |           |      |            |          |
|         |           |      |            |          |
|         |           |      |            |          |
|         |           |      |            |          |
|         |           |      |            |          |





## 5.2 操作日志数据结构规范



![image-20210203155452354](https://gitee.com/JungwooJava/cloudimg/raw/master/image-20210203155452354.png)

CREATE TABLE `t_operate_log` (
  `op_id` int(10) NOT NULL AUTO_INCREMENT,
  `op_operater` varchar(32) NOT NULL COMMENT '操作人',
  `op_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '操作时间',
  `op_type` char(2) NOT NULL COMMENT '操作类型',
  `op_desc` text NOT NULL COMMENT '操作描述',
  `op_result` bit(1) NOT NULL COMMENT '操作结果0失败1成功',
  `op_ip` varchar(39) NOT NULL COMMENT '操作IP',
  `op_object` varchar(200) NOT NULL COMMENT '操作对象',
  `op_params` text NOT NULL COMMENT '操作参数',
  `op_response` text COMMENT '返回结果',
  `op_exec_time_millis` int(10) NOT NULL COMMENT '执行时间',
  PRIMARY KEY (`op_id`) USING BTREE
) ENGINE=MyISAM AUTO_INCREMENT=139228 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='系统操作日志表 ';



# 6 最终成果

1. 邮件告警

   error 级别日志邮件提醒

2. 日志查询

   操作日志查询、系统日志查询

3. 统计分析

   操作日志统计，比如登录用户数、菜单功能热度统计

# 7 系统出错处理设计

## 7.1 补救措施

故障出现后可能采取的变通措施，包括：

l     后备技术即日志仍保留向文件传输一份，设置清理时长，另一份即采集到日志中心；

l     服务高可用即日志中心采用集群的部署方式，双机热备，当故障出现时实现自动转移。



# 8 问题遗留

```汉语
1、elasticsearch内存占用高，以及索引的管理与维护，还在优化和考虑中。
2、需要开发更加人性化且更易扩展和维护的运控平台供使用方查询日志。
3、日志如果需要保留较长的时间，需要增加大数据组件，将数据存储至HDFS，将存储层的HDFS移到日志中心，需要支持日志同时写入ES集群和HDFS集群。
4、日志数据挖掘的应用。
```

