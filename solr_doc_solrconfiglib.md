# &lt;lib&gt;元素

`<lib>`元素配置指定Solr如何加载插件的依赖jar包（如Analyzer分词器，RequestHandler等）。lib元素的dir属性值是一个相对于当前core根目录的相对路径，regex属性支持正则表达式来模糊配置文件名。

插件是按照它们在solrconfig.xml出现的顺序加载的。如果存在依赖关系，请首先列出最低级别的依赖关系jar。
示例：

```xml
<lib dir="../../../contrib/extraction/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-cell-\d.*\.jar" />
<lib dir="../../../contrib/clustering/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-clustering-\d.*\.jar" />
<lib dir="../../../contrib/langid/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" />
<lib dir="../../../contrib/velocity/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-velocity-\d.*\.jar" />
```
