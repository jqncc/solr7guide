## CoreAdmin API 
<div class="content-intro view-box ">您在运行SolrCloud集群时，核心Admin API主要在Collections API的封面下使用。
      
  
SolrCloud用户通常不应该直接使用CoreAdmin API，但是 API 对于节点或主从 Solr 安装的核心维护操作的用户可能有用。  
CoreAdmin API由CoreAdminHandler实现，CoreAdminHandler是一个用于管理Solr核心的特殊用途请求处理程序。与其他请求处理程序不同，CoreAdminHandler未附加到单个核心。相反，在每个Solr节点中都有一个CoreAdminHandler实例，用于管理在该节点中运行的所有核心，并且可以通过该/solr/admin/cores路径进行访问。  
CoreAdmin操作可以通过HTTP请求来执行，这些请求指定一个action请求参数，其他特定于操作的参数作为附加参数提供。  
所有的操作名称都是大写的，并在下面的部分中进行了深入的定义。  

## STATUS操作

该STATUS操作将返回所有正在运行的Solr内核的状态，或仅返回指定内核的状态：  

```
admin/cores?action=STATUS&amp;core=core-name
```


### STATUS参数

- core  

        核心的名称，如<code>solr.xml</code>中<code>&lt;core&gt;</code>元素的“name”属性中所列。  
- indexInfo  

    
        如果<code>false</code>，则有关索引的信息将不会返回一个核心状态请求。在具有大量内核（即，超过数百个）的Solr实现中，检索每个内核的索引信息可能花费大量时间，并且并不总是需要的。默认是<code>true</code>。  


## CREATE操作

CREATE操作将创建一个新的核心并进行注册。  
如果具有给定名称的Solr核心已经存在，则在新核心初始化时它将继续处理请求。当新的核心准备就绪时，将会有新的请求，旧核心将被卸载：  

```
admin/cores?action=CREATE&amp;name=core-name&amp;instanceDir=path/to/dir&amp;config=solrconfig.xml&amp;dataDir=data
```
请注意，此命令是唯一不支持该core参数的Core Admin API命令。相反，该name参数是必需的，如下文所示。  
```
注意事项：CREATE必须能够找到一个配置！CREATE调用必须能够找到配置，否则它将无法成功。当您在运行 SolrCloud 并为集合创建新内核时，将从集合继承配置。每个集合链接到一个存储在ZooKeeper中的configName。这满足配置要求。不过，有一点值得注意--如果您正在运行 SolrCloud，则根本不应该使用 CoreAdmin API。使用集合 API。
当您没有运行 SolrCloud 时，如果定义了配置集，则可以使用 configSet 参数，如下所述。如果没有配置集，则在 CREATE 调用中指定的 instanceDir 必须已经存在，并且它必须包含一个conf目录，该目录又必须包含solrconfig.xml，您的模式（通常命名为managed-schema或schema.xml）和任何由这些配置引用的文件。
配置和架构文件名可以用config和schema参数指定, 但这些都是专家选项。要避免创建配置目录，您可以做的一件事是使用指向绝对路径的config和schema参数，但是除非你完全理解你在做什么，否则会导致配置混乱。
```
```
Note：CREATE 和core.properties文件：
core.properties文件是作为CREATE命令的一部分生成的。如果您自己在核心目录中创建了一个core.properties文件，然后尝试使用CREATE添加到 Solr 的核心，您会得到一个错误告诉你，另一个核心已经定义在那里。在使用 CREATE 命令调用 CoreAdmin API 之前，不能存在核心属性文件。
```

### <span style="font-family: inherit;">CREATE </span>Core 参数


**name**

    
        新核心的名称。同<code>&lt;core&gt;</code>元素上的<code>name</code>。该参数是必需的。  
**instanceDir**

    
        该内核的文件应该存储的目录。同<code>instanceDir</code>在上<code>&lt;core&gt;</code>元素。缺省值是为<code>name</code>参数指定的值（如果未提供）。  
    
