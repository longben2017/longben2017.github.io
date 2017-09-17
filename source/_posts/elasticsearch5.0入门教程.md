---
title: elasticsearch5.0入门教程
date: 2017-07-09 11:12:30
categories: 
- 技术分享
tags: 
- elasticsearch
- java
- retrofit2

---

本文主要介绍elasticsearch的一些基础概念以及一些简单的用法。

![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/elasticsearchLogo.png)
<!-- more -->

## 一、Elasticsearch是什么？
### Elasticsearch简介
&emsp;&emsp;<font color=red>es是一个实时分布式搜索和分析引擎。</font>基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。
&emsp;&emsp;但Lucene非常复杂，需要深入了解检索的相关知识来理解它是如何工作的。
&emsp;&emsp;而es就是使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是<font color=red>通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。</font>

&emsp;&emsp;实际案例:

 - 维基百科使用es提供全文搜索并高亮关键字，以及输入实时搜索(search-as-you-type)和搜索纠错(did-you-mean)等搜索建议功能。
 - 英国卫报使用es将访问者日志与社会网络数据相结合，向公众提供有关公众对新文章的回应的实时反馈。
 - StackOverflow将全文搜索与地理位置查询相结合，以及more-like-this功能这种来查找相关的问题和答案。
 - Github使用Elasticsearch检索1300亿行的代码。

### Elasticsearch特性
&emsp;&emsp;a)分布式的实时文件存储，每个字段都被索引并可被搜索；
&emsp;&emsp;b)分布式的实时分析搜索引擎；
&emsp;&emsp;c)可以拓展到上百台服务器，处理PB级结构化或非结构化数据。

### Elasticsearch基本概念
&emsp;&emsp;**集群(cluster)**
一个集群就是由一个或多个节点组织在一起，它们共同持有你整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识。
&emsp;&emsp;**节点(node)**
一个节点是你集群中的一个服务器，作为集群的一部分，它存储你的数据，参与集群的索引和搜索功能。一个节点也是由一个名字来标识的。一个节点可以通过配置集群名称的方式来加入一个指定的集群。
&emsp;&emsp;**索引(index)**
索引是一类文档的集合，所有的操作比如索引(索引数据)、搜索、分析都是基于索引完成的。在一个集群中，可以定义任意数量的索引。
&emsp;&emsp;**类型(type)**
在一个索引中，你可以定义一种或多种类型。类型可以理解成一个索引的逻辑分区，用于标识不同的文档字段信息的集合。
&emsp;&emsp;**文档(document)**
文档是存储数据信息的基本单元，使用json来表示。
&emsp;&emsp;**分片和复制(shards & replicas)**
一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。

分片的好处：

 - 如果一个索引数据量很大，会造成硬件硬盘和搜索速度的瓶颈。如果分成多个分片，分片可以分摊压力;
 - 分片允许用户进行水平的扩展和拆分;
 - 分片允许分布式的操作，可以提高搜索以及其他操作的效率;
 
拷贝一份分片就完成了分片的备份，那么备份有什么好处呢？

 - 当一个分片失败或者下线时，备份的分片可以代替工作，提高了高可用性。
 - 备份的分片也可以执行搜索操作，分摊了搜索的压力。
 

> 最后附上elasticsearch与传统关系型数据库的对比图

![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/elasticsearch%26db.png)

