## Solr添加副本：ADDREPLICA 
<div class="content-intro view-box ">将副本添加到集合中的分片。如果副本将在特定节点中创建，则可以指定节点名称。
      
  
/admin/collections?action=ADDREPLICA&amp;collection=collection&amp;shard=shard&amp;node=nodeName  

### ADDREPLICA参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#addreplica-parameters"/>

- collection  

   
        应该创建副本的集合的名称。该参数是必需的。  
    
- shard  

    
        要添加副本的分片的名称。  
        
            如果没有指定<code>shard</code>，那么一定是<code>_route_</code>。  
          
    
- _route_  

    
        如果确切的分片名称是未知的，则用户可以传递该<code>_route_</code>值，并且系统将识别分片的名称。  
        
            如果<code>shard</code>参数也被指定，则忽略该参数。  
          
    
- node  

    
        应该创建副本的节点的名称。  
    
- instanceDir  

    
        将被创建的核心的instanceDir。  
    
- dataDir  

    
        应在其中创建核心的目录。  
    
- type  

    
        要创建的副本的类型。以下这些可能的值是允许的：  
        
            
                
                    <ul>
                        <li>
                            <code>nrt</code>：NRT类型维护事务日志并在本地更新其索引。这是默认和最常用的。  
                        
                        - 
                            <code>tlog</code>：TLOG类型维护事务日志，但只通过复制更新其索引。  
                        
                        - 
                            <code>pull</code>：PULL类型不维护事务日志，只通过复制更新其索引。这种类型没有资格成为领导者。  
                        
                    
                  
              
          
        
            有关副本类型选项的更多信息，请参阅副本类型一节。  
          
    </li><li>property.name=value  

    
        将核心属性name设置为value。有关支持的属性和值的详细信息，请参阅定义core.properties。  
    </li><li>async  

   
        请求ID来跟踪这个将被异步处理的动作
            <a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#asynchronous-calls"/>
          
    </li>
</ul>

### 使用ADDREPLICA的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-addreplica"/>

在本次实例中，输入如下：  
```
http://localhost:8983/solr/admin/collections?action=ADDREPLICA&amp;collection=test2&amp;shard=shard2&amp;node=192.167.1.2:8983_solr
```

将得到的输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;3764&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="success"&gt;
    &lt;lst&gt;
      &lt;lst name="responseHeader"&gt;
        &lt;int name="status"&gt;0&lt;/int&gt;
        &lt;int name="QTime"&gt;3450&lt;/int&gt;
      &lt;/lst&gt;
      &lt;str name="core"&gt;test2_shard2_replica4&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
