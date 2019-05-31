# Solr配置文件

Solr的配置文件大部是XML格式的，但有关配置设置的API接口是通过JSON进行访问的

## Solr Home

Solr Home即Solr的主目录,**主目录包含重要的配置信息，是Solr存储索引的位置**。  
首次安装Solr时，主目录是：server/solr。但是，某些示例可能会更改此位置（例如，如果您运行：bin/solr start -e cloud，主目录将会是：example/cloud）。  
要注意的是,独立模式和SolrCloud模式主目录的文件结构是不同的。  
以下示例显示了Solr主目录中的关键部分：  

### 单机模式主目录

```
<solr-home-directory>/
   solr.xml
   core_name1/
      core.properties
      conf/
         solrconfig.xml
         managed-schema
      data/
   core_name2/
      core.properties
      conf/
         solrconfig.xml
         managed-schema
      data/
```


### SolrCloud模式主目录

```
<solr-home-directory>/
   solr.xml
   core_name1/
      core.properties
      data/
   core_name2/
      core.properties
      data/
```

您可能会看到其他文件，但您需要了解的主要部分将在下一节中讨论。

## 配置文件

在Solr home中，你会发现这些配置文件：  

- solr.xml：Solr服务器实例配置选项。有关 solr.xml 的更多信息，请参阅：[Solr cores和solr.xml](solr_do_cores.md)。
- Solr core:

  - core.properties：定义每个核心的特定的属性，例如其名称、核心所属的集合、schema的位置以及其他参数。更多详细信息，请参阅定义 core.properties
  - solrconfig.xml：控制高级行为。例如，您可以为数据目录指定一个备用位置。有关solrconfig.xml 的更多信息，请参阅[配置solrconfig.xml](solr_doc_solrconfigxml.md)。
  - managed-schema（或用 schema.xml 替代）描述您将要求Solr索引的文档。Schema将文档定义为字段集合。您可以同时定义字段类型和字段本身。字段类型定义功能强大，包含有关Solr如何处理传入字段值和查询值的信息。有关Solr架构的更多信息，请参阅 [Documents,Fields和Schema设计](solr_doc-ofsstart.md)以及[Schema API](solr_doc_clientapi_overview.md)。
  - data/：包含低级索引文件的目录。

请注意，SolrCloud 示例不包含每个SolrCore的conf目录（所以没有solrconfig.xml或Schema文件）。这是因为通常在 conf 目录中找到的配置文件存储在ZooKeeper中，所以它们可以在群集中传播。  
如果您正在使用 SolrCloud 与嵌入式 ZooKeeper 的情况下，您还可以看到 zoo.cfg 和 zoo.data，它们是 ZooKeeper 的配置和数据文件。但是，如果您正在运行自己的 ZooKeeper 集成，则您在启动 ZooKeeper 配置文件时，将会提供您自己的 ZooKeeper 配置文件，而Solr中的副本将不会被使用。有关SolrCloud 的更多信息，请参阅[SolrCloud部分](solr_doc_cloudstart.md)
