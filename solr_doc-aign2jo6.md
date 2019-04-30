## 用于ReplicationHandler的HTTP API命令 
<div class="content-intro view-box ">
## 用于ReplicationHandler的HTTP API命令<a href="http://lucene.apache.org/solr/guide/7_0/index-replication.html#http-api-commands-for-the-replicationhandler"/>

您可以使用下面的HTTP命令来控制ReplicationHandler的操作。  
- enablereplication  
    
        为所有slave在“master”上启用复制。  
```
http://_master_host:port_/solr/_core_name_/replication?command=enablereplication
```
    
- disablereplication  
    
        在master设备上禁用所有slave设备的复制。  
```
http://_master_host:port_/solr/_core_name_/replication?command=disablereplication
```
    
- indexversion  
    
        返回指定master设备或slave设备上最新的可复制索引版本。  
```
http://_host:port_/solr/_core_name_/replication?command=indexversion
```
    
- fetchindex  
    
        强制指定的slave服务器从其master服务器获取索引副本。  
```
http://_slave_host:port_/solr/_core_name_/replication?command=fetchindex
```
        
            如果你愿意的话，你可以传递一个额外的属性，比如<code>masterUrl</code>或者 <code>compression</code>（或者在<code>&lt;lst name="slave"&gt;</code>标签中指定的任何其他参数）来从master进行一次复制。这消除了在slave设备中对master设备进行硬编码的需要。  
          
    
- abortfetch  
   
        中止从master复制索引到指定的slave。  
```
http://_slave_host:port_/solr/_core_name_/replication?command=abortfetch
```
    
- enablepoll  
    
        启用指定的slave轮询master上的更改。  
```
http://_slave_host:port_/solr/_core_name_/replication?command=enablepoll
```
    
- disablepoll  
    
        禁止指定的slave轮询master上的更改。  
```
http://_slave_host:port_/solr/_core_name_/replication?command=disablepoll
```
    
- details  
    
        检索配置细节和当前状态。  
```
http://_slave_host:port_/solr/_core_name_/replication?command=details
```
    
- filelist  
    
        检索存在于指定主机索引中的Lucene文件列表。  
```
http://_host:port_/solr/_core_name_/replication?command=filelist&amp;generation=&lt;_generation-number_&gt;
```
        
            通过运行<code>indexversion</code>命令可以发现索引的生成编号。  
          
    
- backup  
    
        如果服务器中有提交的索引数据，则在master服务器上创建备份，否则，什么都不做。  
```
http://_master_host:port_/solr/_core_name_/replication?command=backup
```
        
            该命令对定期备份非常有用。有几个支持的请求参数：  
          
        
            <ul>
                <li>
                    <code>numberToKeep:</code>：除非在处理程序上指定了<code>maxNumberOfBackups</code>初始化参数，否则这可以与备份命令一起使用 - 在这种情况下<font color="#c7254e" face="Consolas, Courier New, Courier, monospace"><span style="white-space: nowrap; background-color: rgb(249, 242, 244);">，</span></font>总是使用在<code>maxNumberOfBackups</code>初始化参数，并尝试使用<code>numberToKeep</code>请求参数将导致错误。  
                
                - 
                    <code>name</code>:(可选）备份名称。快照将在核心的数据目录内调用的<code>snapshot.&lt;name&gt;</code>目录中创建。默认情况下，名称是使用日期<code>yyyyMMddHHmmssSSS</code>格式生成的。如果传递了<code>location</code>参数，将使用它来代替数据目录。  
                
                - 
                    <code>location</code>：备份位置。  
                
            
          
    </li><li>deletebackup  
    
        删除使用该<code>backup</code>命令创建的任何备份。  
```
http://_master_host:port_ /solr/_core_name_/replication?command=deletebackup
```
        
            有两个支持的参数：  
          
        
            
                - 
                    <code>name</code>：快照的名称。具有<code>snapshot.<em>name</em></code>名称的快照必须存在。如果没有，则会引发错误。  
                
                - 
                    <code>location</code>：创建快照的位置。  
                
            
          
    </li>
</ul>
