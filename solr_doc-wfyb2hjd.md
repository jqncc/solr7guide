## SolrConfig中的DataDir和DirectoryFactory 
<div class="content-intro view-box ">Solr在哪里和如何存储其索引是可配置的选项。  
  

## 使用dataDir参数指定索引数据的位置<a href="http://lucene.apache.org/solr/guide/7_0/datadir-and-directoryfactory-in-solrconfig.html#specifying-a-location-for-index-data-with-the-datadir-parameter"/>

默认情况下，Solr将其索引数据存储在一个名为/data的目录下中，该目录位于核心的实例目录下（instanceDir）。如果您想要指定不同的目录来存储索引数据，则可以在core.properties文件中为核心配置dataDir，或使用solrconfig.xml文件中的&lt;dataDir&gt;参数。您可以使用绝对路径或相对于SolrCore的instanceDir的路径名指定另一个目录。例如：  
```
&lt;dataDir&gt;/solr/data/${solr.core.name}&lt;/dataDir&gt;
```
所述${solr.core.name}取代将导致当前核心的名称被取代，这导致每个核心的数据被保持在一个单独的子目录中。  
  
如果使用复制来复制Solr索引（如传统扩展和分发中所述），那么该&lt;dataDir&gt;目录应该对应于复制配置中使用的索引目录。  
如果定义了环境变量 SOLR_DATA_HOME，或者为DirectoryFactory配置了solr.data.home，或者solr.xml包含一个&lt;solrDataHome&gt;元素，则数据目录的位置将是&lt;SOLR_DATA_HOME&gt;/&lt;instance_name&gt;/data  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">为索引指定DirectoryFactory</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/datadir-and-directoryfactory-in-solrconfig.html#specifying-the-directoryfactory-for-your-index"/>

默认solr.StandardDirectoryFactory是基于文件系统的，并且试图为当前的JVM和平台选择最好的实现。您可以通过指定solr.MMapDirectoryFactory、solr.NIOFSDirectoryFactory或solr.SimpleFSDirectoryFactory来强制执行特定的实现或配置选项。  
  
```
&lt;directoryFactory name="DirectoryFactory"
                  class="solr.MMapDirectoryFactory"&gt;
  &lt;bool name="preload"&gt;true&lt;/bool&gt;
&lt;/directoryFactory&gt;
```
这solr.RAMDirectoryFactory是基于内存的，不是持久性的，并且不适用于复制。使用此DirectoryFactory将您的索引存储在RAM中。  
```
&lt;directoryFactory class="org.apache.solr.core.RAMDirectoryFactory"/&gt;
```
如果您正在使用Hadoop并希望将索引存储在HDFS中，那么应该使用solr.HdfsDirectoryFactory，而不是上述任何一种实现。有关更多细节，请参见在HDFS上运行Solr的部分。  
