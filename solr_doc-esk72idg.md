## Solr性能统计参考 
<div class="content-intro view-box ">本页面解释了Solr公开的一些统计信息。  
  
有两种方法来检索度量。首先，您可以使用Metrics API，也可以启用JMX并从MBean请求处理程序或通过外部工具（如JConsole）获取度量标准。以下的说明将重点介绍如何使用Metrics
    API检索度量标准，但如果使用MBean请求处理程序或外部工具，则度量标准名称是相同的。  
这些统计是每个核心。当您在SolrCloud模式下运行时，这些统计信息将与单个副本的性能相关联。  

## 请求处理程序统计

### 更新请求处理程序

更新请求处理程序是将数据发送到Solr的端点。我们可以看到有多少更新请求正在被触发，执行速度有多快，以及有关请求的其他有价值的信息。  
注册表和路径：  
```
solr.&lt;core&gt;:UPDATE./update
```
您可以使用 API 请求更新请求处理程序统计信息，例如：http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=UPDATE。  

### 搜索请求处理程序

可以用来衡量和追踪搜索查询的次数，响应时间等。如果您没有使用“select”处理程序，那么路径需要进行适当的更改。同样的，如果您正在使用“sql”处理程序或“export”处理程序，那么也可以找到实时处理程序“get”或任何其他处理程序类似的统计信息。  
  
注册表和路径：  
```
solr.&lt;core&gt;:QUERY./select
```
您可以使用API​​请求（例如，http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=QUERY./select）以请求/select请求处理程序的统计信息。  

### 请求处理程序常用的统计信息

所有更新和搜索请求处理程序将提供以下统计信息。  
  
请求时间  
要获取请求时间，具体而言，您可以发送API请求，例如：  

    - http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=UPDATE./update.requestTimes
    - http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=QUERY./select.requestTimes
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">属性</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">15minRate  
            </td>
            <td>
                过去15分钟内收到的每秒请求数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">5minRate  
            </td>
            <td>
                过去5分钟内收到的每秒请求数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">p75_ms  
            </td>
            <td><span style="background-color: transparent;">请求处理时间为属于第七十五百分位的请求</span>。例如，如果收到100个请求，那么这个统计将报告第75个最快的请求时间。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">p95_ms  
            </td>
            <td>
                请求属于第95百分位的请求的处理时间（以毫秒为单位）。例如，如果接收到80个请求，则在这个统计中将报告第76个最快的请求时间。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">p999_ms  
            </td>
            <td>
                请求属于第99.9百分位的请求的处理时间（以毫秒为单位）。例如，如果收到1000个请求，则在此统计中将报告第999个最快的请求时间。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">p99_ms  
            </td>
            <td>
                请求属于第99百分位的请求的处理时间（以毫秒为单位）。例如，如果接收到200个请求，那么在这个统计中将报告第198个最快的请求时间。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">count  
  
            </td>
            <td>
                自Solr进程开始以来发出的请求总数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">median_ms  
            </td>
            <td>
                所有请求处理时间的中位数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">avgRequestsPerSecond  
            </td>
            <td>
                每秒接收的平均请求数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">avgTimePerRequest  
            </td>
            <td>
                处理请求所用的平均时间。随着时间的推移，这个参数将会衰减，<span style="background-color: transparent;">在过去5分钟内对活动有偏差</span><span style="background-color: transparent;">。</span>  
</td></tr></tbody></table>
### <h3>错误和其他时间：

还提供了其他类型的数据，如错误和超时。这些在不同的度量标准名称下可用。例如：  

    - http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=UPDATE./update.errors
    - http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=QUERY./select.errors

下表显示了要请求的指标名称和属性：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">度量标准名称</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.errors</code><code>UPDATE./update.errors</code>
                  
            </td>
            <td>
                处理程序遇到的错误数。除了计数错误外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.clientErrors</code><code>UPDATE./update.clientErrors</code>
                  
            </td>
            <td>
                客户端发出请求时的语法或解析错误的数量。除了计数错误外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.serverErrors</code><code>UPDATE./update.serverErrors</code>
                  
            </td>
            <td>
                执行请求时由服务器抛出的错误数量。除了计数错误外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.timeouts</code><code>UPDATE./update.timeouts</code>
                  
            </td>
            <td>
                收到部分结果的回复数量。除了计数超时事件外，还可以使用平均值，1分钟，5分钟和15分钟的速率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.totalTime</code><code>UPDATE./update.totalTime</code>
                  
            </td>
            <td>
                自Solr进程开始以来所有请求处理时间的总和。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>QUERY./select.handlerStart</code><code>UPDATE./update.handlerStart</code>
                  
            </td>
            <td>
                处理程序注册的时间。  
            </td>
        </tr>
    </tbody>
</table>
## 更新处理程序

本节包含有关增加的总数以及针对Solr核心进行了多少次提交的信息。  
  
