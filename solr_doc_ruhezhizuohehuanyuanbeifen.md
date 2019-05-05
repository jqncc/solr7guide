## Solr如何制作和还原备份 
<div class="content-intro view-box ">如果您担心数据丢失，当然您应该这样做，那么您需要一种备份Solr索引的方法，以便在出现灾难性故障的情况下能够快速恢复。  
  
Solr提供了两种方法来备份和恢复Solr核心或集合，具体取决于您如何运行Solr。如果在SolrCloud模式下运行，则将使用Collections API。如果以独立模式运行Solr，则将使用复制处理程序。  

## SolrCloud备份

Collections API提供了运行SolrCloud时支持备份的功能。这样可以在多个碎片之间生成备份，并将其恢复为与原始分类相同数量的碎片和副本。  
  
有两个命令可用：  

    - action=BACKUP：该命令备份Solr索引和配置。有关更多信息，请参阅“备份集合”一节。
    - action=RESTORE：该命令将恢复Solr索引和配置。“还原收集”部分提供了更多信息。

## 独立模式备份

备份和恢复使用Solr的复制处理程序。Solr包含对复制的隐式支持，因此可以使用此API。但是，可以通过在solrconfig.xml中定义自己的复制处理程序来自定义复制处理程序的配置。有关配置复制处理程序的详细信息，请参阅“配置ReplicationHandler”一节。  
  

### Backup API

该backup API需要发送一个命令到/replication处理程序来备份系统。  
  
您可以使用这样的HTTP命令来触发备份（用正在使用的核心名称替换“getting start”）：  
backup API示例如下：  
```
http://localhost:8983/solr/gettingstarted/replication?command=backup
```
该backup命令是一个异步调用，它将表示来自最新索引提交点的数据。所有的索引和搜索操作将像往常一样继续对索引执行。  
  
在任何时间点，只能对一个核心进行一次备份调用。当正在进行的备份操作正在发生时，随后的恢复调用将引发异常。  
备份请求还可以采用以下附加参数：  

**location**
    
        将创建备份的路径。如果路径不是绝对的，则备份路径将与Solr的实例目录相关。|name |The snapshot将在名为<code>snapshot.&lt;name&gt;</code>的目录中创建。如果没有指定名称，则目录名称将具有以下格式：<code>snapshot.&lt;yyyyMMddHHmmssSSS&gt;</code>。  
**numberToKeep**
    
        要保留的备份数量。如果<code>maxNumberOfBackups</code>已在<code>solrconfig.xml</code>中的复制处理程序指定，则始终使用<code>maxNumberOfBackups</code>并尝试使用<code>numberToKeep</code>将导致错误。另外，如果指定了备份名称，则不会考虑此参数。有关<code>maxNumberOfBackups</code>的详细信息，请参阅配置ReplicationHandler一节。  
**repository**
    
        用于备份的存储库的名称。如果没有指定仓库，那么本地文件系统仓库将被自动使用。  
    
**commitName**
    
        使用CREATESNAPSHOT命令拍摄快照时使用的提交的名称。  


### Backup Status
可以监视备份操作, 以便通过将details命令发送到/replication处理程序来查看它是否已完成，如以下示例所示:  
Status API示例如下所示：  
```
http://localhost:8983/solr/gettingstarted/replication?command=details
```
输出代码片段：  
```
&lt;lst name="backup"&gt;
  &lt;str name="startTime"&gt;Sun Apr 12 16:22:50 DAVT 2015&lt;/str&gt;
  &lt;int name="fileCount"&gt;10&lt;/int&gt;
  &lt;str name="status"&gt;success&lt;/str&gt;
  &lt;str name="snapshotCompletedAt"&gt;Sun Apr 12 16:22:50 DAVT 2015&lt;/str&gt;
  &lt;str name="snapshotName"&gt;my_backup&lt;/str&gt;
&lt;/lst&gt;
```
如果失败了，则会在响应中发送 snapShootException。  

### Restore API

还原备份需要将restore命令发送到/replication处理程序，然后是要还原的备份的名称。  
  
您可以使用如下命令从备份中还原：  
用法示例：  
```
http://localhost:8983/solr/gettingstarted/replication?command=restore&amp;name=backup_name
```
这会将指定的索引快照还原到当前的核心。还原完成后，搜索将开始反映快照数据。  
该restore请求可以采取这些附加参数：  

