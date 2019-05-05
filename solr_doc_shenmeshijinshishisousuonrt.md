## 什么是Solr近实时搜索（NRT） 
<div class="content-intro view-box "><span style="font-family: inherit; font-size: 16px; font-weight: 600;">近实时搜索</span>
      
  
什么是 Solr 近实时搜索？近实时（Near Real Time，简称NRT）搜索意味着文档在被索引后几乎立即可用于搜索。  
这允许在“接近”实时看到文档的添加和更新。在提交过程中，Solr 不会阻止更新。在打开新的索引和返回的新搜索之前，它也不会等待后台合并完成。  
通过使用 NRT，您可以将 commit 命令修改为软提交，这样可以避免可能造成代价的标准提交的部分内容。您仍然希望执行标准提交以确保文档处于稳定存储状态，但软提交允许您在此期间同时查看索引的非常近实时视图。  
但是，要特别注意缓存和自动设置（autowarm）设置，因为它们会对 NRT 性能产生很大的影响。  

## 提交和优化（Commits 和 Optimizing）

## <a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#commits-and-optimizing"/>

提交操作使索引更改对新的搜索请求可见。一个硬提交（hard commit）使用事务日志来获得的最新文档更改的ID，并在索引文件上调用 fsync，以确保它们已被刷新到稳定的存储，并且不会因电源故障而导致数据丢失。当前的事务日志被关闭，并打开一个新的事务日志。有关数据丢失问题，请参阅下面的“事务日志”讨论。  
软提交（soft commit）的速度要快得多，因为它只会使索引更改可见，而不 fsync 索引文件，或者编写新的索引描述符或启动新的事务日志。搜索具有 NRT 要求（希望对搜索快速可见的索引更改）的集合将希望经常进行软提交，但不太经常提交。softCommit 可能花费较少，但它不是免费的，因为它可能会降低吞吐量。有关数据丢失问题，请参阅下面的“事务日志”讨论。  
优化就像硬提交, 除非它强制所有的索引段首先合并到单个段中。根据使用情况，这个操作应该很少进行，因为它涉及读取和重写整个索引。片段通常会随着时间的推移而合并（由合并政策确定），而优化只是迫使这些合并立即发生。  
软提交使用两个参数：maxDocs 和 maxTime。  
- maxDocs  

   
        整数。定义在将它们推送到索引之前要排队的文档的数量。它与<code>update_handler_autosoftcommit_max_time</code>参数一起工作，如果达到任何限制，文档将被推送到索引。  
    
- maxTime  

        将文档推送到索引之前要等待的毫秒数。它与<code>update_handler_autosoftcommit_max_docs</code>参数一起工作，如果达到任何限制，文档将被推送到索引。  
    

使用 maxDocs 和 maxTime 可以适当地调整您的提交策略。  

### 事务日志（tlogs）<a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#transaction-logs-tlogs"/>

事务日志是至少最后一个 N (默认 100) 文档索引的 "滚动窗口"。Tlogs 在 solrconfig. xml 中配置，包括 N 的值。当前的事务日志是关闭的，每当发生各种硬性提交时都会打开一个新的事务日志。软提交对事务日志没有影响。
      
  
启用 tlog 时，添加到索引中的文档将在索引调用返回到客户端之前写入 tlog。如果发生不正常的关闭（例如：断电、JVM 崩溃等），则任何写入到 Solr 停止时打开的 tlog 的文档将在启动时重播。  
当 Solr 被正常关闭（即使用 bin/solr stop 命令等）时，Solr 将关闭 tlog 文件和索引段，因此在启动时不需要重播。  

### AutoCommits<a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#autocommits"/>

自动提交也使用 maxDocs 和 maxTime 参数。然而，在许多策略中使用硬性 autocommit 和 autosoftcommit 实现更灵活的提交是有用的。
      
  
一个常见的配置是每隔1 - 10分钟执行一次硬性 autocommit，每秒钟执行一次 autosoftcommit。使用这种配置，新文档将在添加后大约一秒钟内显示出来，如果断电，软提交将会丢失，除非已经完成了硬提交。  
例如：  
```
&lt;autoSoftCommit&gt;
  &lt;maxTime&gt;1000&lt;/maxTime&gt;
&lt;/autoSoftCommit&gt;
```

