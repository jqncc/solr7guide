## Solr创建或修改集合的别名：CREATEALIAS 
<div class="content-intro view-box ">该CREATEALIAS操作将创建一个指向一个或多个集合的新别名。如果已经存在一个同名的别名，那么这个操作将会取代现有的别名，这就像是一个原子的“MOVE”命令。
      
  
/admin/collections?action=CREATEALIAS&amp;name=name&amp;collections=collectionlist  

### CREATEALIAS参数<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#createalias-parameters"/>

- name  

    
        要创建的别名。该参数是必需的。  
    
- collections  

   
        逗号分隔的收藏列表将被别名。集合必须已经存在于集群中。该参数是必需的。  
    
- async  

   
        请求ID来跟踪这个将被异步处理的动作。  
    


### CREATEALIAS响应<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#createalias-response"/>

输出将只是一个responseHeader，其中包含处理请求所花费的时间的详细信息。要确认别名的创建，您可以查看Solr Admin UI中Cloud部分下的aliases.json文件。  

### 使用CREATEALIAS的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-createalias"/>

输入如下：  
创建一个名为“testalias”的别名，并将其链接到名为“anotherCollection”和“testCollection”的集合：  
```
http://localhost:8983/solr/admin/collections?action=CREATEALIAS&amp;name=testalias&amp;collections=anotherCollection,testCollection
```

得到的输出是：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;122&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
