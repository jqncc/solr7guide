# DataDir和DirectoryFactory

Solr在哪里和如何存储其索引是可配置的选项。

## 使用dataDir参数指定索引数据的位置

默认情况下，Solr将其索引数据存储在一个名为/data的目录下中，该目录位于core的实例目录下（instanceDir）。如果您想要指定不同的目录来存储索引数据，则可以在core.properties文件中为core配置dataDir，或使用solrconfig.xml文件中的`<dataDir>`参数。您可以使用绝对路径或相对于SolrCore的instanceDir的路径名指定另一个目录。例如：

```
<dataDir>/solr/data/${solr.core.name}</dataDir>
```

${solr.core.name}为变量，即当前core的名称。  

如果使用主从模式来复制Solr索引（如传统扩展和分发中所述），那么该 `<dataDir>`目录应该对应于主从配置中使用的索引目录。  

>如果定义了环境变量 SOLR_DATA_HOME，或者为DirectoryFactory配置了solr.data.home，或者solr.xml包含一个`<solrDataHome>`元素，则数据目录的位置将是`<SOLR_DATA_HOME>/<instance_name>/data`

## 为索引指定DirectoryFactory

默认solr.StandardDirectoryFactory是基于文件系统的，并且试图为当前的JVM和平台选择最佳实现。您可以通过指定solr.MMapDirectoryFactory、solr.NIOFSDirectoryFactory或solr.SimpleFSDirectoryFactory来强制执行特定的实现或配置选项。  

```xml
<directoryFactory name="DirectoryFactory"
                  class="solr.MMapDirectoryFactory">
  <bool name="preload">true</bool>
</directoryFactory>
```

solr.RAMDirectoryFactory是基于内存的，不是持久性的，并且不适用于主从。使用此DirectoryFactory将您的索引存储在RAM中。

```xml
<directoryFactory class="org.apache.solr.core.RAMDirectoryFactory"/>
```

>如果您正在使用Hadoop并希望将索引存储在HDFS中，那么应该使用solr.HdfsDirectoryFactory，而不是上述任何一种实现。有关更多细节，请参见在HDFS上运行Solr的部分。
