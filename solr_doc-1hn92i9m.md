## 在HDFS上运行Solr

Solr支持将其索引和事务日志文件写入和读取到HDFS分布式文件系统。
      
  
这不使用Hadoop MapReduce来处理Solr数据，而只是使用HDFS文件系统进行索引和事务日志文件存储。要使用Hadoop MapReduce处理Solr数据，请参阅Solr contrib区域中的MapReduceIndexerTool。  
要使用HDFS而不是本地文件系统，您必须使用Hadoop 2.x，并且需要指示Solr使用HdfsDirectoryFactory。还有几个要定义的附加参数。这些可以通过以下三种方式之一进行设置：  

    - 将JVM参数传递给bin/solr脚本。每次使用bin/solr启动Solr时都需要传递这些信息。
    - 修改solr.in.sh（或在Windows上为solr.in.cmd），以在使用bin/solr时自动传递JVM参数，而无需手动设置它们。
    - 定义solrconfig.xml中的属性。对于每个集合，都需要重复这些配置更改，因此，如果您只希望将某些集合存储在HDFS中，那么这是一个很好的选择。


## 在HDFS上启动Solr<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#starting-solr-on-hdfs"/>


### 独立的Solr实例<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#standalone-solr-instances"/>

对于独立的Solr实例，在启动Solr之前，应该确保修改一些参数。这些可以在solrconfig.xml中设置（下面的更多内容），或者在启动时传递给bin/solr脚本。
      
  

    - 您需要使用HdfsDirectoryFactory和表单hdfs://host:port/path的一个数据目录
    - 您需要指定表单hdfs://host:port/path的UpdateLog位置
    - 您应该指定一个锁工厂类型'hdfs'或没有。

如果不修改solrconfig.xml，则可以使用以下命令在HDFS上启动Solr：  
```
bin/solr start -Dsolr.directoryFactory=HdfsDirectoryFactory
     -Dsolr.lock.type=hdfs
     -Dsolr.data.dir=hdfs://host:port/path
     -Dsolr.updatelog=hdfs://host:port/path
```

这个例子将以独立模式启动Solr，使用定义的JVM属性（在下面更详细地解释）。  

### SolrCloud实例<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#solrcloud-instances"/>

在SolrCloud模式中，最好将数据和更新日志目录保留为Solr自带的默认值，并简单地指定solr.hdfs.home。所有动态创建的集合将在solr.hdfs.home根目录下自动创建相应的目录。
      
  

    - 在hdfs://host:port/path表单中设置solr.hdfs.home
    - 您应该指定一个锁工厂类型'hdfs'或没有。

```
bin/solr start -c -Dsolr.directoryFactory=HdfsDirectoryFactory
     -Dsolr.lock.type=hdfs
     -Dsolr.hdfs.home=hdfs://host:port/path
```

该命令使用定义的JVM属性以SolrCloud模式启动Solr。  

### 修改solr.in.sh（* nix）或solr.in.cmd（Windows）<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#modifying-solr-in-sh-nix-or-solr-in-cmd-windows"/>

上面的例子假设您每次使用bin/solr启动Solr的时候都会传递JVM参数作为start命令的一部分。但是，bin/solr查找名为solr.in.sh（在Windows上为solr.in.cmd）的包含文件来设置环境变量。默认情况下，该文件位于bin目录中，您可以对其进行修改以永久添加HdfsDirectoryFactory设置，并确保每次启动Solr时都使用它们。
      
  
例如，要将JVM参数设置为在SolrCloud模式下运行时始终使用HDFS（如上所示），则可以添加一个如下所示的部分：  
```
# Set HDFS DirectoryFactory &amp; Settings
-Dsolr.directoryFactory=HdfsDirectoryFactory \
-Dsolr.lock.type=hdfs \
-Dsolr.hdfs.home=hdfs://host:port/path \
```


## 块缓存<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#the-block-cache"/>