注册表和路径：  
```
solr.&lt;core&gt;:UPDATE.updateHandler
```
您可以使用API​​请求（例如，http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=UPDATE.updateHandler）获取下表中显示的所有更新处理程序统计信息。  
以下介绍您可以获得的具体统计信息：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">属性</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.adds</code>
                  
            </td>
            <td>
                自上次提交以来的“<span style="background-color: transparent;">add</span><span style="background-color: transparent;">”请求的总数。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.autoCommitMaxTime</code>
                  
            </td>
            <td>
                两次自动提交执行之间的最长时间。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.autoCommits</code>
                  
            </td>
            <td>
                自动提交的总数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.commits</code>
                  
            </td>
            <td>
                执行的提交总数。  
                除了提交次数之外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.cumulativeAdds</code>
                  
            </td>
            <td>
                在整个生命周期内执行的“<span style="background-color: transparent;">effective</span><span style="background-color: transparent;">”添加的数量。计数器在执行“add”命令时递增，在执行“rollback”时递减。</span>  
                除了添加计数之外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.cumulativeDeletesById</code>
                  
            </td>
            <td>
                在整个生命周期中由ID执行的文档删除次数。计数器在执行“<span style="background-color: transparent;">delete</span><span style="background-color: transparent;">”命令时递增，在执行“回滚”时递减。</span>  
                除了删除计数外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.cumulativeDeletesByQuery</code>
                  
            </td>
            <td>
                在整个生命周期内通过查询执行的文档删除次数。计数器在执行“删除”命令时递增，在执行“<span style="background-color: transparent;">rollback</span><span style="background-color: transparent;">”时递减。</span>  
                除了删除计数外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.cumulativeErrors</code>
                  
            </td>
            <td>
                在整个生命周期内对文档执行添加/删除操作时收到的错误消息的数量。  
                除了计数错误外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.deletesById</code>
                  
            </td>
            <td>
                目前通过ID未提交删除。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.deletesByQuery</code>
                  
            </td>
            <td>
                目前通过查询未提交删除。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.docsPending</code>
                  
            </td>
            <td>
                未决提交的文档数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.errors</code>
                  
            </td>
            <td>
                在核心的生命周期内对文档执行添加/删除/提交/回滚（<span style="background-color: transparent;">addition/deletion/commit/rollback</span><span style="background-color: transparent;">）操作时收到的错误消息数量。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.expungeDeletes</code>
                  
            </td>
            <td>
                在清除删除时发出的提交命令的数量。  
                除了删除的删除计数外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.merges</code>
                  
            </td>
            <td>
                已发生索引合并的数量。  
                除了合并计数之外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.optimizes</code>
                  
            </td>
            <td>
                显式优化命令的数量。  
                除了优化计数之外，还可以使用平均值，1分钟，5分钟和15分钟的费率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.rollbacks</code>
                  
            </td>
            <td>
                执行的回滚数量。  
                除了计算回滚之外，还可以使用平均值，1分钟，5分钟和15分钟的速率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.softAutoCommitMaxTime</code>
                  
            </td>
            <td>
                两个软自动提交之间的最大文件“<span style="background-color: transparent;">adds</span><span style="background-color: transparent;">”。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>UPDATE.updateHandler.softAutoCommits</code>
                  
            </td>
            <td>
                执行的软提交的数量。  
            </td>
        </tr>
    </tbody>
</table>
## 缓存统计信息

### 文档缓存

这个缓存包含Lucene Document对象（每个文档的存储字段）。由于Lucene的内部文档ID是暂时的，所以这个缓存不能被auto-warmed。  
  
注册表和路径：  
```
 solr.&lt;core&gt;:CACHE.searcher.documentCache
```
您可以通过API请求（例如，http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=CACHE.searcher.documentCache）获取下表中显示的统计信息。  

### 查询结果缓存

此高速缓存包含以前搜索的结果：基于查询、排序和所请求的文档范围的文档ID的有序列表。  
  
注册表和路径：  
```
solr.&lt;core&gt;:CACHE.searcher.queryResultCache
```
您可以通过API请求（例如，http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=CACHE.searcher.queryResultCache）获取下表中显示的统计信息。  

### 筛选缓存

此缓存用于筛选所有与查询匹配的文档的无序集合。  
  
注册表和路径：  
```
solr.&lt;core&gt;:CACHE.searcher.filterCache
```
您可以通过API请求（例如，http://localhost:8983/solr/admin/metrics?group=core&amp;prefix=CACHE.searcher.filterCache）来获取下表中显示的统计信息。  

### 统计信息缓存

以下统计信息可用于上面提到的每个缓存：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">属性</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">cumulative_evictions  
            </td>
            <td>
                自该节点运行以来，所有高速缓存中的高速缓存逐出次数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">cumulative_hitratio  
            </td>
            <td>
                自该节点运行以来，所有高速缓存中的高速缓存命中与查找的比率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">cumulative_hits  
            </td>
            <td>
                自该节点运行以来，所有缓存中的缓存命中数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">cumulative_inserts  
            </td>
            <td>
                自该节点运行以来，所有高速缓存中的高速缓存插入次数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">cumulative_lookups  
            </td>
            <td>
                自该节点运行以来，所有高速缓存中的高速缓存查找数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">evictions  
  
            </td>
            <td>
                当前索引搜索器的缓存逐出次数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">hitratio  
            </td>
            <td>
                当前索引搜索器的高速缓存命中与查找的比率。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">hits  
  
            </td>
            <td>
                当前索引搜索器的匹配数。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">inserts  
  
            </td>
            <td>
                插入缓存中的数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">lookups  
  
            </td>
            <td>
                针对缓存的查找数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">size  
  
            </td>
            <td>
                该特定实例的缓存大小（以KB为单位）。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">warmupTime  
  
            </td>
            <td>
                注册索引搜索器的预热时间。考虑到高速缓存的“自动升温”这一次。  
            </td>
        </tr>
    </tbody>
</table>
有关Solr缓存的更多信息，请参见SolrConfig中的“ 查询设置 ”部分。  
