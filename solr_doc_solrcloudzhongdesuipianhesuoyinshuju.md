## SolrCloud中的碎片和索引数据 
<div class="content-intro view-box ">当您的集合对于一个节点来说太大时，您可以通过创建多个分片将其分解并分段存储。  
  
碎片是集合的逻辑分区，包含集合中的文档的子集，使集合中的每个文档都包含在一个碎片中。哪个碎片包含集合中的每个文档取决于该集合的整体“碎片”策略。  
例如，您可能有一个集合，其中每个文档的“国家/地区”字段确定它属于哪个分片，因此来自同一个国家/地区的文档位于同一位置。不同的集合可以简单地在每个文档的uniqueKey上使用“哈希”来确定其碎片。  
在SolrCloud之前，Solr支持分布式搜索，它允许在多个分片上执行一个查询，因此查询是针对整个Solr索引执行的，搜索结果中不会有任何文档被遗漏。所以在一个碎片上分割一个索引并不完全是一个SolrCloud的概念。然而，分布式方法存在几个问题，需要使用SolrCloud进行改进：  
1 <li>将索引分割成碎片是有点手动的。</li>2 <li>没有支持分布式索引，这意味着您需要明确地发送文件到一个特定的碎片；Solr无法自行找出发送文件的碎片。</li>3 <li>没有负载平衡或故障转移，所以如果您有大量的查询，您需要找出发送它们的位置，如果一个碎片死了，它就消失了。</li>
SolrCloud解决了这些限制。支持自动分配索引进程和查询，并且ZooKeeper提供故障转移和负载平衡。另外，每个碎片都可以有多个复制副本，以增强可靠性。  

## Leader和副本

在SolrCloud中没有主人或奴隶。相反，每个碎片至少包含一个物理副本，其中一个是Leader。Leader自动当选，最初是先到先得，然后根据http://zookeeper.apache.org/doc/trunk/recipes.html#sc_leaderElection上描述的ZooKeeper流程。  
  
如果一个Leader失败了，其他副本中的一个会自动选为新的Leader。  
当文档被发送到Solr节点进行索引时，系统首先确定该文档属于哪个碎片，然后确定哪个节点当前正在主管该碎片的Leader。然后将文档转发给当前Leader进行索引，并且Leader将更新转发给所有其他副本。  

### 副本的类型

默认情况下，如果leader失败，所有副本都有资格成为leader。但是，这是有要求的：如果所有副本都可以在任何时候成为leader，那么每个副本都必须始终与其leader保持同步。添加到leader的新文档必须路由到副本，每个副本必须提交。如果副本发生故障或暂时不可用，然后重新加入群集，则恢复可能会很慢，因为它错过了大量的更新。  
  
这些问题对于大多数用户来说不是问题。但是，如果副本的表现更像前一种模式，则某些用例会更好一些，或者不是实时同步，或者根本就没有资格成为leader。  
Solr通过允许您在创建新集合或添加副本时设置副本类型来完成此操作。可用的类型是：  

    - NRT：这是默认的设置。NRT副本（NRT = NearRealTime）维护事务日志，并将新文档写入本地索引。任何这种类型的副本都有资格成为leader。传统上，这是Solr唯一支持的类型。
    - TLOG：这种类型的副本维护事务日志，但不会在本地索引文档更改。这种类型有助于加快索引，因为副本中不需要发生任何提交。当这种类型的副本需要更新其索引时，通过复制leader的索引来实现。这种类型的复制品也有资格成为碎片的leader；它会通过首先处理它的事务日志来做到这一点。如果它确实成为leader，它将表现得如同它是NRT类型的复制品一样。
    - PULL：这种类型的副本不会维护事务日志，也不会在本地修改索引文档。它只复制碎片leader的索引。没有资格成为碎片leader，根本不参加碎片leader候选。

如果在创建时未指定副本的类型，则将是NRT类型。  

### 组合群集中的副本类型

有三种推荐的副本类型的组合：  
  

    - 所有NRT副本
    - 所有TLOG副本
    - 带有PULL副本的TLOG副本

#### 所有NRT副本

将其用于小型到中型的群集，甚至是更新（索引）吞吐量不太高的大型群集。NRT是唯一支持soft-commits的副本，所以在需要NearRealTime时也可以使用这种组合。  

#### 所有TLOG副本

如果不需要NearRealTime，并且每个碎片的副本数量很高，则使用此组合，但是您仍然希望所有副本能够处理更新请求。  

#### TLOG副本加上PULL副本
如果不需要NearRealTime，则每个分片的副本数很高，并且希望通过文档更新提高搜索查询的可用性，即使这意味着暂时服务过时的结果，也可以使用此组合。  

#### 副本类型的其他组合

不推荐其他副本类型的组合。如果碎片中的多个副本正在写入自己的索引，而不是从NRT副本中复制，则leader选举可能导致碎片的所有副本与leader不同步，并且所有副本都必须复制完整索引。  

### 使用PULL副本恢复

如果PULL副本下降或离开群集，则需要考虑几个方案。  
  
如果PULL副本由于leader关闭而无法同步到leader，则不会发生复制。但是，它将继续提供查询服务。一旦它可以再次连接到leader，副本将恢复。  
如果PULL副本无法连接到ZooKeeper，它将从群集中删除，查询将不会从群集路由到它。  
如果PULL副本因任何其他原因死亡或无法访问，将无法进行查询。当它重新加入集群时，它将从leader复制，当完成时，它将准备再次提供查询服务。  

