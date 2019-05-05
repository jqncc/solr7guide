## Solr词汇表 
<div class="content-intro view-box ">
## Solr词汇表

本节将介绍Solr的常用术语。  

## Solr术语

在可能的情况下，术语与Solr参考指南的相关部分相关联，以获取更多信息。  
- <b>原型更新（Atomic updates）</b>  

   
        一种仅更新文档的一个或多个字段的方法，而不是重新索引整个文档。  
    

- <b>布尔运算符（Boolean operators）</b>  

        这些控件通过使用AND、OR和NOT等运算符来控制查询中关键字的包含或排除。  
    
- <b>集群（Cluster）</b>  

   
        在Solr中，一个集群是一组Solr节点，通过ZooKeeper彼此协调运行，并作为一个单元进行管理。一个集群可能包含许多集合。参见SolrCloud。  
    
- <b>集合（Collection）</b>  

   
        在Solr中，使用单个配置和Schema将一个或多个文档组合在一个逻辑索引中。  
        
            在SolrCloud中，一个集合可以被分成多个逻辑分片，这些分片又可以分布在多个节点上，或者在单个节点Solr安装中，集合可以是单个Core。  
          
    
- <b>提交（Commit）</b>  

    
        使索引中的文档永久更改。在添加文档的情况下，它们将在提交后进行搜索。  
    
- <b>核心（Core）</b>  

   
        一个单独的Solr实例（表示一个逻辑索引）。多个核心可以在单个节点上运行。另请参见SolrCloud。  
    
- <b>核心重新加载（Core reload）</b>  

    
        在对<code>schema.xml</code>，<code>solrconfig.xml</code>或其他配置文件进行更改后重新初始化 Solr 内核。  
    
- <b>分布式搜索（Distributed search）</b>  

    
        分布式搜索是跨多个Shard处理查询的地方。  
    
- <b>文件（Document）</b>  

    
        一组字段及其值。文档是集合中数据的基本单位。文档被分配给使用标准哈希的分片，或者指定在文档 ID 中指定一个分片。文档在每次写入操作后进行版本控制。  
    
- <b>集成（Ensemble）</b>  

    
        一个ZooKeeper术语，用于指示多个ZooKeeper实例同时运行并相互协调以实现容错。  
    
- <b>小平面（Facet）</b>  

   
        搜索结果根据索引条款的类别安排。  
    
- <b>字段（Field）</b>  

    
        要索引/搜索的内容以及定义Solr如何处理内容的元数据。  
    
- <b>逆文档频率（IDF）</b>  

    
        衡量一个术语的总体重要性。它是按文档总数除以特定单词在集合中出现的文档数来计算的。请参阅：http://en.wikipedia.org/wiki/Tf-idf和Lucene TFIDFSimilarity javadocs，以获取更多有关TF-IDF评分和Lucene评分的信息。另请参见：术语频率。  
    
- <b>倒置索引</b>  

    
        创建可搜索索引的方法是列出每个单词和包含这些单词的文档，类似于书籍后面的索引，其中列出可以找到它们的单词和页面。当执行关键字搜索时，这种方法被认为比替代方法更有效，这将会创建与每个文档中使用的每个单词配对的文档列表。由于用户使用期望在文档中的术语进行搜索，所以在文档之前找到术语节省了处理资源和时间。  
    
- <b>领导（Leader）</b>  

    
        单个副本的每个碎片的是负责在同一分片协调索引更新（文件添加或缺失）到其他副本的。这是通过选举分配给一个节点的临时责任，如果当前碎片Leader（Shard Leader）发生故障，将自动选择一个新的节点代替它。另请参见SolrCloud。  
    
- <b>元数据（Metadata）</b>  

    
        从字面上看，这是表示关于数据的数据。元数据是关于文档的信息，例如其标题、作者或位置。  
    
- <b>自然语言查询（Natural language query）</b>  

    
        以用户身份输入的搜索通常会说或写，如“什么是阿司匹林？”  
    
- <b>节点（Node）</b>  

    
        运行Solr的JVM实例。也被称为Solr服务器。  
    
- <b>开放式并发（Optimistic concurrency）</b>  

    
        也称为“开放式锁定（optimistic locking）”，这是一种允许在保留锁定或版本控制的情况下对当前索引中的文档进行更新的方法。  
    
