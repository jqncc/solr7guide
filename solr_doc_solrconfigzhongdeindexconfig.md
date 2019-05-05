## SolrConfig中的IndexConfig 
<div class="content-intro view-box ">solrconfig.xml的&lt;indexConfig&gt;部分定义了Lucene索引编写器的低级行为。
      
  
默认情况下，这些设置在Solr包含的示例solrconfig.xml中注释掉，这意味着使用默认值。在大多数情况下，默认值是好的。  
```
&lt;indexConfig&gt;
  ...
&lt;/indexConfig&gt;
```


## 编写新的段<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#writing-new-segments"/>


### ramBufferSizeMB<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#rambuffersizemb"/>

一旦累积的文档更新超过了这么多的内存空间（以兆字节为单位），那么待完成的更新就会被刷新。这也可以创建新的段或触发合并。通常使用这个设置对于maxBufferedDocs是可取的。如果同时在solrconfig.xml中设置了maxBufferedDocs与ramBufferSizeMB的话，当到达任何一个限制时，就会出现一个刷新。默认值是100Mb。  
```
&lt;ramBufferSizeMB&gt;100&lt;/ramBufferSizeMB&gt;
```


### maxBufferedDocs<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#maxbuffereddocs"/>

将文档更新的数量设置为内存中的缓冲区，然后再刷新为新段。这也可能触发合并。默认的 Solr 配置集由 RAM 使用量 (ramBufferSizeMB) 刷新。  
```
&lt;maxBufferedDocs&gt;1000&lt;/maxBufferedDocs&gt;
```


### useCompoundFile<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#usecompoundfile"/>

控制新写入的（还没有合并的）索引段是否应该使用复合文件段格式。默认值是false。  
```
&lt;useCompoundFile&gt;false&lt;/useCompoundFile&gt;
```


## 合并索引段<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#merging-index-segments"/>


### mergePolicyFactory<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#mergepolicyfactory"/>

定义合并分段的完成方式。
      
  
Solr中的默认值是使用 TieredMergePolicy，它将合并大小相同的段，并受到每层允许的段数限制。  
其他可用的政策是LogByteSizeMergePolicy、LogDocMergePolicy和UninvertDocValuesMergePolicy。有关这些策略的更多信息，请参阅MergePolicy javadocs。  
```
&lt;mergePolicyFactory class="org.apache.solr.index.TieredMergePolicyFactory"&gt;
  &lt;int name="maxMergeAtOnce"&gt;10&lt;/int&gt;
  &lt;int name="segmentsPerTier"&gt;10&lt;/int&gt;
&lt;/mergePolicyFactory&gt;
```


### 控制段大小：合并因素<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#merge-factors"/>

用户对配置TieredMergePolicy（或LogByteSizeMergePolicy）所做的最常见的调整是“合并因素”，以一次更改应合并的段数。
      
  
对于TieredMergePolicy，这是通过设置&lt;int name="maxMergeAtOnce"&gt;和&lt;int name="segmentsPerTier"&gt;选项来控制的，而LogByteSizeMergePolicy只有一个&lt;int name="mergeFactor"&gt;选项（全部默认值为10）。  
要理解这些选项的重要性，请考虑使用LogByteSizeMergePolicy对索引进行更新时会发生什么情况：文档始终添加到最近打开的段中。当某个段填满时，会创建一个新分段，并随后进行更新。  
如果创建新的段会导致最低层段的数量超过mergeFactor值，那么所有这些段被合并在一起形成一个大的段。因此，如果合并因子为10，则每次合并都会创建一个单独的段，该段大约是其十个成分中的每一个的十倍。当有10个这些较大的段时，他们又被合并成一个更大的单个段。这个过程可以无限期地继续下去。  
当使用TieredMergePolicy时，过程是相同的，但不是一个单一的mergeFactor值，该segmentsPerTier设置被用作阈值来决定是否应该发生合并，并且该maxMergeAtOnce设置确定合并中应该包括多少段。  
选择最佳的合并因素通常是索引速度与搜索速度的权衡。在索引中有更少的段通常会加速搜索，因为查找的位置更少。它也可以导致磁盘上的物理文件更少。但是为了保持低段的数量，合并会更频繁地发生，这会给系统增加负载并且减慢对索引的更新。  
相反，保留更多的段可以加快索引，因为合并发生的次数少，使得更新不太可能触发合并。但是搜索的计算成本更高，并且可能会变慢，因为搜索条件必须在更多的索引段中查找。更快的索引更新也意味着更短的提交周转时间，这意味着更及时的搜索结果。  

### 定制合并策略<a href="http://lucene.apache.org/solr/guide/7_0/indexconfig-in-solrconfig.html#customizing-merge-policies"/>

如果内置合并策略的配置选项不完全适合您的用例，则可以对它们进行自定义：通过创建您在配置中指定的自定义合并策略工厂，或者配置使用wrapped.prefix配置选项用于控制将如何配置其包装的工厂：
      
  
```
&lt;mergePolicyFactory class="org.apache.solr.index.SortingMergePolicyFactory"&gt;
  &lt;str name="sort"&gt;timestamp desc&lt;/str&gt;
  &lt;str name="wrapped.prefix"&gt;inner&lt;/str&gt;
  &lt;str name="inner.class"&gt;org.apache.solr.index.TieredMergePolicyFactory&lt;/str&gt;
  &lt;int name="inner.maxMergeAtOnce"&gt;10&lt;/int&gt;
  &lt;int name="inner.segmentsPerTier"&gt;10&lt;/int&gt;
&lt;/mergePolicyFactory&gt;
```

