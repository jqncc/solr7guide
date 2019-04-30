## Solr结果分页 
<div class="content-intro view-box ">在大多数搜索应用程序中，“top” 匹配结果（按分数或其他标准排序）将显示给某些用户。  
在许多应用程序中，这些排序结果的用户界面以“页面”显示给用户，其中包含固定数量的匹配结果，而用户通常不会查看经过前几页结果的结果。  

## 基本分页

在 Solr 中，使用 start 和 rows 参数支持这种基本的分页搜索，通过使用 queryResultCache 和根据预期的页面大小调整 queryResultWindowSize 配置选项，可以调整这种常见行为的性能。
      
  

### 基本分页示例

谈到基础的分页，最简单的方法就是将所需的页码乘以每页的行数 (将第一页的页码视为 "0")。如在以下伪代码中所示：  
```
function fetch_solr_page($page_number, $rows_per_page) {
  $start = $page_number * $rows_per_page
  $params = [ q = $some_query, rows = $rows_per_page, start = $start ]
  return fetch_solr($params)
}
```


### 索引更新对基本分页的影响

Solr 请求中指定的 start 参数指示客户端希望 Solr 用作当前“页面”开头的完整排序匹配列表中的绝对 “偏移量”。
      
  
如果索引修改 (如添加或删除文档) 影响与查询匹配的有序文档的顺序，则会在客户端的两个请求之间发生，从而导致后续页的结果，那么这些修改可能会产生在多个页上返回的同一文档，或者当结果集收缩或增大时，文档被 "跳过"。  
例如，考虑一个包含 26 个文档的索引，如下所示：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">ID</th>
            <th style="text-align: center;">名称</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">1  
            </td>
            <td>
                <p style="text-align: center;">A  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">2  
            </td>
            <td>
                <p style="text-align: center;">B  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">...  
            </td>
            <td style="text-align: center;">...</td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">26  
            </td>
            <td>
                <p style="text-align: center;">Z  
            </td>
        </tr>
    </tbody>
</table>
      
后跟以下请求和索引修改交错：  

    - 客户请求
```
 q=:&amp;rows=5&amp;start=0&amp;sort=name asc 
```
<span style="background-color: initial; color: rgb(51, 51, 51); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: pre-wrap;"> </span>带有
        1-5 的 ID 的文档将返回给客户端
    - ID 为 3 的文档被删除
    - 客户端请求“page＃2” 使用
```
q=:&amp;rows=5&amp;start=5&amp;sort=name asc
```
        文件 7-11 将被退回；  
已跳过文档 6，因为它现在是所有匹配结果的排序集合中的第5个文档 - 它将在 “page＃1” 的新请求上返回。
              
          
    
    - 现在添加了ID为90，91以及92的3页新的文件；这三个文件都有一个名称
    - 客户端请求“第3页”使用：
```
q=:&amp;rows=5&amp;start=10&amp;sort=name asc
```
文档9、10和11已在 page #2 和 page #3 中返回，因为它们移到了排序结果列表中的更远的后面。

在典型的情况下，从索引更改对分页搜索的影响不会显著影响用户体验 - 因为它们在相当静态的集合中极少发生，或者是因为用户认识到数据集合不断发展并期望看到文档在结果集中上下移动。  

### “深度分页”的性能问题

在某些情况下，Solr 搜索的结果不适用于简单的分页用户界面。
      
  
当您希望从 Solr 中获取大量的排序结果，并将其输入到外部系统中时，为 startor rows 参数使用非常大的值可能是非常低效的。分页使用 start 和 rows 不仅要求 Solr 计算（和排序）在内存中应为当前页面提取的所有匹配文档，而且还需要在以前的页面上出现的所有文档。  
虽然请求 start=0&amp;rows=1000000 可能显然是低效率的，因为它要求 Solr 维护和排序一百万份文档，同样 start=999000&amp;rows=1000，由于同样的原因，请求同样是低效的。Solr 无法计算出排序顺序中的哪个匹配文档是 999001 个结果，而无需先确定前 999000 个匹配排序结果是什么。  
如果索引是分布式的（在 SolrCloud 模式下运行时常见），则从每个分片中检索一百万个文档。对于十个分片索引，必须检索和排序一千万个条目以找出与这些查询参数匹配的 1000 个文档。  

## 获取大量排序结果：Cursor

作为增加 “start” 参数以请求后续页的排序结果的替代方法，Solr 支持使用 “Cursor” 扫描结果。
      
  
Solr 中的 Cursor 是一个逻辑概念，不涉及在服务器上缓存任何状态信息。而是使用返回给客户端的最后一个文档的排序值来计算表示排序值的有序空间中的逻辑点的“mark”。这个“mark”可以在随后的请求参数中指定，告诉 Solr 在哪里继续。  

