## Collections API修改集合属性的操作：MODIFYCOLLECTION 
<div class="content-intro view-box ">
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">MODIFYCOLLECTION：修改集合的属性</span>

/admin/collections?action=MODIFYCOLLECTION&amp;collection=&lt;collection-name&gt;&amp;&lt;attribute-name&gt;=&lt;attribute-value&gt;&amp;&lt;another-attribute-name&gt;=&lt;another-value&gt;  
可以一次编辑多个属性。更改这些值只会更新ZooKeeper上的z节点，它们不会更改集合的拓扑。例如，增加replicationFactor不会自动向集合中添加更多的副本，但将让更多的ADDREPLICA命令成功。  

### MODIFYCOLLECTION参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#modifycollection-parameters"/>

- collection  

    
        要修改的集合的名称。该参数是必需的。  
    
- attribute=value  

    
        属性名称和属性值的键值对。至少有一个是必需的。  
        
            可以修改的属性是：  
          
        
            <ul>
                <li>
                    maxShardsPerNode  
                
                - 
                    replicationFactor  
                
                - 
                    autoAddReplicas  
                
                - 
                    collection.configName  
                
                - 
                    rule
                          
                      
                
                - 
                    snitch
                          
                      
                    
                        有关这些属性的详细信息，请参阅上一节的[CREATE操作](https://www.w3cschool.cn/solr_doc/solr_doc-sqwv2ina.html)部分。  
  
  
</li></ul>
