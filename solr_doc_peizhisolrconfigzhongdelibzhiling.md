## Solr配置：SolrConfig中的Lib指令 
<div class="content-intro view-box ">Solr允许通过在solrconfig.xml中定义&lt;lib/&gt;指令来加载插件。  
  
插件是按照它们在solrconfig.xml出现的顺序加载的。如果存在依赖关系，请首先列出最低级别的依赖关系jar。  
可以使用正则表达式来提供对同一目录中其他jar的依赖关系的控制加载jar。所有目录都解析为相对于Solr instanceDir。  
```
&lt;lib dir="../../../contrib/extraction/lib" regex=".*\.jar" /&gt;
&lt;lib dir="../../../dist/" regex="solr-cell-\d.*\.jar" /&gt;
&lt;lib dir="../../../contrib/clustering/lib/" regex=".*\.jar" /&gt;
&lt;lib dir="../../../dist/" regex="solr-clustering-\d.*\.jar" /&gt;
&lt;lib dir="../../../contrib/langid/lib/" regex=".*\.jar" /&gt;
&lt;lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" /&gt;
&lt;lib dir="../../../contrib/velocity/lib" regex=".*\.jar" /&gt;
&lt;lib dir="../../../dist/" regex="solr-velocity-\d.*\.jar" /&gt;
```
