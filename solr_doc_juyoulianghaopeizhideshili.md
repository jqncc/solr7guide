# 具有良好配置的Solr实例

本节将告诉您如何优化您的Solr实例以获得最佳性能。  
  
本节涵盖了以下主题：  
配置solrconfig.xml：描述如何使用Solr的主配置文件：solrconfig.xml，它覆盖了文件的主要部分。  
Solr Cores和solr.xml：描述如何使用solr.xml和core.properties配置Solr核心，或配置在单个实例中的多个Solr核心。  
配置API：描述用于配置Solr: Blob Store，Config，Request Parameters 和Managed Resources的几个API。  
隐式RequestHandlers：描述Solr自动提供的各种端点以及如何配置它们。  
Solr插件：介绍Solr插件的指针来获取更多信息。  
JVM设置：提供有关使用Java Virtual Machines（Java虚拟机）的最佳实践的一些指导。  
本节的重点通常是配置单个Solr实例，但是对于那些有兴趣在集群环境中扩展Solr实现的人来说，请参阅SolrCloud部分。还可以通过分片或复制来进行扩展，这是在Legacy Scaling and Distribution部分中描述的。  
