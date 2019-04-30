## 编解码器：Codec Factory 
<div class="content-intro view-box ">可以在solrconfig.xml中指定一个codecFactory来确定将索引写入磁盘时使用哪个Lucene Codec。  
  
如果未指定，则Lucene的默认编解码器将被隐式使用。  

## 替代默认的编解码器

Lucene的默认编解码器有两种选择，如下所述：  

### solr.SchemaCodecFactory

该solr.SchemaCodecFactory支持以下2个主要特点：  
  

    - 基于架构的每个字段类型配置docValuesFormat和postingsFormat - 请参阅“字段类型属性”部分以获取更多详细信息。
    - 一个compressionMode选项：BEST_SPEED （默认）针对搜索速度性能进行了优化；BEST_COMPRESSION 针对磁盘空间使用情况进行了优化

如下示例：  
```
&lt;codecFactory class="solr.SchemaCodecFactory"&gt;
  &lt;str name="compressionMode"&gt;BEST_COMPRESSION&lt;/str&gt;
&lt;/codecFactory&gt;
```

### solr.SimpleTextCodecFactory

Lucene的这个工厂SimpleTextCodecFactory生成一个纯文本可读的索引格式。  
  
这个编解码器不应该在生产中使用。<code>SimpleTextCodec</code>相对较慢，占用大量的磁盘空间。它的使用应该仅限于教育和调试目的  
示例：  
```
&lt;codecFactory class="solr.SimpleTextCodecFactory"/&gt;
```