为了提高性能，HdfsDirectoryFactory使用一个将缓存HDFS块的目录。这种缓存机制是为了替代Solr所使用的标准文件系统缓存。默认情况下，此缓存分配在堆外。此缓存通常需要相当大，您可能需要提高您正在运行Solr的特定JVM的堆内存限制。对于Oracle / OpenJDK JVM，以下是一个示例命令行参数，您可以使用它在启动Solr时提高限制：  
  
```
-XX:MaxDirectMemorySize=20g
```

## HdfsDirectoryFactory参数<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#hdfsdirectoryfactory-parameters"/>

HdfsDirectoryFactory 有许多设置，它们被定义为 directoryFactory 配置的一部分。  
  

### Solr HDFS设置<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#solr-hdfs-settings"/>


**solr.hdfs.home**

    
        Solr将集合数据写入HDFS中的根位置。不是为数据目录或更新日志目录指定HDFS位置，而是使用此位置来指定一个根位置，并在该HDFS位置中自动创建一切。这个参数的结构是<code>hdfs://host:port/path/solr</code>。  
    


### 块缓存设置<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#block-cache-settings"/>


**solr.hdfs.blockcache.enabled**

    
        启用块缓存。默认是<code>true</code>。  
    
**solr.hdfs.blockcache.read.enabled**

    
        启用读缓存。默认是<code>true</code>。  
    
**solr.hdfs.blockcache.direct.memory.allocation**

    
        启用直接内存分配。如果为<code>false</code>，则使用堆。默认是<code>true</code>。  
    
**solr.hdfs.blockcache.slab.count**

    
        要分配的内存块的数量。每个平板大小为128 MB。默认是<code>1</code>。  
    
**solr.hdfs.blockcache.global**

    
        为所有SolrCores启用/禁用一个全局缓存。使用的设置将来自第一个创建的HdfsDirectoryFactory。默认是<code>true</code>。  
    


### NRTCaching目录设置<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#nrtcachingdirectory-settings"/>


**solr.hdfs.nrtcachingdirectory.enable**

    
        true | 启用NRTCachingDirectory的使用。默认是<code>true</code>。  
    
**solr.hdfs.nrtcachingdirectory.maxmergesizemb**

    
        NRTCachingDirectory最大段大小的合并。默认是<code>16</code>。  
    
**solr.hdfs.nrtcachingdirectory.maxcachedmb**

    
        NRTCachingDirectory最大缓存大小。默认是<code>192</code>。  
    


### HDFS客户端配置设置<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#hdfs-client-configuration-settings"/>


**solr.hdfs.confdir**

    
        传递HDFS客户端配置文件的位置 - 例如HDFS HA所需的位置。  
    


### Kerberos身份验证设置<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#kerberos-authentication-settings"/>

在尝试访问像 HDFS 这样的核心服务时，可以将Hadoop配置使用Kerberos协议来验证用户身份。如果您的HDFS目录使用Kerberos进行保护，那么您需要配置Solr的HdfsDirectoryFactory，以使用Kerberos进行身份验证，以读取和写入HDFS。要从Solr启用Kerberos身份验证，您需要设置以下参数：  
  

**solr.hdfs.security.kerberos.enabled**

    
        设置为<code>true</code>，则表示启用Kerberos身份验证。默认是<code>false</code>。  
    
**solr.hdfs.security.kerberos.keytabfile**

    
        密钥表文件包含Kerberos主体和加密密钥对，当Solr尝试使用安全的Hadoop进行身份验证时，允许进行无密码验证。  
        
            此文件将需要在此参数中提供的相同路径上的所有Solr服务器上存在。  
          
    
**solr.hdfs.security.kerberos.principal**

    
        Solr应使用Kerberos主体进行身份验证以确保Hadoop安全；典型的Kerberos V5主体的格式是：<code>primary/instance@realm</code>。  
    


## 用于HDFS的示例solrconfig.xml<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#example-solrconfig-xml-for-hdfs"/>