**config**

    
        配置文件的名称（例如，<code>solrconfig.xml</code>）相对于<code>instanceDir</code>。  
    
**schema**

    
        用于核心的架构文件的名称。请注意，如果您使用的是“托管架构”（默认行为），那么此属性的任何值与有效值都不匹配，<code>managedSchemaResourceName</code>将被读取一次，备份并转换为托管架构使用。有关详细信息，请参阅SolrConfig中的架构工厂定义。  
    
**dataDir**

    
        数据目录的名称，相对于<code>instanceDir</code>。  
**configSet**

    
        用于此内核的configset的名称。有关更多信息，请参阅配置集。  
    
**collection**

    
        此核心所属的集合的名称。默认是核心的名称。<code>collection.<em>param</em>=<em>value</em></code>导致在创建新集合时要设置的<code><em>param</em>=<em>value</em></code>的属性。使用<code>collection.configName=<em>config-name</em></code>指向新集合的配置。  
<table>
                <tbody>
                    <tr>
                        <td>
                        </td>
                        <td>虽然可以为不存在的集合创建核心，但不支持此方法，也不推荐。直接为其创建核心之前，始终使用Collections API创建一个集合。</td></tr></tbody></table>
**shard**

    
        这个核心代表的分片ID。通常您想自动分配一个分片ID。  
    
**property.name=value**

    
        将核心属性name设置为value。请参阅定义core.properties文件内容的部分。  
    
**async**

    
        请求ID来跟踪这个将被异步处理的动作。  
    

使用collection.configName=configname指向配置为一个新的集合。  

### CREATE示例

```
http://localhost:8983/solr/admin/cores?action=CREATE&amp;name=my_core&amp;collection=my_collection&amp;shard=shard2
```


## RELOAD操作
RELOAD操作从现有已注册的Solr内核的配置中加载一个新的内核。在新的核心正在初始化时，现有核心将继续处理请求。当新的Solr内核准备好时，它接管并卸载旧的内核。  

```
admin/cores?action=RELOAD&amp;core=core-name
```
当您对磁盘上的Solr内核配置进行更改（如添加新的字段定义）时，这非常有用。调用RELOAD操作可以应用新的配置，而无需重新启动Solr。  
RELOAD操作执行 SolrCore 的 "实时" 重装，重新使用一些现有的对象。某些配置选项 (如 solrconfig 中的 dataDir 位置和 IndexWriter 相关设置) 不能通过简单的重新加载操作进行更改和激活。  

### RELOAD Core 参数


**core**

    
        内核的名称，如<code>solr.xml</code>中<code>&lt;core&gt;</code>元素的“name”属性中所列。该参数是必需的。  



## RENAME操作

该RENAME操作将更改Solr内核的名称。  

```
admin/cores?action=RENAME&amp;core=core-name&amp;other=other-core-name
```

### RENAME参数


**core**

    
        要重命名的Solr核心的名称。该参数是必需的。  
    
**other**

    
        Solr核心的新名称。如果<code>&lt;solr&gt;</code>的persistent属性为<code>true</code>，则新名称将作为<code>&lt;core&gt;</code>属性的<code>name</code>属性被写入<code>solr.xml</code>。该参数是必需的。  
**async**

    
        请求ID来跟踪这个将被异步处理的动作。  
    


## SWAP操作

SWAP自动交换用于访问两个现有Solr核心的名称。这可以用来将新内容交换到生产中。先前的核心仍然可用，如有必要，可以交换回来。交换之后，每个核心将被另一个核心的名称所知。  

```
admin/cores?action=SWAP&amp;core=core-name&amp;other=other-core-name
```
注意：不要将 <span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); font-family: Consolas, &quot;Courier New&quot;, Courier, monospace; white-space: nowrap;">SWAP </span><span style="white-space: nowrap;">与 SolrCloud 节点一起使用。它不受支持，可能会导致核心无法使用。</span>  

### SWAP参数


**core**

    
        要交换的核心之一的名称。该参数是必需的。  
    
