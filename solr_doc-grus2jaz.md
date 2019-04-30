## BACKUP: 备份集合 
<div class="content-intro view-box "><h2>BACKUP  
</h2>
将Solr集合和相关配置备份到共享文件系统 - 例如网络文件系统。
      
  
/admin/collections?action=BACKUP&amp;name=myBackupName&amp;collection=myCollectionName&amp;location=/path/to/my/shared/drive  
BACKUP命令将备份指定集合的​​Solr索引和配置。BACKUP命令从索引的每个分片中获取一个副本。对于配置，它备份与集合和元数据关联的configSet。  

### 备份参数

- collection  
   
        要备份的集合的名称。该参数是必需的。  
    
- location  
    
        共享驱动器上用于写入备份命令的位置。或者，可以将其设置为群集属性。  
    
- async  
    
        请求ID来跟踪这个将被异步处理的动作。  
    
- repository  
   
        要用于备份的存储库的名称。如果没有指定仓库，那么本地文件系统仓库将被自动使用。  

