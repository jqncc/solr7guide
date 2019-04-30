## Slave复制 
<div class="content-intro view-box ">
## <span style="font-family: inherit;">Slave</span>复制（Slave Replication）

## <a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#slave-replication"/>
master完全不知道slave。  
  
slave服务器持续不断地轮询master（取决于pollInterval参数）以检查master的当前索引版本。如果slave服务器发现master服务器有更新版本的索引，它将启动复制过程。步骤如下：  
- slave发出一个filelist命令来获取文件列表。此命令返回文件的名称以及一些元数据（例如大小，上次修改的时间戳和别名（如果有的话））。
- 如果slave目录中有本地索引中的任何文件，则它将检查其自身的索引。然后运行filecontent命令来下载缺少的文件。这将使用自定义格式（类似于HTTP块编码）来下载完整内容或每个文件的一部分。如果连接中断，下载将从失败的地方恢复。在任何时候，slave尝试5次，然后才能完全放弃一个复制。
- 这些文件被下载到一个临时目录中，以便在下载过程中如果slave服务器或master服务器崩溃，则不会损坏任何文件。相反，当前的复制将会中止。
- 下载完成后，所有新文件将被移动到实时索引目录，并且文件的时间戳与master服务器上的对应文件相同。
- slave的ReplicationHandler在slave设备上发出提交命令，并加载新的索引。
### 复制配置文件<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#replicating-configuration-files"/>
要复制配置文件，请使用confFiles参数列出它们。只有在 master 的 Solr 实例的conf目录中找到的文件才会被复制。  
  
只有当索引本身被复制时，Solr才会复制配置文件。这意味着即使在master服务器上更改了配置文件，只有在master服务器的索引上有新提交/优化之后，该文件才会被复制。  
与索引文件不同的是，时间戳足以确定它们是否相同，配置文件将与其校验和进行比较。如果它们的校验和是相同的，则认为这些schema.xml文件（在master和slave上）是相同的。  
作为复制配置文件的一个预防措施，Solr将配置文件复制到临时目录，然后将它们移动到conf目录中的最终位置。旧的配置文件被重命名并保存在同一个conf/目录中。ReplicationHandler不会自动清理这些旧文件。  
如果复制涉及至少下载一个配置文件，则ReplicationHandler将发出核心重新加载命令而不是提交命令。  
### 解决slave服务器上的损坏问题<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#resolving-corruption-issues-on-slave-servers"/>
如果将文档添加到slave，则slave不再与其master同步。但是，slave不会采取任何行动使其自身同步，直到master有新的索引数据。  
  
当在master设备上进行提交操作时，master设备的索引版本与slave设备的索引版本不同。然后，slave服务器获取文件列表，发现master服务器上的一些文件也存在于本地索引中，但大小和时间标记不同。这意味着master和slave具有不兼容的索引。  
为了解决这个问题，slave服务器将所有索引文件从master服务器复制到一个新的索引目录，并要求核心从新目录中加载新的索引。  
