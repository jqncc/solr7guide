# UpdateHandlers

本节中的设置在solrconfig.xml中的`<updateHandler>`元素中进行配置，可能会影响索引更新的性能。这些设置影响在内部进行更新的方式。`<updateHandler>`配置不会影响处理客户端更新请求的RequestHandler的更高级配置。

```xml
<updateHandler class="solr.DirectUpdateHandler2">
  ...
</updateHandler>
```

## 提交（Commits）

发送到Solr的数据在被提交到索引之前是不可搜索的。这样做的原因是，在某些情况下，提交可能会很慢，并且应该与其他可能的提交请求隔离进行，以避免覆盖数据。所以，最好提供对何时提交数据的控制。有几个选项可用来控制提交的时间。

### commit和softCommit

在Solr中，commit是一个动作，要求Solr将这些改变“提交”到Lucene索引文件中。当客户端包含具有更新请求的commit=true参数时，可以确保索引更新完成后，就会将更新中受添加和删除影响的所有索引段持久化（如写入磁盘）。

如果指定了另外一个标志：softCommit=true，那么Solr执行“soft commit”，这意味着Solr会快速地将您的更改提交到Lucene数据结构，但不能保证Lucene索引文件已完成持久化。这是近实时存储的一个实现，这个功能可以提高文档的可见性，因为您不必等待后台合并和存储（如果使用SolrCloud，则到ZooKeeper ）完成，然后再转到其他位置。完全提交意味着，如果服务器崩溃，Solr将确切地知道您的数据存储在哪里；soft commit意味着数据被存储，但位置信息尚未被存储。soft commit为您提供更快的可见性，因为它不等待后台合并完成。

### 自动提交

这些设置控制挂起的更新将自动推送到索引的频率。自动提交的另一种替代方法是使用commitWithin，它可以在向Solr发送更新请求（即推送文档时）或更新RequestHandler时定义。

- maxDocs  
 自上次提交以来发生的更新次数。

- maxTime  
 自最早未提交更新以来的毫秒数。  

- openSearcher  
是否在执行提交时打开新的搜索器。如果是false，则提交将刷新最近的索引更改为稳定的存储，但不会导致新的搜索器被打开，使这些更改可见。默认是true。

如果达到了 maxDocs 或 maxTime 限制，Solr 将自动执行提交操作。如果缺少自动提交标记，那么只有显式提交才会更新索引。是否使用自动提交取决于您的应用程序的需求。

确定最佳自动提交设置是性能和准确性之间的权衡。导致频繁更新的设置将提高搜索的准确性，因为新内容可以更快地进行搜索，但由于频繁的更新，性能可能会受到影响。不太频繁的更新可能会提高性能，但在查询中显示更新需要更长的时间。

```xml
<autoCommit>
  <maxDocs>10000</maxDocs>
  <maxTime>30000</maxTime>
  <openSearcher>false</openSearcher>
</autoCommit>
```

指定自动软提交

```xml
<autoSoftCommit>
  </maxTime>60000</maxTime>
</autoSoftCommit>
```

### commitWithin

commitWithin设置允许强制文档提交在定义的时间段内发生。这常用于近实时搜索，因此默认情况下执行软提交。但是，这不会将新文档复制到主/从环境中的从属服务器。如果这是您的实现要求，您可以通过添加参数强制进行硬提交,如下例所示：

```xml
<commitWithin>
  <softCommit>false</softCommit>
</commitWithin>
```

有了这个配置，当您将commitWithin作为更新消息的一部分进行调用时，每次都会自动执行一个硬性提交。

## 事件监听器

UpdateHandler也可以配置与更新相关的事件侦听器。这些可以在任何commit（event="postCommit"）之后触发，或者仅在优化命令（event="postOptimize"）之后触发。

用户可以编写自定义更新事件监听器类，但常见的方式是通过RunExecutableListener运行外部可执行文件：

- exe  
要运行的可执行文件的名称。它应该包括文件的路径，相对于Solr home。
- dir  
用作工作目录的目录。默认是当前目录（"."）。
- wait  
强制调用线程等待，直到可执行文件返回响应。默认是<code>true</code>。  
- args  
任何传递给程序的参数。默认值是none。
- env  
任何环境变量设置。默认值是none。


## 事务日志

如RealTime Get部分所述，该功能需要事务日志。它在solrconfig.xml的updateHandler部分中进行了配置。

Realtime Get当前依赖于更新日志功能,该功能默认启用。它依赖于在solrconfig.xml中配置的更新日志，其内容如下：

```xml
<updateLog>
  <str name="dir">${solr.ulog.dir:}</str>
</updateLog>
```

另外三个专家级配置设置会影响索引性能以及副本在必须进入完全恢复之前可以在多大程度上落后于更新. 有关更多信息，请参见写入端容错部分：

- numRecordsToKeep  
每个日志保留的更新记录数。默认是100。

- maxNumLogsToKeep
保留最大日志数。默认是10。

- numVersionBuckets  
检查重新排序的更新时用于跟踪最大版本值的桶数（bucket）; 增加此值以降低在高容量索引期间同步对版本存储桶的访问的成本 ，每个core的堆空间需要
`(8 bytes (long) * numVersionBuckets)`。默认是65536。  

一个例子，要被包括在solrconfig.xml的`<config><updateHandler>`下，将使用上述高级设置：

```xml
<updateLog>
  <str name="dir">${solr.ulog.dir:}</str>
  <int name="numRecordsToKeep">500</int>
  <int name="maxNumLogsToKeep">20</int>
  <int name="numVersionBuckets">65536</int>
</updateLog>
```