**location**
    
        备份快照文件的位置。如果未指定，它将在Solr的数据目录中查找备份。  
    
**name**
    
        要还原的备份索引快照的名称。如果没有提供该名称，则<code>snapshot.&lt;timestamp&gt;</code>在位置目录中查找带有格式的备份。它在这种情况下选择最新的时间戳备份。  
**repository**
    
        用于备份的存储库的名称。如果没有指定仓库，那么本地文件系统仓库将被自动使用。  
    

该restore命令是一个异步调用。一旦恢复完成，反映的数据将是还原的备份索引。  
在一个时间点上只能对一个核心进行一次restore调用。当正在进行的还原操作正在发生时，随后的还原调用将引发异常。  

### Restore Status API

您还可以通过将restorestatus命令发送到/replication处理程序来检查restore操作的状态，如下例所示：  
  
Status API 示例：  
```
http://localhost:8983/solr/gettingstarted/replication?command=restorestatus
```
Status API输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="restorestatus"&gt;
    &lt;str name="snapshotName"&gt;snapshot.&lt;name&gt;&lt;/str&gt;
    &lt;str name="status"&gt;success&lt;/str&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
状态值可以是“In Progress”，“success”或“failed”。如果失败了，那么也会在响应中发送一个“exception”。  

### <span style="font-family: inherit;">Create Snapshot </span>API

快照（snapshot）功能与备份（backup）功能不同，因为索引文件不会复制到任何位置。索引文件在同一个索引目录中被快照，并且可以在进行备份时被引用。  
  
您可以用这样的HTTP命令触发一个快照命令（将 "techproducts" 替换为您正在使用的核心名称）：  
Create Snapshot API示例：  
```
http://localhost:8983/solr/admin/cores?action=CREATESNAPSHOT&amp;core=techproducts&amp;commitName=commit1
```
所述CREATESNAPSHOT请求参数是：  

**commitName**
    
        将快照存储为的名称。  
    
**core**
    
        要在其上执行快照的核心的名称。  
    
**async**
    
        请求ID来跟踪这个将被异步处理的操作。  
    


### <span style="font-family: inherit;">List Snapshot </span>API

该LISTSNAPSHOTS命令列出了特定核心的所有拍摄快照。  
  
您可以使用像这样的 HTTP 命令触发列表快照命令（将 "techproducts" 替换为您正在使用的核心的名称）：  
List Snapshot API示例：  
```
http://localhost:8983/solr/admin/cores?action=LISTSNAPSHOTS&amp;core=techproducts&amp;commitName=commit1
```
列表快照请求参数是：  

**core**
    
        要列出其快照的核心的名称。  
    
**async**
    
        请求ID来跟踪这个将被异步处理的动作。  
    


### <span style="font-family: inherit;">Delete Snapshot </span>API

该DELETESNAPSHOT命令删除特定核心的快照。  
  
您可以像这样使用HTTP命令来触发删除快照（将“techproducts”替换为正在使用的核心的名称）：  
Delete Snapshot API示例：  
```
http://localhost:8983/solr/admin/cores?action=DELETESNAPSHOT&amp;core=techproducts&amp;commitName=commit1
```
删除快照请求参数是：  

**commitName**
    
        指定要删除的提交名称  
    
**core**
    
        我们要删除快照的核心的名称  
    
**async**
    
        请求ID来跟踪这个将被异步处理的操作  
    


## 备份/还原存储库

Solr提供了接口来插入不同的存储系统进行备份和还原。例如，您可以在本地文件系统（如EXT3）上运行Solr群集，但可以将索引备份到HDFS文件系统，反之亦然。  
  
存储库接口需要在solr.xml文件中配置。在运行备份/恢复（backup/restore ）命令时，我们可以指定要使用的存储库。  
如果未配置任何存储库，则将自动使用本地文件系统存储库。  
示例solr.xml部分用于配置类似HDFS的存储库：  
```
&lt;backup&gt;
  &lt;repository name="hdfs" class="org.apache.solr.core.backup.repository.HdfsBackupRepository" default="false"&gt;
    &lt;str name="location"&gt;${solr.hdfs.default.backup.path}&lt;/str&gt;
    &lt;str name="solr.hdfs.home"&gt;${solr.hdfs.home:}&lt;/str&gt;
    &lt;str name="solr.hdfs.confdir"&gt;${solr.hdfs.confdir:}&lt;/str&gt;
  &lt;/repository&gt;
&lt;/backup&gt;
```
