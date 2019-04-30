## Solr分布式请求 
当一个Solr节点收到一个搜索请求时，这个请求就会在后台传送到作为要搜索的集合一部分的碎片的副本。  
  
所选择的副本充当聚合器：它创建内部请求以随机选择集合中每个分片的副本，协调响应，根据需要发出任何后续内部请求（例如，改进facet值或请求额外存储的字段），并为客户构建最终响应。  
## 限制查询的碎片
虽然使用SolrCloud的优点之一是能够查询分布在各种碎片中的非常大的集合，但在某些情况下，您可能会知道您只对碎片的一部分结果感兴趣。您可以选择搜索所有数据或仅搜索部分数据。  
  
查询集合的所有碎片应该看起来很熟悉；就好像SolrCloud甚至没有发挥作用：  

```
http://localhost:8983/solr/gettingstarted/select?q=*:*
```
另一方面，如果您只想搜索一个分片，则可以通过其逻辑ID指定该分片，如下所示：  

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&amp;shards=shard1
```
如果您想搜索一组分片id，您可以一起指定它们：  

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&amp;shards=shard1,shard2
```
在上面的两个例子中，分片Id（s）将被用来挑选该分片的随机副本。  
或者，您可以指定您希望使用的显式副本来代替分片ID：  
```
http://localhost:8983/solr/gettingstarted/select?q=*:*&amp;shards=localhost:7574/solr/gettingstarted,localhost:8983/solr/gettingstarted
```
或者，您可以通过使用管道符号（|）来指定一个副本列表，以便为单个分片选择（用于负载平衡）：  
```
http://localhost:8983/solr/gettingstarted/select?q=*:*&amp;shards=localhost:7574/solr/gettingstarted|localhost:7500/solr/gettingstarted
```
当然，您可以指定一个由一系列副本（由管道“|”分隔）定义的分片列表（用逗号“，”分隔）。在这个例子中，查询了2个分片，第一个是来自shard1的随机副本，第二个是显式管道分隔列表中的随机副本：  
```
http://localhost:8983/solr/gettingstarted/select?q=*:*&amp;shards=shard1,localhost:7574/solr/gettingstarted|localhost:7500/solr/gettingstarted
```
## 配置ShardHandlerFactory
您可以直接配置在Solr中的分布式搜索中使用的并发和线程池的各个方面。这允许更精细的粒度的控制，您可以调整它以满足您自己的具体要求。默认配置有利于吞吐量超过延迟。  
  
要配置标准搜索处理程序，请在solrconfig.xml中提供类似如下的配置：  

```
&lt;requestHandler name="/select" class="solr.SearchHandler"&gt;
  &lt;!-- other params go here --&gt;
  &lt;shardHandler class="HttpShardHandlerFactory"&gt;
    &lt;int name="socketTimeOut"&gt;1000&lt;/int&gt;
    &lt;int name="connTimeOut"&gt;5000&lt;/int&gt;
  &lt;/shardHandler&gt;
&lt;/requestHandler&gt;
```
可以指定的参数如下：  

**socketTimeout**
允许套接字等待的时间（以毫秒为单位）。默认情况下为<code>0</code>，将使用操作系统的默认值。  
**connTimeout**
用于绑定/连接套接字的时间（以毫秒为单位）。默认情况下为<code>0</code>，将使用操作系统的默认值。  
**maxConnectionsPerHost**
在分布式搜索中对每个独立分片进行的最大并发连接数。默认是<code>20</code>。  
**maxConnections**
分布式搜索中的最大并发连接数。默认是<code>10000</code>  
**corePoolSize**
在协调分布式搜索中使用的线程数保持最低限制。默认是<code>0</code>。  
**maximumPoolSize**
用于协调分布式搜索的最大线程数。默认是<code>Integer.MAX_VALUE</code>。  
**maxThreadIdleTime**
在线程被缩减以响应减少的负载之前等待的时间量（秒）。默认是<code>5</code>。  
**sizeOfQueue**
如果指定，则线程池将使用后备队列，而不是直接的越区切换缓冲区。高吞吐量的系统将希望将其配置为直接手动关闭（使用<code>-1</code>）。渴望更好的延迟的系统将希望配置合理的队列大小来处理请求中的变化。默认是<code>-1</code>。  
**fairnessPolicy**
选择处理公平策略队列的JVM细节，如果启用，分布式搜索将以先进先出的方式进行处理，其代价是吞吐量。如果禁用的吞吐量将优先于滞后时间。默认是<code>false</code>。  

## 配置statsCache（分布式IDF）
需要文档和术语统计来计算相关性。当涉及到文档统计计算时，Solr提供了四种实现方式：  
  
- LocalStatsCache：这只使用本地术语和文档统计信息来计算相关性。在统一分布在分片上的情况下，这种方式工作得很好。如果没有配置&lt;statsCache&gt;，则这个选项是默认的。
- ExactStatsCache：此实现使用文档频率的全局值（在集合中）。
- ExactSharedStatsCache：这与其功能中的确切统计高速缓存完全相同，但是全局统计信息被重复用于具有相同条件的后续请求。
- LRUStatsCache：这个实现使用一个LRU缓存来保存在请求之间共享的全局统计信息。
可以通过在 solrconfig. xml 中设置 &lt;statsCache&gt; 来选择实现。例如，以下行使Solr使用ExactStatsCache实现：  
```
&lt;statsCache class="org.apache.solr.search.stats.ExactStatsCache"/&gt;
```
## 避免分布式死锁
每个分片服务于top-level查询请求，然后向所有其他分片发出子请求。应该注意确保服务于HTTP请求的线程的最大数量大于来自top-level客户机和其他分片的可能数量的请求。如果不是这种情况，则配置可能会导致分布式死锁。  
  
例如，在两个分片的情况下可能发生死锁，每个分片只有一个线程来处理HTTP请求。两个线程都可以同时接收top-level请求，并相互发送子请求。由于没有剩余的线程来处理请求，传入的请求将被阻塞，直到其他待处理的请求完成，但是由于它们正在等待子请求，所以它们不会完成。通过确保Solr被配置为处理足够数量的线程，可以避免像这样的死锁情况。  
## preferLocalShards参数
Solr允许您传递一个名为preferLocalShards的可选布尔参数，以指示分布式查询在可用时应首选分片的本地副本。换句话说，如果查询包含preferLocalShards=true，那么查询控制器将查找本地副本来为查询服务，而不是从整个集群中随机选择副本。当查询请求每个文档返回许多字段或大字段时，这非常有用，因为它避免了在本地可用时通过网络传输大量数据。此外，此功能可用于最大限度地降低性能下降的问题副本的影响，因为它降低了降级的副本将被其他健康副本命中的可能性。  
  
最后，随着集合中分片数量的增加，这个特性的值会减少，因为查询控制器将不得不将查询指向大部分分片的非本地副本。换句话说，这个特性对于优化针对具有少量分片和多个副本的集合的查询是非常有用的。此外，只有在负责所有正在查询的集合的副本的节点间进行负载平衡请求时，才应使用此选项，因为Solr的CloudSolrClient将执行此操作。如果不是负载均衡，则此功能会在群集中引入热点，因为查询不会在群集中均匀分布。  
  
  