**other**

    
        要交换的核心之一的名称。该参数是必需的。  
    
**async**

    
        请求ID来跟踪这个将被异步处理的动作。  
    


## UNLOAD操作

这个UNLOAD操作从Solr中删除了一个核心。活动请求将继续处理，但不会有新的请求发送到指定的内核。如果核心以多个名称注册，则只会删除给定的名称。  

```
admin/cores?action=UNLOAD&amp;core=core-name
```
该UNLOAD操作需要一个参数（core）来标识要删除的核心。如果&lt;solr&gt;的 persistent 属性设置为true，则具有此name属性的&lt;core&gt;元素将从solr.xml中移除。  
注意：卸载SolrCloud集合中的所有核心会导致从ZooKeeper中删除该集合的元数据。  
### UNLOAD<span style="font-family: inherit; font-weight: 600;">参数</span>



**core**

    
        要删除的核心的名称。该参数是必需的。  
    
**deleteIndex**

    
        如果为<code>true</code>，则卸载核心时将删除索引。默认是<code>false</code>。  
    
**deleteDataDir**

    
        如果为<code>true</code>，则删除<code>data</code>目录和所有的子目录。默认是<code>false</code>。  
    
**deleteInstanceDir**

    
        如果为<code>true</code>，则删除与核心相关的所有内容，包括索引目录，配置文件和其他相关文件。默认是<code>false</code>。  
    
**async**

    
        请求ID来跟踪这个将被异步处理的动作。  
    


## MERGEINDEXES 操作
该MERGEINDEXES操作将一个或多个索引合并到另一个索引。索引必须已经完成提交，并且应该锁定写入，直到合并完成或者生成的合并索引可能会损坏。目标核心索引必须已经存在，并且具有与将被合并到其中的一个或多个索引的兼容模式。合并完成后，还应执行目标核心上的另一个提交：  

```
admin/cores?action=MERGEINDEXES&amp;core=new-core-name&amp;indexDir=path/to/core1/data/index&amp;indexDir=path/to/core2/data/index
```
在这个例子中，我们使用indexDir参数来定义源核的索引位置。该core参数定义了目标索引。这种方法的好处是，我们可以合并任何可能与Solr核心无关的基于Lucene的索引。  
或者，我们可以使用一个srcCore参数，如下例所示：  

```
admin/cores?action=mergeindexes&amp;core=new-core-name&amp;srcCore=core1-name&amp;srcCore=core2-name
```
这种方法允许我们定义可能没有与目标内核位于同一物理服务器上的索引路径的内核。但是，我们只能使用Solr核心作为源索引。这种方法的另一个好处是，如果写入与源索引并行发生，那么我们没有那么高的风险。  
  
我们可以通过指定async参数并传递一个 request-id 来异步运行此调用。然后可以使用此ID来使用REQUESTSTATUS API检查已提交的任务的状态。  

### MERGEINDEXES参数


**core**

    
        目标核心/索引的名称。该参数是必需的。  
    
**indexDir**

    
        多值的，将被合并的目录。  
    
**srcCore**

    
        将被合并的多值源Core。  
    
**async**

    
        请求ID来跟踪这个将被异步处理的动作  
    


## SPLIT操作

该SPLIT操作将索引分成两个或更多个索引。被拆分的索引可以继续处理请求。拆分片段可以放在服务器文件系统的指定目录中，也可以合并为正在运行的Solr核心。  
该SPLIT操作支持五个参数，如下表所述。  

### SPLIT参数


**core**

    
        要拆分的核心的名称。该参数是必需的。  
    
**path**

    
        一个多值参数，索引将被写入的目录路径。这个参数或者<code>targetCore</code>必须被指定。如果指定了这个参数，则不能使用该<code>targetCore</code>参数。  
**targetCore**

    
        一个多值参数，一个索引将被合并到的目标Solr核心。这个参数或者<code>path</code>必须被指定。如果指定了这个参数，则不能使用该<code>path</code>参数。  
    
