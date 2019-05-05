## Solr自定义更新请求处理器：UIMA集成 
<div class="content-intro view-box ">您可以将 Apache 非结构化信息管理体系结构（[UIMA](https://uima.apache.org/)）与 Solr 进行集成。通过 UIMA，您可以定义分析引擎（Analysis Engines）的自定义管道，以增量方式将元数据作为注释添加到文档中。 
      
  

## Solr 如何配置 UIMA<a href="http://lucene.apache.org/solr/guide/7_0/uima-integration.html#configuring-uima"/>

SolrUIMA UpdateRequestProcessor 是一个自定义的更新请求处理器，它将文档编入索引，将它们发送到 UIMA 管道，然后返回富集了指定元数据的文档。要为 Solr配置 UIMA，请按照下列步骤操作：  
1 <li>VALID_ALCHEMYAPI_KEY 是您的 AlchemyAPI 访问密钥。您需要注册一个 AlchemyAPI 访问秘钥以使用 AlchemyAPI 服务，注册地址如下： http : //www.alchemyapi.com/api/register.html。</li>2 <li>VALID_OPENCALAIS_KEY 是您的 Calais 服务密钥（Calais Service Key）。您需要注册 Calais 服务密钥才能使用 Calais 服务，注册地址如下：http : //www.opencalais.com/apikey。</li>3 <li>analyzeFields 必须包含需要由 UIMA 分析的输入字段。如果 merge=true 那么他们的内容将被合并和分析一次。</li>4 <li>字段映射描述哪些类型的功能应该在字段中进行。</li></li>
    <li>在您的 solrconfig.xml 替换现有的默认 UpdateRequestHandler 或创建一个新的 UpdateRequestHandler：
```
&lt;requestHandler name="/update" class="solr.XmlUpdateRequestHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;uima&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
    </li>
</ol>
一旦您完成了配置，您的文档将在索引时自动丰富指定的字段。
      
  
有关 Solr UIMA 集成的更多信息，请参阅：https://wiki.apache.org/solr/SolrUIMA。  
