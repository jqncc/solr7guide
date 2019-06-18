# IndexConfig

solrconfig.xml的`<indexConfig>`部分定义了Lucene索引编写器的低级行为。

默认情况下，这些设置在Solr包含的示例solrconfig.xml中注释掉，这意味着使用默认值。在大多数情况下，默认值是好的。

```
<indexConfig>
  ...
</indexConfig>
```


## 编写新的段

### ramBufferSizeMB

ramBufferSizeMB用于设置Lucene需要缓存在创建索引时待添加的document以及尚未刷新到索引目录的待删除document所需的内存缓存区的大小（单位MB，默认值为100MB）。一旦累积的文档更新超过设置的内存大小，则刷新挂起的更新。这也可以创建新段或触发合并。

```
<ramBufferSizeMB>100</ramBufferSizeMB>
```

### maxBufferedDocs

内存缓冲区缓存的document最大个数

```
<maxBufferedDocs>1000</maxBufferedDocs>
```

如果同时设置maxBufferedDocs与ramBufferSizeMB的话，任一条件满足就会触发一次索引刷新写入到索引目录的操作。

### useCompoundFile

控制新写入的（还没有合并的）索引段是否应该使用复合文件段格式。默认值是false。

```
<useCompoundFile>false</useCompoundFile>
```

## 合并索引段

### mergePolicyFactory

定义合并分段的完成方式。

Solr的默认值是使用 TieredMergePolicy，它将合并大小相同的段，并受到每层允许的段数限制。  
其他可用的策略有：LogByteSizeMergePolicy、LogDocMergePolicy和UninvertDocValuesMergePolicy。

```xml
<mergePolicyFactory class="org.apache.solr.index.TieredMergePolicyFactory">
  <int name="maxMergeAtOnce">10</int>
  <int name="segmentsPerTier">10</int>
</mergePolicyFactory>
```


### 控制段大小：合并因子

用户对配置TieredMergePolicy（或LogByteSizeMergePolicy）所做的最常见的调整是“合并因子”，以更改一次应合并多少个段。

对于TieredMergePolicy，这是通过设置`<int name="maxMergeAtOnce">`和`<int name="segmentsPerTier">`选项来控制的，而LogByteSizeMergePolicy只有一个`<int name="mergeFactor">`选项（全部默认值为10）。

要理解这些选项的重要性，请考虑使用LogByteSizeMergePolicy对索引进行更新时会发生什么情况：文档始终添加到最近打开的段中。当某个段填满时，会创建一个新分段，并随后进行更新。

如果创建新的段会导致最低层段的数量超过mergeFactor值，那么所有这些段被合并在一起形成一个大的段。因此，如果合并因子为10，则每次合并都会创建一个单独的段，该段大约是其十个成分中的每一个的十倍。当有10个这些较大的段时，他们又被合并成一个更大的单个段。这个过程可以无限期地继续下去。

当使用TieredMergePolicy时，过程是相同的，但不是一个单一的mergeFactor值，该segmentsPerTier设置被用作阈值来决定是否应该发生合并，并且该maxMergeAtOnce设置确定合并中应该包括多少段。

选择最佳的合并因素通常是索引速度与搜索速度的权衡。在索引中有更少的段通常会加速搜索，因为查找的位置更少。它也可以导致磁盘上的物理文件更少。但是为了保持低段的数量，合并会更频繁地发生，这会给系统增加负载并且减慢对索引的更新。

相反，保留更多的段可以加快索引，因为合并发生的次数少，因此更新不太可能触发合并。但是搜索的计算成本更高，并且可能会变慢，因为搜索条件必须在更多的索引段中查找。更快的索引更新也意味着更短的提交周转时间，这意味着更及时的搜索结果。

### 自定义合并策略

如果内置合并策略的配置选项不完全适合您的用例，则可以对它们进行自定义：通过创建您在配置中指定的自定义合并策略工厂，或者配置使用wrapped.prefix配置选项用于控制将如何配置其包装的工厂：

```xml
<mergePolicyFactory class="org.apache.solr.index.SortingMergePolicyFactory">
  <str name="sort">timestamp desc</str>
  <str name="wrapped.prefix">inner</str>
  <str name="inner.class">org.apache.solr.index.TieredMergePolicyFactory</str>
  <int name="inner.maxMergeAtOnce">10</int>
  <int name="inner.segmentsPerTier">10</int>
</mergePolicyFactory>
```