**ranges**

    以逗号分隔的十六进制格式的哈希范围列表。如果使用这个参数，则不应该使用<code>split.key</code>。有关如何使用此参数的示例，请参阅下面的SPLIT示例。  
**split.key**

    
        用于分割索引的关键。如果使用这个参数，则不该使用<code>ranges</code>。有关如何使用此参数的示例，请参阅下面的SPLIT示例。  
    
**async**

    
        请求ID来跟踪这个将被异步处理的动作。  
    


### SPLIT示例

该core索引将被分成尽可能多的与path或targetCore参数的数量相同的部分。  
  

#### 两个targetCore参数的用法：

```
http://localhost:8983/solr/admin/cores?action=SPLIT&amp;core=core0&amp;targetCore=core1&amp;targetCore=core2
```

这里core索引将被分成两部分并合并成两个targetCore索引。  

#### 使用两个路径参数：

```
http://localhost:8983/solr/admin/cores?action=SPLIT&amp;core=core0&amp;path=/path/to/index/1&amp;path=/path/to/index/2
```

该core索引将被分成两个部分，并写入到指定的两个目录路径。  

#### 与split.key参数一起使用：

```
http://localhost:8983/solr/admin/cores?action=SPLIT&amp;core=core0&amp;targetCore=core1&amp;split.key=A!
```

这里所有的文件都有与split.key“A！” 相同的路由密钥。将从core索引中拆分并写入targetCore。  

#### 使用范围参数：

```
http://localhost:8983/solr/admin/cores?action=SPLIT&amp;core=core0&amp;targetCore=core1&amp;targetCore=core2&amp;targetCore=core3&amp;ranges=0-1f4,1f5-3e8,3e9-5dc
```
本示例使用在十六进制中指定的哈希范围0-500、501-1000 和1001-1500 的ranges参数。这里索引将被分成三部分，每个targetCore接收匹配指定哈希范围的文件，即core1将得到哈希范围为0-500的文档，core2将接收哈希范围为501-1000的文档，最后，core3将接收带有哈希范围1001-1500的文档。至少要指定一个哈希范围。请注意，使用与路由密钥哈希范围相同的单个哈希范围不等同于使用该split.key参数，因为多个路由密钥可以散列到相同的范围。  
  
在targetCore必须已经存在，并且必须具有与core索引兼容的架构。在分割之前，core索引会自动调用一个 commit。  
该命令用作SPLITSHARD命令的一部分，但也可用于 non-cloud Solr内核。当针对没有split.key参数的 non-cloud 核心使用时，此操作将拆分源索引并交替分发其文档，以便每个拆分文件包含相同数量的文档。如果指定了split.key参数，那么只有具有相同路由密钥的文档才会从源索引中分离出来。  

## REQUESTSTATUS 操作

请求已提交的异步CoreAdmin API调用的状态。  
```
admin/cores?action=REQUESTSTATUS&amp;requestid=id
```

### 核心REQUESTSTATUS参数

REQUESTSTATUS命令只有一个参数。  

**requestid**

    
        用户为异步请求定义了request-id。该参数是必需的。  
    

下面的调用将返回已经提交的异步CoreAdmin调用的状态。  
```
http://localhost:8983/solr/admin/cores?action=REQUESTSTATUS&amp;requestid=1
```


## REQUESTRECOVERY 操作

该REQUESTRECOVERY操作手动要求核心通过与领导者同步来恢复。这应该被认为是一个“专家级”的命令，应该用于节点（SorlCloud副本）无法自动激活的情况。  
```
admin/cores?action=REQUESTRECOVERY&amp;core=core-name
```

### REQUESTRECOVERY参数


**core**

    
        要重新同步的核心的名称。该参数是必需的。  
    


### REQUESTRECOVERY示例

```
http://localhost:8981/solr/admin/cores?action=REQUESTRECOVERY&amp;core=gettingstarted_shard1_replica1
```

指定的核心可以通过管理UI扩展适当的ZooKeeper节点来找到。  
