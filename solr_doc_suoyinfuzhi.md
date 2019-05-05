## Solr索引复制 
<div class="content-intro view-box ">
## 索引复制

索引复制将主索引的完整副本分布给一个或多个从属服务器。主服务器继续管理对索引的更新。所有查询都由从属处理。通过这种分工，Solr可以进行扩展，以针对大型搜索量提供足够的查询响应。
      
  
下图显示了使用索引复制的Solr配置。主服务器的索引被复制到从属服务器上。  
<p style="text-align: center; ">
    ![solr](https://7n.w3cschool.cn/attachments/image/20180115/1515988084245820.png)
  
提示：Solr索引可以跨多个从服务器复制，然后处理请求。  

## Solr中的索引复制<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#index-replication-in-solr"/>

Solr包含一个在 HTTP 上工作的索引复制的 Java 实现：
      
  

    - 影响复制的配置由单个文件（solrconfig.xml）控制
    - 支持配置文件以及索引文件的复制
    - 在具有相同配置的平台上工作
          
    
    - 不依赖于操作系统相关的文件系统功能（例如：硬链接）
    - 与Solr紧密集成；一个管理页面可以对复制的每个方面进行细粒度的控制
    - 基于Java的复制功能被实现为请求处理程序。因此，配置复制类似于任何普通的请求处理程序。

### SolrCloud 中的复制

虽然在 SolrCloud 群集中没有 "主/从" 节点的明确概念，但在该页上讨论的 ReplicationHandler 仍然被 SolrCloud 用于支持 "碎片恢复"，这是以对等方式进行的。  
使用 SolrCloud 时，ReplicationHandler 必须通过<code>/replication</code>路径可用。Solr 会隐式执行此操作，除非在您的 solrconfig. xml 中显式重写，但如果您希望重写默认行为，请确定您没有显式设置下面提到的任何 "master" 或 "slave" 配置选项，否则它们将干扰正常的 SolrCloud 操作。  

## 复制术语<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#replication-terminology"/>

以下内容定义了与Solr复制关联的关键术语。  

  
- Index  
    
        Lucene索引是文件的目录。这些文件构成了Solr核心的可搜索和可返回数据。  
    
- Distribution  
    
        将索引从主服务器复制到所有从服务器。分布过程利用了Lucene的索引文件结构。  
    
- Inserts 和 Deletes  
   
        当索引中发生插入和删除时，目录保持不变。文档总是被插入到新创建的文件中。被删除的文件不会从文件中删除。它们被标记在文件中，可删除，并且不会从文件中删除，直到索引被优化。  
    
- Master 和 Slave  
    
        Solr复制主机是一个单独的节点，它最初接收所有的更新并保持所有的组织。Solr复制从节点不直接接收更新，而是针对单个主节点进行所有更改（如插入，更新，删除等）。主服务器上所做的更改将分发给所有从客户端处理所有查询请求的从服务器节点。  
    
- Update  
        更新是针对单个Solr实例的单个更改请求。这可能是删除文档、添加新文档、更改文档、删除与查询匹配的所有文档等的请求。更新在单个Solr实例中同步处理。  
    
- Optimization  
    
        压缩索引并合并段以提高查询性能的过程。优化只能在主节点上运行。相比于在许多更新中已经变得碎片化的索引，经优化的索引可以提供查询性能增益。分配一个优化的索引需要比将新的分段分配给未优化的索引要长得多的时间。  
    
- Segments  

        索引的一个自包含子集，包含与这些文档中的术语倒排索引相关的一些文档和数据结构。  
    
- mergeFactor  
        一个控制索引中段数的参数。例如，当mergeFactor设置为3时，Solr将用文档填充一个段，直到满足限制maxBufferedDocs，然后它将开始一个新的段。当达到由mergeFactor指定的段数（在这个例子中为3）时，Solr将把所有段合并成单个索引文件，然后开始将新文档写入新段。  
    
- Snapshot  
        包含指向索引数据文件的硬链接的目录。Snapshot从主节点分发，当从服务器拉取它们时，“智能复制”从节点在快照目录中没有包含到最新索引数据文件的硬链接的任何段。  

