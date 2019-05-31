# core.properties

core.properties文件是一个简单的Java属性文件，solr.xml、solrconfig.xml、data-config.xml都可以通过从core.properties中获取参数。core.properties与这些配置文件放在同级目录下即可，如果要自定义它的加载路径可以通过-Dsystem.properties=confi\solrcore.properties系统参数来自定义。

## Core自动发现机制

当Solr启动时,会在SOLR_HOME目录下递归查找名称为core.properties的配置文件,然后根据core.properties中的配置加载core,一般一个core对应一个core.properties文件.

## core.properties位置

Solr core是通过在solr.home子目录下放置一个名为core.properties的文件来配置的。对目录的深度没有限制，对于可以定义的core数量也没有限制。core可能在目录树中的任何目录下，但已经有core定义的目录下不能再定义新的core。也就是说，以下内容是不允许的：

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

可以将Solr分割成多个core，每个core都有自己的配置和索引。core可以专用于单个应用程序或非常不同的应用程序，但所有内容都通过一个通用的管理界面进行管理。您可以即时创建新的Solr core，关闭core，甚至可以将一个正在运行的core替换为另一个core，而不用停止或重新启动Solr。

core.properties文件可以是空的。假设core.properties位于./cores/core1（相对于solr_home），但是是空的。在这种情况下，核心名称被假定为“core1”。instanceDir将是包含core.properties（即，./cores/core1）的文件夹。dataDir将会是../cores/core1/data，等等。
>您可以在不配置任何内核的情况下运行Solr。


## 定义core.properties文件

最小的core.properties文件是一个空文件，在这种情况下，所有的属性都是默认值。  
Java属性文件允许hash（#）或bang（!）字符指定注释到行尾（comment-to-end-of-line）。 以下属性可用：

- name  
SolrCore的名称。在使用CoreAdminHandler运行命令时，您将使用此名称来引用SolrCore。  

- config  
给定核心的配置文件名称。默认是solrconfig.xml。  

- schema  
给定核心的架构文件名称。默认值是schema.xml，但是请注意，如果您使用的是“托管模式”（默认行为），那么此属性的任何值如果与有效值managedSchemaResourceName不匹配，将被读取一次，备份并转换为托管模式使用。有关更多详细信息，请参阅SolrConfig中的架构工厂定义。  
    
- dataDir  
核心的数据目录（存储索引的位置）可以是绝对路径名，也可以是相对于instanceDir值的路径。默认的是data。  

- configSet  
如果需要，定义的configset的名称可用于配置内核（请参阅配置集以了解更多详细信息）。  

- properties  
此核心的属性文件的名称。该值可以是绝对路径名或相对于instanceDir值的路径。  

- transient  
如果为true，如果Solr达到transientCacheSize，则核心可以被卸载。如果未指定，则默认为 false。按照最近最少使用的顺序卸载内核。在SolrCloud模式下不建议将其设置为true。  

- loadOnStartup  
如果为true，则默认如果未指定，则在Solr启动时加载核心。在SolrCloud模式下不建议将此设置为false。  

- coreNodeName  
仅在SolrCloud中使用，这是承载此副本的节点的唯一标识符。默认情况下coreNodeName会自动生成，但通过显式设置此属性允许您手动分配新的核心来替换现有的副本。例如，通过从具有新主机名或端口的新计算机上进行备份恢复来更换发生硬件故障的计算机时，这可能很有用。  

- ulogDir  
此核心（SolrCloud）的更新日志的绝对或相对目录。  

- shard  
将此核心分配给（SolrCloud）的分片。  

- collection  
这个核心的集合的名称是（SolrCloud）的一部分。  

- roles  
未来的SolrCloud参数或用户标记节点以供自己使用的方式。  

可以指定其他用户定义的属性作为变量。有关如何定义本地属性的更多信息，请参见替换Solr配置文件中的属性一节。  