## 二、Elasticsearch怎么用？
### 安装
&emsp;&emsp;确保java版本至少为1.8
&emsp;&emsp;1)下载
```bash
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.2.tar.gz
```
&emsp;&emsp;2)解压
```bash
tar -xvf elasticsearch-5.0.2.tar.gz
```
&emsp;&emsp;3)使用root用户修改系统配置
&emsp;&emsp;&emsp;&emsp;a)修改/etc/sysctl.conf文件并执行sysctl -p使其生效
```bash
echo "vm.max_map_count= 262144" >> /etc/sysctl.conf
sysctl -p
```
&emsp;&emsp;&emsp;&emsp;b)修改/etc/security/limits.conf文件，添加以下内容并重新登录用户使其生效
```bash
*               soft    nproc           65536
*               hard    nproc           65536
*               soft    nofile          65536
*               hard    nofile          65536
```
&emsp;&emsp;4)进入bin目录并运行elasticsearch
```bash
cd elasticsearch-5.0.2/bin
./elasticsearch
```
&emsp;&emsp;若启动成功，你应该能在控制台看到类似以下的信息：
```bash 
[2016-09-16T14:17:51,251][INFO ][o.e.n.Node               ] [] initializing ...
[2016-09-16T14:17:51,329][INFO ][o.e.e.NodeEnvironment    ] [6-bjhwl] using [1] data paths, mounts [[/ (/dev/sda1)]], net usable_space [317.7gb], net total_space [453.6gb], spins? [no], types [ext4]
[2016-09-16T14:17:51,330][INFO ][o.e.e.NodeEnvironment    ] [6-bjhwl] heap size [1.9gb], compressed ordinary object pointers [true]
[2016-09-16T14:17:51,333][INFO ][o.e.n.Node               ] [6-bjhwl] node name [6-bjhwl] derived from node ID; set [node.name] to override
[2016-09-16T14:17:51,334][INFO ][o.e.n.Node               ] [6-bjhwl] version[5.0.2], pid[21261], build[f5daa16/2016-09-16T09:12:24.346Z], OS[Linux/4.4.0-36-generic/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_60/25.60-b23]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [aggs-matrix-stats]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [ingest-common]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-expression]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-groovy]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-mustache]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-painless]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [percolator]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [reindex]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [transport-netty3]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [transport-netty4]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded plugin [mapper-murmur3]
[2016-09-16T14:17:53,521][INFO ][o.e.n.Node               ] [6-bjhwl] initialized
[2016-09-16T14:17:53,521][INFO ][o.e.n.Node               ] [6-bjhwl] starting ...
[2016-09-16T14:17:53,671][INFO ][o.e.t.TransportService   ] [6-bjhwl] publish_address {192.168.8.112:9300}, bound_addresses {{192.168.8.112:9300}
[2016-09-16T14:17:53,676][WARN ][o.e.b.BootstrapCheck     ] [6-bjhwl] max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
[2016-09-16T14:17:56,731][INFO ][o.e.h.HttpServer         ] [6-bjhwl] publish_address {192.168.8.112:9200}, bound_addresses {[::1]:9200}, {192.168.8.112:9200}
[2016-09-16T14:17:56,732][INFO ][o.e.g.GatewayService     ] [6-bjhwl] recovered [0] indices into cluster_state
[2016-09-16T14:17:56,748][INFO ][o.e.n.Node               ] [6-bjhwl] started
```

&emsp;&emsp;此时，打开浏览器，并访问127.0.0.1:9200,可看到如下图所示：
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/9200page.png)

&emsp;&emsp;默认情况下，只有本机地址能访问到elasticsearch，若需要其他机器也能访问此elasticsearch，需要修改一些elasticsearch的配置文件。
&emsp;&emsp;&emsp;&emsp;a)打开elasticsearch配置文件
```bash
vi elasticsearch-5.0.2/config/elasticsearch.yml
```
&emsp;&emsp;&emsp;&emsp;b)把network.host修改为广播地址0.0.0.0(注意冒号后面有空格)
```bash 
network.host: 0.0.0.0
```