以下是solrconfig.xml在HDFS上存储Solr索引的示例配置：  
```
&lt;directoryFactory name="DirectoryFactory" class="solr.HdfsDirectoryFactory"&gt;
  &lt;str name="solr.hdfs.home"&gt;hdfs://host:port/solr&lt;/str&gt;
  &lt;bool name="solr.hdfs.blockcache.enabled"&gt;true&lt;/bool&gt;
  &lt;int name="solr.hdfs.blockcache.slab.count"&gt;1&lt;/int&gt;
  &lt;bool name="solr.hdfs.blockcache.direct.memory.allocation"&gt;true&lt;/bool&gt;
  &lt;int name="solr.hdfs.blockcache.blocksperbank"&gt;16384&lt;/int&gt;
  &lt;bool name="solr.hdfs.blockcache.read.enabled"&gt;true&lt;/bool&gt;
  &lt;bool name="solr.hdfs.nrtcachingdirectory.enable"&gt;true&lt;/bool&gt;
  &lt;int name="solr.hdfs.nrtcachingdirectory.maxmergesizemb"&gt;16&lt;/int&gt;
  &lt;int name="solr.hdfs.nrtcachingdirectory.maxcachedmb"&gt;192&lt;/int&gt;
&lt;/directoryFactory&gt;
```

如果使用Kerberos，则需要将三个与Kerberos相关的属性添加到solrconfig.xml中的&lt;directoryFactory&gt;元素，例如：  
```
&lt;directoryFactory name="DirectoryFactory" class="solr.HdfsDirectoryFactory"&gt;
   ...
  &lt;bool name="solr.hdfs.security.kerberos.enabled"&gt;true&lt;/bool&gt;
  &lt;str name="solr.hdfs.security.kerberos.keytabfile"&gt;/etc/krb5.keytab&lt;/str&gt;
  &lt;str name="solr.hdfs.security.kerberos.principal"&gt;solr/admin@KERBEROS.COM&lt;/str&gt;
&lt;/directoryFactory&gt;
```


## 在SolrCloud中自动添加副本<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#automatically-add-replicas-in-solrcloud"/>

在HDFS中运行Solr的一个好处是，当监督注意到碎片已经停止时，能够自动添加新的副本。由于“gone”的索引分片存储在HDFS中，所以会创建一个新的核心，新的核心将指向HDFS中现有的索引。  
在共享文件系统上使用 autoAddReplicas = true 创建的集合自动添加了启用的副本。以下设置可用于重写 &lt;solrcloud&gt; solr. xml 部分中的默认值。  

**autoReplicaFailoverWorkLoopDelay**

    
        监督团队检查之间的时间（以毫秒为单位）检测并可能采取行动建立替代复制品。默认是<code>10000</code>。  
    
**autoReplicaFailoverWaitAfterExpiration**

    
        在首次发现替换副本之后，等待启动替换副本的最短时间（以毫秒为单位）。这对于在停止或启动群集时防止误报很重要。默认是<code>30000</code>。  
    
**autoReplicaFailoverBadNodeExpiration**

    
        延迟（以毫秒为单位）之后，标记为关闭的副本将被标记为未标记。默认是<code>60000</code>。  
    


### 暂时禁用整个群集的autoAddReplicas<a href="http://lucene.apache.org/solr/guide/7_0/running-solr-on-hdfs.html#temporarily-disable-autoaddreplicas-for-the-entire-cluster"/>

在群集上进行脱机维护, 以及在管理员希望临时禁用自动添加副本的各种其他用例中, 以下 api 将禁用和重新启用群集中所有集合的 autoAddReplicas:  
通过将群集属性 autoAddReplicas 设置为 false, 禁用自动添加副本群集的功能:  
  
  
  
在群集上进行脱机维护时，以及在管理员想暂时禁用自动添加副本的各种其他用例中，以下API将禁用并重新启用群集中所有集合的 autoAddReplicas ：  
通过将群集属性autoAddReplicas设置为false，禁用自动添加副本群集的功能：  
```
http://localhost:8983/solr/admin/collections?action=CLUSTERPROP&amp;name=autoAddReplicas&amp;val=false
```

通过取消 autoAddReplicas 群集属性 (如果未提供 val 参数，则不设置群集属性)，重新启用自动添加副本 (对于那些用 autoAddReplica = true 创建的集合)：  
```
http://localhost:8983/solr/admin/collections?action=CLUSTERPROP&amp;name=autoAddReplicas
```