上面的例子显示了 Solr 的 SortingMergePolicyFactory 被配置为通过 "timestamp desc" 对合并段中的文档进行排序，并将 TieredMergePolicyFactory 配置为使用 maxMergeAtOnce=10 和 segmentsPerTier=10 的值，通过SortingMergePolicyFactory的wrapped.prefix选项定义的inner前缀。

### mergeScheduler

合并调度程序控制如何执行合并。默认ConcurrentMergeScheduler使用单独的线程在后台执行合并。替代方法是SerialMergeScheduler，不执行与单独线程的合并。

```
<mergeScheduler class="org.apache.lucene.index.ConcurrentMergeScheduler"/>
```


### mergedSegmentWarmer

当使用Solr进行近实时搜索时，可以将合并的段加热器配置为在合并提交之前为新合并的段加热读取器。这不是近实时搜索所必需的，但会在合并完成后减少打开新的近实时读取器时的搜索延迟。

```
<mergedSegmentWarmer class="org.apache.lucene.index.SimpleMergedSegmentWarmer"/>
```

## 复合文件段

每个Lucene段通常由十几个文件组成。Lucene可以配置为使用文件扩展名为.cfs; 将段的所有文件捆绑到单个复合文件中; 它是Compound File Segment的缩写。

由于各种原因，CFS 段可能会受到轻微的性能影响，具体取决于运行时环境。例如，文件系统缓冲区通常与打开的文件描述符关联，这可能会限制每个索引可用的总缓存空间。

在每个进程允许的打开文件数量有限的系统上，CFS可能会避免达到该限制。打开的文件限制也可以通过Linux / Unix ulimit命令对您的操作系统进行调整，或者其他操作系统的类似操作。

>注意：CFS: 新段与合并段：要配置新写入的段是否应使用 CFS, 请参阅上面描述的 useCompoundFile 设置。要配置合并段是否使用 CFS, 请查看 mergePolicyFactory 的 Javadocs。
许多合并策略实现都支持 noCFSRatio 和 maxCFSSegmentSizeMB 设置, 并使用默认值防止复合文件被用于大段, 但对于小段, 也要用复合文件。

## 索引锁

### 锁定类型

LockFactory选项指定要使用的锁定实现。

有效锁定类型选项集取决于您已配置的DirectoryFactory。下面列出的值受StandardDirectoryFactory（默认值）支持:

- native（默认）使用NativeFSLockFactory指定本机操作系统文件锁定。如果第二个Solr进程试图访问该目录，它将会失败。当多个Solr Web应用程序试图共享单个索引时，请勿使用它们。
- simple 使用 SimpleFSLockFactory 指定一个纯文件进行锁定。
- single（专家）使用SingleInstanceLockFactory。用于只读索引目录的特殊情况，或者当不可能有多个进程尝试修改索引（甚至是连续的）时。这种类型将防止同一个 JVM 内的多个内核试图访问相同的索引。警告！如果不同JVM中的多个Solr实例修改索引，则此类型不会防止索引损坏。
- hdfs使用HdfsLockFactory支持将索引和事务日志文件读写到HDFS文件系统。有关使用此功能的更多详细信息，请参阅HDFS上的运行Solr部分。

有关每个LockFactory的细微差别的更多信息，请参阅http://wiki.apache.org/lucene-java/AvailableLockFactories。

```
<lockType>native</lockType>
```

### writeLockTimeout

在IndexWriter上等待写入锁定的最长时间。缺省值是1000，单位毫秒。

```
<writeLockTimeout>1000</writeLockTimeout>
```

## 其他索引设置

还有一些其他参数可能对您的实现配置很重要。这些设置会影响对索引进行更新的方式或时间。

- **reopenReaders**  
  控制是否重新打开IndexReader，而不是关闭然后打开，这通常效率较低。默认值是true。  

- **deletionPolicy**  
 控制在回滚情况下如何保留提交。默认值是SolrDeletionPolicy，其中有最大提交数量（maxCommitsToKeep）的子参数，要保留的最优化提交数（`maxOptimizedCommitsToKeep`），以及任何提交保留（maxCommitAge）的最大保留期限，它支持`>DateMathParser`语法。

- infoStream  
  InfoStream设置指示底层Lucene类从索引过程中编写详细的调试信息作为Solr日志消息。  

```xml
<reopenReaders>true</reopenReaders>
<deletionPolicy class="solr.SolrDeletionPolicy">
  <str name="maxCommitsToKeep">1</str>
  <str name="maxOptimizedCommitsToKeep">0</str>
  <str name="maxCommitAge">1DAY</str>
</deletionPolicy>
<infoStream>false</infoStream>
```