### 探索集群
&emsp;&emsp;1)查看集群健康
&emsp;&emsp;以基本的健康检查作为开始，我们可以利用它来查看我们集群的状态。此过程中，我们使用curl。要检查集群健康，我们将使用_cat API。
```bash
curl 'localhost:9200/_cluster/health?pretty'
```
&emsp;&emsp;相应的响应是:
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/curlHealth.png)
&emsp;&emsp;可以看到，我们集群的名字是“elasticsearch”，正常运行，并且状态是黄色。
&emsp;&emsp;当我们询问集群状态的时候，我们要么得到绿色、黄色或红色。绿色代表一切正常（集群功能齐全），黄色意味着所有的数据都是可用的，但是某些复制没有被分配（集群功能齐全），红色则代表因为某些原因，某些数据不可用。注意，即使是集群状态是红色的，集群仍然是部分可用的（它仍然会利用可用的分片来响应搜索请求），但是可能你需要尽快修复它，因为你有丢失的数据。

&emsp;&emsp;2)列出所有的索引
```bash
curl 'localhost:9200/_cat/indices?v'
```
&emsp;&emsp;相应的响应是:
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/curlIndices.png)
&emsp;&emsp;这个结果意味着，在我们的集群中，我们目前没有任何索引。

&emsp;&emsp;3)创建一个索引
现在让我们创建一个叫做“customer”的索引，然后再列出所有的索引：
```bash
curl -XPUT 'localhost:9200/customer?pretty'
curl 'localhost:9200/_cat/indices?v'
```
第一个命令使用PUT创建了一个叫做“customer”的索引。我们简单地将pretty附加到调用的尾部，使其以美观的形式打印出JSON响应（如果有的话）。
&emsp;&emsp;响应如下:
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/createIndex.png)
第二个命令的结果告知我们，我们现在有一个叫做customer的索引，并且它有5个主分片和1份复制（都是默认值），其中包含0个文档。

&emsp;&emsp;4)索引并查询一个文档
现在让我们放一些东西到customer索引中。首先要知道的是，为了索引一个文档，我们必须告诉Elasticsearch这个文档要到这个索引的哪个类型（type）下。
&emsp;&emsp;让我们将一个简单的客户文档索引到customer索引、“external”类型中，这个文档的ID是1，操作如下:
```bash
curl -XPUT 'localhost:9200/customer/external/1?pretty' -d '
        {
          "name": "John Doe"
        }'
```
&emsp;&emsp;响应如下:
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/createDoc.png)
从上面的响应中，我们可以看到，一个新的客户文档在customer索引和external类型中被成功创建。文档也有一个内部id 1， 这个id是我们在索引的时候指定的。

&emsp;&emsp;有一个关键点需要注意，Elasticsearch在你想将文档索引到某个索引的时候，并<font color=red>不强制要求这个索引被显式地创建</font>。在前面这个例子中，如果customer索引不存在，Elasticsearch将会自动地创建这个索引。

&emsp;&emsp;5)更新文档

&emsp;&emsp;6)删除文档

&emsp;&emsp;7)批处理
&emsp;&emsp;除了能够对单个的文档进行索引、更新和删除之外，Elasticsearch也提供了以上操作的批量处理功能，这是通过使用_bulk API实现的。这个功能之所以重要，在于它提供了非常高效的机制来尽可能快的完成多个操作，与此同时使用尽可能少的网络往返。

作为一个快速的例子，以下调用在一次bulk操作中索引了两个文档（ID 1 - John Doe and ID 2 - Jane Doe）:
```bash
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'
```
以下例子在一个bulk操作中，首先更新第一个文档（ID为1），然后删除第二个文档（ID为2）：
```bash
curl -XPOST 'localhost:9200/customer/external/_bulk?pretty' -d '
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'
```
注意上面的delete动作，由于删除动作只需要被删除文档的ID，所以并没有对应的源文档。

### 探索数据
载入样本数据([点击下载样本数据][1])

 [1]: https://github.com/bly2k/files/blob/master/accounts.zip?raw=tr
```bash
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl 'localhost:9200/_cat/indices?v'
```

&emsp;&emsp;响应:
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/curlIndeces2.png)
这意味着我们成功批量索引了1000个文档到银行索引中（account类型）。

