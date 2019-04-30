## SolrCloud参数引用 
<div class="content-intro view-box ">
## SolrCloud<span style="font-family: inherit; font-size: 16px; font-weight: 600;">群集参数</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/parameter-reference.html#cluster-parameters"/>

**numShards**
默认为<code>1</code>。散列文档的碎片数量。每个碎片必须有一个leader，每个leader可以有N个副本。  

## SolrCloud实例参数<a href="http://lucene.apache.org/solr/guide/7_0/parameter-reference.html#solrcloud-instance-parameters"/>
这些被设置在solr.xml中，但默认情况下，host和hostContext参数设置为也与系统属性一起使用。  
**host**
默认为找到的第一个本地主机地址。如果自动找到错误的主机地址，则可以使用此参数覆盖主机地址。  
**hostPort**
默认为通过<code>bin/solr -p &lt;port&gt;</code>指定的端口，如果未指定，则为<code>8983</code>。Solr运行的端口。这个值只有在指定<code>-DzkRun</code>（见下面）的情况下使用，才能计算嵌入式ZooKeeper运行的默认端口。在Solr附带的<code>solr.xml</code>中，<code>hostPort</code>系统属性没有被引用，所以被忽略。如果要在非默认端口上运行Solr，请使用<code>bin/solr -p &lt;port&gt;</code>而不是指定<code>-DhostPort</code>。  
**hostContext**
默认为<code>solr</code>。Solr Web应用程序的上下文路径。  

## SolrCloud实例ZooKeeper参数<a href="http://lucene.apache.org/solr/guide/7_0/parameter-reference.html#solrcloud-instance-zookeeper-parameters"/>

**zkRun**
默认为<code>localhost:&lt;hostPort+1000&gt;</code>。导致Solr运行ZooKeeper的嵌入式版本。设置为该节点上ZooKeeper的地址；这使我们能够在<code>zkHost</code>连接字符串中的地址列表中知道您是谁。使用<code>-DzkRun</code>（无值）来获取默认值。  
**zkHost**
ZooKeeper的主机地址。通常这是ZooKeeper集合中每个节点的逗号分隔列表。  
**zkClientTimeout**
默认为15000。客户端在会话到期之前不允许与ZooKeeper交谈的时间。  

zkRun和zkHost使用系统属性进行设置。zkClientTimeout是solr.xml默认设置的，但也可以使用系统属性来设置。  
## SolrCloud核心参数<a href="http://lucene.apache.org/solr/guide/7_0/parameter-reference.html#solrcloud-core-parameters"/>

**shard**
默认为基于numShards的自动分配。指定该核心作为副本的哪个分片。<code>shard</code>可以在[<code>core.properties</code>](http://lucene.apache.org/solr/guide/7_0/defining-core-properties.html#defining-core-properties)每个核心中指定。  

在 solr 的格式中讨论了其他与cloud相关的参数。  