### 使用 Cursor

要在 Solr 中使用 Cursor ，请指定具有 \* 值的 cursorMark 参数。您可以把这 start=0 看作是告诉 Solr “在我的排序结果开始处开始”的一种方法，它也告诉 Solr 您想使用一个 Cursor。
      
  
除了返回前 N 个排序结果（可以使用 rows 参数控制 N ）之外，Solr 响应还将包括一个名为 nextCursorMark 的编码字符串。然后从响应中取 nextCursorMark 字符串值，并将其作为cursorMark 参数传递回 Solr 作为下一个请求。您可以重复这个过程，直到您已经获取尽可能多的文档，或者直到返回的 nextCursorMark 与已指定的 cursorMark 匹配为止，这表示没有更多的结果。  

### 使用 Cursor 时的约束

在 Solr 请求中使用 cursorMark 参数时需要注意一些重要的约束条件：
      
  
1 <li>排序包括基于日期数学的函数，涉及与 NOW 相关的计算将导致混淆的结果，因为每个文档将在每个后续请求中获得新的排序值。这很容易导致永远不会结束的 Cursor，并且不断地返回相同的文档 - 即使文档从不更新。在这种情况下，为所有 Cursor 请求中的 "NOW" 请求参数选择和重用一个固定值。</li>
游标标记值是根据结果中每个文档的排序值计算出来的，这意味着如果多个具有相同排序值的文档中的一个是结果页面上的最后一个文档，则会产生相同的 Cursor 标记值。在这种情况下，使用 cursorMark 的后续请求将不知道具有相同标记值的哪个文档应该被跳过。要求将 uniqueKey 字段作为排序标准中的一个子句使用，可以确保返回一个确定性排序，并且每个 cursorMark值都将标识文档序列中的一个唯一点。  

### Cursor 示例


#### 获取所有文档

此处显示的伪代码显示了使用 Cursor 获取与查询匹配的所有文档时涉及的基本逻辑：  
```
// when fetching all docs, you might as well use a simple id sort
// unless you really need the docs to come back in a specific order
$params = [ q =&gt; $some_query, sort =&gt; 'id asc', rows =&gt; $r, cursorMark =&gt; '*' ]
$done = false
while (not $done) {
  $results = fetch_solr($params)
  // do something with $results
  if ($params[cursorMark] == $results[nextCursorMark]) {
    $done = true
  }
  $params[cursorMark] = $results[nextCursorMark]
}
```

使用 SolrJ，这个伪代码将是：  
```
SolrQuery q = (new SolrQuery(some_query)).setRows(r).setSort(SortClause.asc("id"));
String cursorMark = CursorMarkParams.CURSOR_MARK_START;
boolean done = false;
while (! done) {
  q.set(CursorMarkParams.CURSOR_MARK_PARAM, cursorMark);
  QueryResponse rsp = solrServer.query(q);
  String nextCursorMark = rsp.getNextCursorMark();
  doCustomProcessingOfResults(rsp);
  if (cursorMark.equals(nextCursorMark)) {
    done = true;
  }
  cursorMark = nextCursorMark;
}
```

如果您想用 curl 手工完成，请求的顺序看起来是这样的：  
```
$ curl '...&amp;rows=10&amp;sort=id+asc&amp;cursorMark=*'
{
  "response":{"numFound":32,"start":0,"docs":[
    // ... 10 docs here ...
  ]},
  "nextCursorMark":"AoEjR0JQ"}
$ curl '...&amp;rows=10&amp;sort=id+asc&amp;cursorMark=AoEjR0JQ'
{
  "response":{"numFound":32,"start":0,"docs":[
    // ... 10 more docs here ...
  ]},
  "nextCursorMark":"AoEpVkRCREIxQTE2"}
$ curl '...&amp;rows=10&amp;sort=id+asc&amp;cursorMark=AoEpVkRCREIxQTE2'
{
  "response":{"numFound":32,"start":0,"docs":[
    // ... 10 more docs here ...
  ]},
  "nextCursorMark":"AoEmbWF4dG9y"}
$ curl '...&amp;rows=10&amp;sort=id+asc&amp;cursorMark=AoEmbWF4dG9y'
{
  "response":{"numFound":32,"start":0,"docs":[
    // ... 2 docs here because we've reached the end.
  ]},
  "nextCursorMark":"AoEpdmlld3Nvbmlj"}
$ curl '...&amp;rows=10&amp;sort=id+asc&amp;cursorMark=AoEpdmlld3Nvbmlj'
{
  "response":{"numFound":32,"start":0,"docs":[
    // no more docs here, and note that the nextCursorMark
    // matches the cursorMark param we used
  ]},
  "nextCursorMark":"AoEpdmlld3Nvbmlj"}
```


