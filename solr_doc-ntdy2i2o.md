## Solr插件 
<div class="content-intro view-box ">Solr允许您加载自定义代码，以执行Solr中的各种任务，从自定义请求处理程序处理您的搜索，对您的文本字段的自定义分析器和令牌筛选器。您甚至可以加载自定义字段类型。这些自定义代码被称为插件。  
不是每个人都需要为他们的Solr实例创建插件 - 通常为大多数应用程序提供了足够的东西。但是，如果您需要某些东西，则您可能希望在 SolrPlugins 上查看有关插件的 Solr Wiki 文档。  
如果您想要使用插件，并且您在SolrCloud模式下运行，则可以使用 Blob Store API 和 Config API 将 jar 加载到 Solr。在 SolrCloud 模式下添加自定义插件的部分介绍了要使用的命令。  