## 文档路由

Solr提供了在创建集合时通过指定router.name参数来指定集合使用的路由器实现的功能。  
  
如果使用compositeId路由器（默认），则可以在文档ID中使用前缀发送文档，该文档ID将用于计算哈希Solr用于确定文档发送到索引的碎片。前缀可以是您想要的任何东西（例如，它不一定是分片名称），但它必须是一致的，所以Solr的行为一致。  
例如，如果要为客户共同定位文档，则可以使用客户名称或ID作为前缀。如果您的客户是“IBM”，例如，ID为“12345”的文档，则可以在文档ID字段中插入前缀“IBM！12345”。感叹号（'！'）在这里至关重要，因为它区分了用于确定哪个分片用于指引文档的前缀。  
然后在查询时，用_route_参数（即，q=solr&amp;_route_=IBM!）将查询前缀（es）包含到查询中，以将查询引导到特定的碎片。在某些情况下，这可能会提高查询性能，因为它在查询所有碎片时克服了网络延迟。  
该compositeId路由器支持含有最多2级路由的前缀。例如：首先按地区进行前缀路由，然后由客户：“USA！IBM！12345”  
另一个用例可能是：客户“IBM”拥有大量文档，并且希望将其分散到多个碎片中。这种用例的语法是：shard_key/num!document_id；其中 /num是从复合哈希中使用的分片密钥中的比特数。  
  
所以IBM/3!12345将从分片密钥中获取3位，从独特的文档ID中获取29位，在集合中传播占用超过1/8的碎片。同样，如果num值是2，则会将文档分散到碎片数量的1/4。在查询时，您可以使用_route_参数（即q=solr&amp;_route_=IBM/3!）将查询的前缀（es）和位数包含到查询中，以将查询定向到特定的分片。  
如果您不想影响文档的存储方式，则不需要在文档ID中指定前缀。  
如果创建了集合并在创建时定义了“隐式”路由器，则还可以定义一个router.field参数，以使用每个文档中的字段标识文档所属的分片。如果指定的字段在文档中缺失，则该文档将被拒绝。您也可以使用该_route_参数来命名特定的分片。  

## 碎片分割

当您在SolrCloud中创建一个集合时，您决定要使用的初始数字碎片。但是事先很难知道您需要的碎片的数量，特别是当组织需求在可以随时改变的时候，以及稍后发现您选择错误的成本可能很高，包括创建新的内核和重新索引您的所有数据。  
  
分割碎片的能力在Collections API中。目前它允许将碎片分成两部分。现有的分片保持原样，所以拆分操作有效地将数据的两个副本作为新的分片。准备就绪后，您可以稍后删除旧的碎片。  
有关如何使用碎片分割的更多详细信息，请参考“Collection API SPLITSHARD命令”部分。  

## 在SolrCloud中忽略来自客户端应用程序的提交

在大多数情况下，在SolrCloud模式下运行时，索引客户端应用程序不应该发送显式提交请求。相反，您应该使用openSearcher=false和自动soft-commits配置自动提交以使最近的更新在搜索请求中可见。这可以确保在集群中定期执行自动提交。  
  
为了强制客户端应用程序不应该发送显式提交的策略，您应该更新将数据索引到SolrCloud的所有客户端应用程序。然而，这并不总是可行的，所以Solr提供了IgnoreCommitOptimizeUpdateProcessorFactory，它允许您忽略显式的提交或优化来自客户应用程序的请求，而不用重构您的客户应用程序代码。  
要激活这个请求处理器，您需要将以下内容添加到您的solrconfig.xml中：  
```
&lt;updateRequestProcessorChain name="ignore-commit-from-client" default="true"&gt;
  &lt;processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory"&gt;
    &lt;int name="statusCode"&gt;200&lt;/int&gt;
  &lt;/processor&gt;
  &lt;processor class="solr.LogUpdateProcessorFactory" /&gt;
  &lt;processor class="solr.DistributedUpdateProcessorFactory" /&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateRequestProcessorChain&gt;
```
如上面的示例所示，处理器将返回200给客户端，但会忽略“提交/优化”请求。请注意，您还需要连接SolrCloud所需的隐式处理器，因为此自定义链代替了默认链。  
  
在下面的示例中，处理器将使用带有自定义错误消息的403代码引发异常：  
```
&lt;updateRequestProcessorChain name="ignore-commit-from-client" default="true"&gt;
  &lt;processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory"&gt;
    &lt;int name="statusCode"&gt;403&lt;/int&gt;
    &lt;str name="responseMessage"&gt;Thou shall not issue a commit!&lt;/str&gt;
  &lt;/processor&gt;
  &lt;processor class="solr.LogUpdateProcessorFactory" /&gt;
  &lt;processor class="solr.DistributedUpdateProcessorFactory" /&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateRequestProcessorChain&gt;
```
最后，您还可以将其配置为只忽略优化，并让提交通过执行：  
```
&lt;updateRequestProcessorChain name="ignore-optimize-only-from-client-403"&gt;
  &lt;processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory"&gt;
    &lt;str name="responseMessage"&gt;Thou shall not issue an optimize, but commits are OK!&lt;/str&gt;
    &lt;bool name="ignoreOptimizeOnly"&gt;true&lt;/bool&gt;
  &lt;/processor&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateRequestProcessorChain&gt;
```
