## LIST：列表集合 
<div class="content-intro view-box ">获取集群中集合的名称。  
/admin/collections?action=LIST  

### 使用LIST的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-list"/>

在本次示例中输入：  
```
http://localhost:8983/solr/admin/collections?action=LIST
```
得到的输出为：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":2011},
  "collections":["collection1",
    "example1",
    "example2"]}
```
