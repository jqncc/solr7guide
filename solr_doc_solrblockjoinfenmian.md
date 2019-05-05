## Solr：BlockJoin分面 
<div class="content-intro view-box ">BlockJoin 分面允许您通过父分面汇总子分面数。
      
  
通常的要求是，如果一个父文档有多个子文档，则它们都只需要增加一个 facet 值计数。此功能由 BlockJoinDocSetFacetComponent 提供，并且 BlockJoinFacetComponent 只是一个用于兼容性的别名。   
此组件被认为是实验性的，并且必须为 solrconfig. xml 中的请求处理程序显式启用，与任何其他搜索组件的方式相同。  
下述的示例演示如何将此搜索组件添加到 solrconfig. xml 并在请求处理程序中定义它:  
```
 &lt;searchComponent name="bjqFacetComponent" class="org.apache.solr.search.join.BlockJoinDocSetFacetComponent"/&gt;
   &lt;requestHandler name="/bjqfacet" class="org.apache.solr.handler.component.SearchHandler"&gt;
    &lt;lst name="defaults"&gt;
      &lt;str name="shards.qt"&gt;/bjqfacet&lt;/str&gt;
    &lt;/lst&gt;
    &lt;arr name="last-components"&gt;
      &lt;str&gt;bjqFacetComponent&lt;/str&gt;
    &lt;/arr&gt;
  &lt;/requestHandler&gt;
```

可以将这个组件添加到任何搜索请求处理程序中。此组件在 SolrCloud 模式下使用分布式搜索。
      
  
文档应按索引嵌套子文档中所述添加到子父级块中。示例如下：  
示例文档：  
```
&lt;add&gt;
  &lt;doc&gt;
    &lt;field name="id"&gt;1&lt;/field&gt;
    &lt;field name="type_s"&gt;parent&lt;/field&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;11&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Red&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;XL&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;6&lt;/field&gt;
    &lt;/doc&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;12&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Red&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;XL&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;7&lt;/field&gt;
    &lt;/doc&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;13&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Blue&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;L&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;5&lt;/field&gt;
    &lt;/doc&gt;
  &lt;/doc&gt;
  &lt;doc&gt;
    &lt;field name="id"&gt;2&lt;/field&gt;
    &lt;field name="type_s"&gt;parent&lt;/field&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;21&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Blue&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;XL&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;6&lt;/field&gt;
    &lt;/doc&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;22&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Blue&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;XL&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;7&lt;/field&gt;
    &lt;/doc&gt;
    &lt;doc&gt;
      &lt;field name="id"&gt;23&lt;/field&gt;
      &lt;field name="COLOR_s"&gt;Red&lt;/field&gt;
      &lt;field name="SIZE_s"&gt;L&lt;/field&gt;
      &lt;field name="PRICE_i"&gt;5&lt;/field&gt;
    &lt;/doc&gt;
  &lt;/doc&gt;
&lt;/add&gt;
```

查询的构造方式与父 Block Join 查询的方式相同。例如：  
```
http://localhost:8983/solr/bjqfacet?q={!parent which=type_s:parent}SIZE_s:XL&amp;child.facet.field=COLOR_s
```

因此，我们应该有 Red(1) 和 Blue(1) 的分面，因为子 id=11 和 id=12 上的匹配被聚合成到父级的单一的打击：id=1。  
上面显示的请求的关键组件是：  
- /bjqfacet?  

  
        已经使用块连接分面组件定义的请求处理程序的名称。  
    
- q={!parent which=type_s:parent}SIZE_s:XL  

    
        作为主查询的强制性父查询。父查询也可以是更复杂查询中的从属子句。  
    
- &amp;child.facet.field=COLOR_s  

    
        子文档字段，根据需要可能会有多次重复多个字段。  
    