最好使用 maxTime 而不是 maxDocs 修改 autoSoftCommit，尤其是在通过提交操作对大量文档进行索引时。关闭 autoSoftCommit 批量索引也是更好的选择。  

### 提交和优化的可选属性<a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#optional-attributes-for-commit-and-optimize"/>

- waitSearcher  
 
        阻塞，直到打开新的搜索器并将其注册为主要查询搜索器，使更改可见。默认是<code>true</code>。  
    
- OpenSearcher  

        打开一个新的搜索器，使目前为止搜索到的所有文档都可见。默认是<code>true</code>。  
    
- softCommit  

        执行一个软提交。这将更快地刷新索引的视图，但不保证文档稳定存储。默认是<code>false</code>。  
    
- expungeDeletes  

        仅适用于<code>commit</code>是否从段中清除已删除的数据。默认是<code>false</code>。  
    
- maxSegments  

    
        仅适用于<code>optimize</code>最多可以优化的部分。默认是<code>1</code>。  
    

提交和优化具有可选属性的示例：  
```
&lt;commit waitSearcher="false"/&gt;
&lt;commit waitSearcher="false" expungeDeletes="true"/&gt;
&lt;optimize waitSearcher="false"/&gt;
```


### 将提交和提交的参数作为 URL 的一部分传递<a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#passing-commit-and-commitwithin-parameters-as-part-of-the-url"/>

更新处理程序也可以获取 commit 相关参数作为更新 URL 的一部分。这个例子添加了一个小的测试文档，并在之后立即产生一个明确的提交：  
```
http://localhost:8983/solr/my_collection/update?stream.body=&lt;add&gt;&lt;doc&gt;
   &lt;field name="id"&gt;testdoc&lt;/field&gt;&lt;/doc&gt;&lt;/add&gt;&amp;commit=true
```

或者，您可能想要使用这个：  
```
http://localhost:8983/solr/my_collection/update?stream.body=&lt;optimize/&gt;
```

这个例子使得索引被优化到最多10个段，但是不会等待，直到完成（waitFlush=false）：  
```
curl 'http://localhost:8983/solr/my_collection/update?optimize=true&amp;maxSegments=10&amp;waitFlush=false'
```

这个例子增加了一个小小的测试文档，包含了 commitWithin，告诉 Solr 确保文档不晚于10秒后提交（这个方法通常优于显式提交）：  
```
curl http://localhost:8983/solr/my_collection/update?commitWithin=10000
  -H "Content-Type: text/xml" --data-binary '&lt;add&gt;&lt;doc&gt;&lt;field name="id"&gt;testdoc&lt;/field&gt;&lt;/doc&gt;&lt;/add&gt;'
```

虽然<code>stream.body</code>功能对于开发和测试非常重要，但它通常不应该在生产系统中启用，因为它允许用户读取可能改变系统状态的权限。该功能在默认情况下是禁用的。有关详细信息，请参见SolrConfig 中的 RequestDispatcher。  

### 更改默认的 commitWithin 行为<a href="http://lucene.apache.org/solr/guide/7_0/near-real-time-searching.html#changing-default-commitwithin-behavior"/>

这些 commitWithin 设置允许强制文档提交在规定的时间段内发生。这在Solr近实时搜索中最常使用，因此默认情况下是执行软提交。但是，这并没有将新文档复制到主/从环境中的从属服务器。如果这是实现的要求，则可以通过添加一个参数来强制执行一个硬提交，如下例所示：
      
  
```
&lt;commitWithin&gt;
  &lt;softCommit&gt;false&lt;/softCommit&gt;
&lt;/commitWithin&gt;
```

有了这个配置，当您将 commitWithin 作为更新消息的一部分进行调用时，每次都会自动执行一个硬性提交。  
