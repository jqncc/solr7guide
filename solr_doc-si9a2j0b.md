## Solr输出列表中的所有别名：LISTALIASES 
<div class="content-intro view-box ">/admin/collections?action=LISTALIASES  
LISTALIASES操作不带任何参数。  

## 列表响应

### <a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#listaliases-response"/>

输出将包含具有相应集合名称的别名列表。  

### 使用LISTALIASES的例子<a href="http://lucene.apache.org/solr/guide/7_0/collections-api.html#examples-using-listaliases"/>

使用LISTALIASES操作将会有如下输出：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;0&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="aliases"&gt;
    &lt;str name="testalias1"&gt;collection1&lt;/str&gt;
    &lt;str name="testalias2"&gt;collection2&lt;/str&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
