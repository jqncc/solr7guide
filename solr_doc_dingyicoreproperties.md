## Solr定义core.properties 
<div class="content-intro view-box ">核心发现意味着创建核心就像core.properties在磁盘上的文件一样简单。
      
  
该core.properties文件是一个简单的Java属性文件，其中每行只是一个key=value对，例如：name=core1。请注意，不需要使用引号。  
最小的core.properties文件看起来像下面的例子。但是，它也可以是空的，请参阅有关core.properties的信息。  
```
name=my_core_name
```


## 安置core.properties<a href="http://lucene.apache.org/solr/guide/7_0/defining-core-properties.html#placement-of-core-properties"/>

Solr核心是通过在solr.home子目录下放置一个名为core.properties的文件来配置的。对树的深度没有先验限制，对于可以定义的内核数量也没有限制。核心可能在树中的任何地方，但核心可能不在现有核心下定义。也就是说，以下内容是不允许的：  
```
./cores/core1/core.properties
./cores/core1/coremore/core5/core.properties
```

在这个例子中，枚举将停在“core1”。  
以下是合法的：  
```
./cores/somecores/core1/core.properties
./cores/somecores/core2/core.properties
./cores/othercores/core3/core.properties
./cores/extracores/deepertree/core4/core.properties
```

可以将Solr分割成多个核心，每个核心都有自己的配置和索引。核心可以专用于单个应用程序或非常不同的应用程序，但所有内容都通过一个通用的管理界面进行管理。您可以即时创建新的Solr核心，关闭核心，甚至可以将一个正在运行的核心替换为另一个核心，而不用停止或重新启动Solr。
      
  
如有必要，您的core.properties文件可以是空的。假设core.properties位于./cores/core1（相对于solr_home），但是是空的。在这种情况下，核心名称被假定为“core1”。instanceDir将是包含core.properties（即，./cores/core1）的文件夹。dataDir将会是../cores/core1/data，等等。  
您可以在不配置任何内核的情况下运行 Solr。  

      
  
<span style="font-family: inherit; font-size: 16px; font-weight: 600;">定义core.properties文件</span>
      
  

## <a href="http://lucene.apache.org/solr/guide/7_0/defining-core-properties.html#defining-core-properties-files"/>

最小的core.properties文件是一个空文件，在这种情况下，所有的属性默认是适当的。  
Java属性文件允许hash（#）或bang（!）字符指定注释到行尾（comment-to-end-of-line）。  
以下属性可用：  
- name  

    
        SolrCore的名称。在使用CoreAdminHandler运行命令时，您将使用此名称来引用SolrCore。  
    
- config  

    
        给定核心的配置文件名称。默认是<code>solrconfig.xml</code>。  
    
- schema  

    
        给定核心的架构文件名称。默认值是<code>schema.xml</code>，但是请注意，如果您使用的是“托管模式”（默认行为），那么此属性的任何值如果与有效值<code>managedSchemaResourceName</code>不匹配，将被读取一次，备份并转换为托管模式使用。有关更多详细信息，请参阅SolrConfig中的架构工厂定义。  
    
- dataDir  

   
        核心的数据目录（存储索引的位置）可以是绝对路径名，也可以是相对于<code>instanceDir</code>值的路径。默认的是<code>data</code>。  
    
- configSet  

    
        如果需要，定义的configset的名称可用于配置内核（请参阅配置集以了解更多详细信息）。  
    
- properties  

  
        此核心的属性文件的名称。该值可以是绝对路径名或相对于<code>instanceDir</code>值的路径。  
    
- transient  

   
        如果为true，如果Solr达到<code>transientCacheSize</code>，则核心可以被卸载。如果未指定，则默认为 false。按照最近最少使用的顺序卸载内核。在SolrCloud模式下不建议将其设置为true。  
    
- loadOnStartup  

    
        如果为true，则默认如果未指定，则在Solr启动时加载核心。在SolrCloud模式下不建议将此设置为false。  
    
- coreNodeName  

    
        仅在SolrCloud中使用，这是承载此副本的节点的唯一标识符。默认情况下<code>coreNodeName</code>会自动生成，但通过显式设置此属性允许您手动分配新的核心来替换现有的副本。例如，通过从具有新主机名或端口的新计算机上进行备份恢复来更换发生硬件故障的计算机时，这可能很有用。  
    
- ulogDir  

    
        此核心（SolrCloud）的更新日志的绝对或相对目录。  
    
- shard  

    
        将此核心分配给（SolrCloud）的分片。  
    
- collection  

  
        这个核心的集合的名称是（SolrCloud）的一部分。  
    
- roles  

    
        未来的SolrCloud参数或用户标记节点以供自己使用的方式。  
    

可以指定其他用户定义的属性作为变量。有关如何定义本地属性的更多信息，请参见替换Solr配置文件中的属性一节。  