上面的例子显示了 Solr 的 SortingMergePolicyFactory 被配置为通过 "timestamp desc" 对合并段中的文档进行排序，并将 TieredMergePolicyFactory 配置为使用 maxMergeAtOnce=10 和 segmentsPerTier=10 的值，通过SortingMergePolicyFactory的wrapped.prefix选项定义的inner前缀。有关使用 SortingMergePolicyFactory 的详细信息, 请参阅
    segmentTerminateEarly 参数。
      
  

### mergeScheduler

合并调度程序控制如何执行合并。默认ConcurrentMergeScheduler使用单独的线程在后台执行合并。替代方法是SerialMergeScheduler，不执行与单独线程的合并。  
```
&lt;mergeScheduler class="org.apache.lucene.index.ConcurrentMergeScheduler"/&gt;
```


### mergedSegmentWarmer

使用Solr进行近实时搜索合并段时，可以配置为在合并提交之前在新合并的段上预热读取器。这对于近乎实时的搜索并不是必需的，但是在合并完成之后将减少在打开新的近实时阅读器时的搜索延迟。  
```
&lt;mergedSegmentWarmer class="org.apache.lucene.index.SimpleMergedSegmentWarmer"/&gt;
```


## 复合文件段

每个Lucene段通常由十几个文件组成。可以将Lucene配置为将一个段的所有文件打包成单个复合文件，其文件扩展名为.cfs；它是复合文件段的缩写。
      
  
由于各种原因，CFS 段可能会受到轻微的性能影响，具体取决于运行时环境。例如，文件系统缓冲区通常与打开的文件描述符关联，这可能会限制每个索引可用的总缓存空间。  
在每个进程允许的打开文件数量有限的系统上，CFS可能会避免达到该限制。打开的文件限制也可以通过Linux / Unix ulimit命令对您的操作系统进行调整，或者其他操作系统的类似操作。  
注意：CFS: 新段与合并段：要配置新写入的段是否应使用 CFS, 请参阅上面描述的 useCompoundFile 设置。要配置合并段是否使用 CFS, 请查看 mergePolicyFactory 的 Javadocs。  
许多合并策略实现都支持 noCFSRatio 和 maxCFSSegmentSizeMB 设置, 并使用默认值防止复合文件被用于大段, 但对于小段, 也要用复合文件。  

## 索引锁


### 锁定类型

LockFactory选项指定要使用的锁定实现。
      
  
这组有效的锁定类型选项取决于您配置的DirectoryFactory。以下列出的值受StandardDirectoryFactory（默认）支持：  

    - native（默认）使用NativeFSLockFactory指定本机操作系统文件锁定。如果第二个Solr进程试图访问该目录，它将会失败。当多个Solr Web应用程序试图共享单个索引时，请勿使用它们。
    - simple 使用 SimpleFSLockFactory 指定一个纯文件进行锁定。
    - single（专家）使用SingleInstanceLockFactory。用于只读索引目录的特殊情况，或者当不可能有多个进程尝试修改索引（甚至是连续的）时。这种类型将防止同一个 JVM 内的多个内核试图访问相同的索引。警告！如果不同JVM中的多个Solr实例修改索引，则此类型不会防止索引损坏。
    - hdfs使用HdfsLockFactory支持将索引和事务日志文件读写到HDFS文件系统。有关使用此功能的更多详细信息，请参阅HDFS上的运行Solr部分。

有关每个LockFactory的细微差别的更多信息，请参阅http://wiki.apache.org/lucene-java/AvailableLockFactories。  
```
&lt;lockType&gt;native&lt;/lockType&gt;
```


### writeLockTimeout

在IndexWriter上等待写入锁定的最长时间。缺省值是1000，以毫秒表示。  
```
&lt;writeLockTimeout&gt;1000&lt;/writeLockTimeout&gt;
```


## 其他索引设置

还有一些其他参数可能对您的实现配置很重要。这些设置会影响对索引进行更新的方式或时间。  
- reopenReaders  

   
        控制是否重新打开IndexReader，而不是关闭然后打开，这通常效率较低。默认值是true。  
    
- deletionPolicy  

   
        控制在回滚情况下如何保留提交。默认值是<code>SolrDeletionPolicy</code>，其中有最大提交数量（<code>maxCommitsToKeep</code>）的子参数，要保留的最优化提交数（<code>maxOptimizedCommitsToKeep</code>），以及任何提交保留（<code>maxCommitAge</code>）的最大保留期限，它支持<code>DateMathParser</code>语法。  
    
- infoStream  

   
        InfoStream设置指示底层Lucene类从索引过程中编写详细的调试信息作为Solr日志消息。  

```
&lt;reopenReaders&gt;true&lt;/reopenReaders&gt;
&lt;deletionPolicy class="solr.SolrDeletionPolicy"&gt;
  &lt;str name="maxCommitsToKeep"&gt;1&lt;/str&gt;
  &lt;str name="maxOptimizedCommitsToKeep"&gt;0&lt;/str&gt;
  &lt;str name="maxCommitAge"&gt;1DAY&lt;/str&gt;
&lt;/deletionPolicy&gt;
&lt;infoStream&gt;false&lt;/infoStream&gt;
```