&emsp;&emsp;现在，让我们以一些简单的搜索来开始。有两种基本的方式来运行搜索：一种是在REST请求的URI中发送搜索参数，另一种是将搜索参数发送到REST请求体中。
&emsp;&emsp;一般情况下，执行一些比较简单的命令时会使用第一种方法，而当执行一些比较复杂的命令时(如过滤，排序，聚合等)使用第二种方法。

&emsp;&emsp;最简单的查询
```bash
curl 'localhost:9200/_search?pretty'
```
 ![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/curlSearch.png)
 对于这个响应，我们看到了以下的部分：
- took —— Elasticsearch执行这个搜索的耗时，以毫秒为单位
- timed_out —— 指明这个搜索是否超时
- _shards —— 指出多少个分片被搜索了，同时也指出了成功/失败的被搜索的shards的数量
- hits —— 搜索结果
- hits.total ——能够匹配我们查询标准的文档的总数目
- hits.hits —— 真正的搜索结果数据（默认只显示前10个文档）
- _score和max_score —— 得分

具体的其他用法就暂不在此展开了，有兴趣的可以另行了解。

## 三、项目中如何使用elasticsearch
在实际项目中，一般不会直接在浏览器上敲一个命令去操作elasticsearch从而解决一些业务上的实际问题。所以，elasticsearch官方提供了多种类型的客户端(如Java,Python,.NET,Groovy等)，下面就介绍几种与java结合一起使用的方法。
### Java API
elasticsearch也提供了java api去调用，这种方式是通过调用elasticsearch的client客户端从而去索引或者查询数据的。
&emsp;&emsp;1)先获取elasticsearch的client;
```java
public void before() throws Exception{
    client = new PreBuiltTransportClient(Settings.EMPTY)
		.addTransportAddress(new InetSocketTransportAddress(
        InetAddress.getByName("192.168.137.128"),9300));
}
```
&emsp;&emsp;2)获取到client后就可以根据elasticsearch给出的各种接口对其数据进行操作了;
```java
public void testGetDocs(){
	GetResponse response = client.prepareGet("bank","account","1").execute().actionGet();
	System.out.println(response.getSourceAsString());
}
```
&emsp;&emsp;3)最后，可获取相应的返回数据。
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/javaApiSearch.png)

### Retrofit2
Retrofit是square开源的网络请求库，底层是使用OKHttp封装的，网络请求速度很快。

&emsp;&emsp;下面直接应用到实际的代码中
&emsp;&emsp;1)Retrofit通过接口来管理HTTP API，那么首先我们先定义一个API的接口：
```java
public interface SearchApi {
	@HEAD("/{index}")
	Call<Void> getIndexExist(@Path("index")String indexName);
	
	@GET("/{index}/_search")
	Call<AccountInfo> getDocs(@Path("index")String indexName);
}
```
&emsp;&emsp;2)然后通过Retrofit.Builder获取到Retrofit实例，并通过create(clazz)方法获取到我们刚才创建的SearchApi接口实例：
```java
public void before(){
	Retrofit retrofit = new Retrofit.Builder()
	.baseUrl("http://192.168.137.128:9200/")
	.addConverterFactory(JacksonConverterFactory.create())
	.build();
	searchApi = retrofit.create(SearchApi.class);
	gson = new GsonBuilder().setPrettyPrinting().create();
}
```
&emsp;&emsp;3)有了SearchApi实例之后，我们就可以在需要网络请求的地方调用了：
```java
public void testGetDocs(){
	try {
		Response<AccountInfo> response = searchApi.getDocs("bank").execute();
		AccountInfo accountInfo = (AccountInfo)response.body();
		System.out.println(gson.toJson(accountInfo));
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
&emsp;&emsp;4)最后，就可以得到请求返回的信息了：
![](https://raw.githubusercontent.com/longben2017/image/master/elasticsearchSharingDoc/retrofitSearch.png)

这次的介绍就先到这里，下次会继续介绍关于elasticsearch的应用场景以及一些实战演练。
