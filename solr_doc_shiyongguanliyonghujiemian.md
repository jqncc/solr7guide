# 使用Solr管理用户界面

本章节讨论 Solr 管理用户界面（“管理 UI”）。  
Solr 的管理用户界面的概述解释了用户界面的基本功能，什么是初始管理UI页面，以及如何在接口上配置。另外，还有一些页面描述了管理功能的每个界面：  

- [获取帮助](solr_doc_gettingssistance.md)：向您展示如何获取有关 UI 的更多信息。
- [Solr日志记录](solr_doc_rizhijilu.md)：显示由此 Solr 节点记录的最新消息，并提供了一种更改特定类的日志记录级别的方法。
- [Cloud界面](solr_doc_cloudscreen.md)以 SolrCloud 模式运行时，Cloud界面显示有关节点的信息。
- [Collections/Core管理界面](solr_doc_collcoreadmin.md) Collections/Core管理基本功能。
- [Java Properties界面](solr_doc_javashuxingjiemian.md)：显示有关每个核心的JVM信息。
- [线程转储](solr_doc_thread_dump.md)：查看有关每个线程的状态和其他详细信息。
- 特定于集合的工具：是解释每个集合可用的其他功能界面的部分。 
  - 分析（Analysis）- 让您分析在特定字段中找到的数据。
  - 导入（Dataimport）- 显示有关数据导入处理程序的当前状态的信息。
  - 文档（Documents）- 提供了一个简单的表单，允许您直接从浏览器执行各种 Solr 索引命令。
  - 文件（Files）- 显示当前的核心配置文件，如 solrconfig.xml。
  - 查询（Query ）- 让您提交关于核心的各种元素的结构化查询。
  - 流（Stream）- 允许您提交流表达式并查看结果和解析解释。
  - Schema浏览器（Schema Browser）- 在浏览器窗口中显示架构数据。
- 特定于核心的工具：是说明每个指定核心可用的额外屏幕的部分。
  - Ping - 让你 ping 一个已命名的核心，并确定核心是否处于活动状态。
  - 插件/统计（Plugins/Stats）- 显示插件和其他已安装的组件的统计信息。
  - 复制（Replication）- 显示核心的当前复制状态，并允许您启用/禁用复制。
  - 细分信息（Segments Info）- 提供底层 Lucene 索引段的可视化。