#### 获取前 N 个文档，基于 Post 处理

由于从 Solr 的角度来看，游标是无状态的，所以一旦您确定有足够的信息，您的客户端代码就可以停止获取额外的结果：  
```
while (! done) {
  q.set(CursorMarkParams.CURSOR_MARK_PARAM, cursorMark);
  QueryResponse rsp = solrServer.query(q);
  String nextCursorMark = rsp.getNextCursorMark();
  boolean hadEnough = doCustomProcessingOfResults(rsp);
  if (hadEnough || cursorMark.equals(nextCursorMark)) {
    done = true;
  }
  cursorMark = nextCursorMark;
}
```


### 索引更新如何影响 Cursor

与基本分页不同，Cursor 分页不依赖于在完成的匹配文档的排序列表中使用绝对“偏移量”。相反，请求中指定的 cursorMark 将根据该文档的绝对排序值封装返回的上一个文档的相对位置信息。这意味着，与基本分页相比，使用 Cursor 时，索引修改的影响要小得多。考虑在讨论基本分页时所描述的相同示例索引：
      
  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">ID</th>
            <th style="text-align: center;">名称</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">1  
            </td>
            <td>
                <p style="text-align: center;">A  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">2  
            </td>
            <td>
                <p style="text-align: center;">B  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">...  
            </td>
            <td style="text-align: center;">...</td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">26  
            </td>
            <td>
                <p style="text-align: center;">Z  
            </td>
        </tr>
    </tbody>
</table>
    - 客户端请求：
```
q=:&amp;rows=5&amp;start=0&amp;sort=name asc, id asc&amp;cursorMark=*
```
        带有 1-5 的 ID 的文档将返回给客户端
              
          
    
    - ID 为 3 的文档被删除
    - 客户端使用前一个响应中的 nextCursorMark 请求5个以上的文档  
文档6-10将被返回 - 删除已经返回的文档不会影响 Cursor 的相对位置
    - 现在添加了ID为90，91以及92的3页新的文档；这三个文档都有一个名称。
    - 客户端使用前一个响应中的 nextCursorMark 请求5个以上的文档  
文档 11-15 将被返回 - 添加已通过排序值的新文档不会影响 Cursor 的相对位置
    - ID 为1的文档更新为将其 “name” 更改为 Q 
    - ID 为17的文档更新为将其 “name” 更改为 A
    - 客户端使用前一个响应中的 nextCursorMark 请求5个以上的文档  
生成的文档以 16、1、18、19、20的顺序排列；  
由于文档1的排序值已更改, 使其位于 Cursor 位置之后, 因此文档将两次返回给客户端；  
由于文档17的排序值已经改变，所以在 Cursor 位置之前，文档已被“跳过”，并且不会因为 Cursor 继续进行而返回给客户端

简而言之：当获取与使用 cursorMark 匹配的查询的所有结果时，索引修改的唯一方式可能导致被跳过的文档或返回两次，如果文档的排序值发生更改。
      
  
确保文档永远不会被返回的一种方法是将 uniqueKey 字段用作主要（因此是唯一有效的）排序标准。
      
  
在这种情况下，您将保证每个文档只返回一次，无论它如何在使用 Cursor 时被修改。  

### “拖放” Cursor

由于 Cursor 请求是无状态的，并且 cursorMark 值封装了从搜索返回的上一个文档的绝对排序值，所以可以“继续”从已经达到其结尾的 Cursor 获取附加结果。如果添加新文档（或更新现有文档）到结果的末尾。
      
  
您可以把它看作类似于在 Unix 中使用 “tail -f” 的东西。如何在索引中添加/更新文档时，如果有 "时间戳" 字段记录，则最常见的示例是如何使用此方法。客户端应用程序可以使用匹配查询的文档的 sort=timestamp asc, id asc 连续轮询 Cursor，并且在添加或更新符合请求条件的文档时总是会收到通知。  
另一个常见的例子是，当您创建新文档时 uniqueKey 值始终增加，并且您可以使用 sort=id asc 连续轮询游标以获得有关新文档的通知。  
拖放 Cursor 的伪代码只是我们早期处理与查询匹配的所有文档的一个小修改：  
```
while (true) {
  $doneForNow = false
  while (not $doneForNow) {
    $results = fetch_solr($params)
    // do something with $results
    if ($params[cursorMark] == $results[nextCursorMark]) {
      $doneForNow = true
    }
    $params[cursorMark] = $results[nextCursorMark]
  }
  sleep($some_configured_delay)
}
```

对于某些特殊情况，/ export 处理程序可以是一个选择。  