- <b>监督员（Overseer）</b>  

    
        SolrCloud中的单个节点，负责处理和协调涉及整个集群的操作。它跟踪现有节点、集合、分片和副本的状态，并将新副本分配给节点。这是一个通过选举分配给节点的临时责任，如果当前的监督员关闭了，则一个新的节点将被自动选择代替。另请参见SolrCloud。  
    
- <b>查询解析器（Query parser）</b>  

    
        查询解析器处理用户输入的术语。  
    
- <b>Recall</b>  

    
        搜索引擎检索用户查询的所有可能匹配的能力。  
    
- <b>Relevance</b>  

    
        文档对用户进行搜索的合适性。  
    
- <b>Replica</b>  

   
        一个在SolrCloud 集合中充当碎片的物理副本的核心。  
    
- <b>Replication</b>  

    
        一种将主索引从一台服务器复制到一台或多台“slave”或“child”服务器的方法。  
    
- <b>RequestHandler</b>  

    
        告诉Solr如何处理传入“请求”的逻辑和配置参数，请求是返回搜索结果，索引文档还是处理其他自定义情况。  
    
- <b>SearchComponent</b>  

   
        请求处理程序用来处理查询请求的逻辑和配置参数。搜索组件的例子包括facet，突出显示和“更像这样”的功能。  
    
- <b>Shard</b>  
    
        在SolrCloud中，一个Collection的逻辑分区。每个碎片至少包含一个物理副本，但可能有多个副本分布在多个节点上以实现容错。另请参见SolrCloud。  
    
- <b>SolrCloud</b>  

    
        Solr中一系列功能的术语，它允许管理Solr节点集群以实现可伸缩性、容错性和高可用性。  
    
- <b>Solr架构（managed-schema或schema.xml）</b>  

    
        Solr索引架构定义要编入索引的字段以及字段的类型（文本，整数等），默认情况下，架构数据可以在运行时使用架构API进行 “管理”，并且通常保存在一个名为<code>managed-schema</code>的文件中，Solr 根据需要进行修改，但是可以将一个集合配置为使用静态Schema，该Schema只在启动时从人工编辑的配置文件（通常以named命名）加载<code>schema.xml</code>。有关详细信息，请参阅SolrConfig中的架构工厂定义。  
    
- <b>SolrConfig（solrconfig.xml）</b>  

   
        Apache Solr 配置文件。定义索引选项、RequestHandlers、突出显示、拼写检查和其他各种配置。solrconfig.xml文件位于Solr home conf目录中。  
    
- <b>拼写检查（Spell Check）</b>  

    
        向用户建议搜索条件的替代拼写的能力，作为检查拼写错误的结果，导致很少或零的结果。  
    
- <b>停用词（Stopwords）</b>  

    
        一般而言，对用户的搜索意义不大但可能已经作为自然语言查询的一部分输入的词语。停用词通常是非常小的代词，连词和介词（如“the”，“with”或“and”）  
    
- <b>建议者（Suggester）</b>  

    
        Solr中的功能提供了在用户键入时向用户建议可能的查询条件的能力。  
    
- <b>同义词（Synonyms）</b>  

    
        同义词通常是意义上相互接近的术语，可以互相替代。在搜索引擎实现中，同义词可以是缩写以及单词，或者不是一致的连字符。在这种情况下的同义词的例子是“Inc.”和“Incorporated”或“iPod”和“i-pod”。  
    
- <b>术语频率（Term frequency）</b>  

    
        给定文档中出现单词的次数。请参阅：http://en.wikipedia.org/wiki/Tf-idf 和 Lucene TFIDFSimilarity javadocs 以获取更多有关TF-IDF评分和Lucene评分的信息。另请参阅： 反向文档频率（IDF）。  
    
- <b>事务日志（Transaction log）</b>  

    
        由每个副本维护的只写操作追加日志。这个日志是SolrCloud实现所必需的，由Solr自动创建和管理。  
    
- <b>通配符（Wildcard）</b>  

    
        通配符允许替换单词的一个或多个字母来解释拼写或时态中可能的变化。  
    
- <b>ZooKeeper</b>  

    
        也被称为Apache ZooKeeper。SolrCloud使用的系统跟踪群集的配置文件和节点名称。ZooKeeper集群用作集群的中央配置存储，用于需要分布式同步的操作的协调器以及用于集群拓扑的记录系统。另请参见SolrCloud。  

