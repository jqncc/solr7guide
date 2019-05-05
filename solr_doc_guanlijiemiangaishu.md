# Solr管理界面概述

Solr具有一个 Web 界面，它使 Solr管理员和程序员可以轻松查看 Solr配置的详细信息、运行查询和分析文档字段，以便微调（ fine-tune）Solr配置并访问联机文档和其他帮助。  
  
![solr](http://lucene.apache.org/solr/guide/7_0/images/overview-of-the-solr-admin-ui/dashboard.png)  
访问 URL http://hostname:8983/solr/ 将显示 Solr主仪表板，它分为两部分。  
屏幕的左侧是 Solr徽标下的菜单，它通过 UI 的屏幕提供导航。第一组链接用于系统级别的信息和配置，并提供对 日志记录、集合/核心管理和 Java 属性等的访问。在这个信息的末尾至少有一个下拉列表为这个实例配置了 Solr核心。在
    SolrCloud 节点上，附加的下拉列表显示了此群集中的所有集合。单击集合或核心名称将显示指定集合或核心的二级信息菜单，如模式浏览器、配置文件、插件和统计信息以及对索引数据执行查询的功能。  
屏幕的中心显示所选选项的详细信息。这可能包括选项的子导航或所请求的数据的文本或图形表示。有关更多详细信息，请参阅本指南中有关每个屏幕的部分。  
在封面之下，Solr管理用户界面重新使用相同的 HTTP APIs，它可供所有客户访问与 Solr相关的数据来驱动一个外部接口。  

>Tip：上面给出的 Solr管理用户界面的路径是：http://hostname:port/solr，它重定向到当前版本中的路径：http://hostname:port/solr/#/。还支持方便的重定向，因此只需访问管理界面：http://hostname:port/ 也将重定
