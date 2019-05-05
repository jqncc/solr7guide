## Solr特定于集合的工具 
<div class="content-intro view-box ">在 Solr 的左侧导航栏中，您将看到一个名为“集合选择器”的下拉菜单，可用于访问集合特定的管理界面。  
  
只有使用 SolrCloud 时才可见：“集合选择器”（Collection Selector）下拉菜单仅适用于以 SolrCloud 模式运行的 Solr 实例。Solr 的单节点或主/从复制实例将不会显示此菜单，而是在 Core Selector（核心选择器）下拉菜单中提供本节中介绍的 Collection 专用 UI 页面。  
  
单击“集合选择器”下拉菜单将显示 Solr 集群中的集合列表，其中包含可用于按名称查找特定集合的搜索框。当您从下拉列表中选择一个集合时，该页面的主要显示将显示关于该集合的一些基本元数据，而在左侧导航栏中将显示一个二级菜单项，其中包含指向其他集合特定管理屏幕的链接。  
  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510210833916471.png)  
下面列出了特定于集合的 UI 界面，并链接到该指南的部分以了解更多信息：  

    - [分析界面](https://www.w3cschool.cn/solr_doc/solr_doc-tc482fwu.html) - 让您分析在特定字段中发现的数据。
    - Dataimport - 向您显示有关数据导入处理程序当前状态的信息。
    - [文档界面](https://www.w3cschool.cn/solr_doc/solr_doc-mco12fwz.html) - 提供了一个简单的表单，允许您直接从浏览器执行各种 Solr 索引命令。
    - 文件 - 显示当前的核心配置文件，如 solrconfig.xml。
    - [查询界面](https://www.w3cschool.cn/solr_doc/solr_doc-kcas2fx9.html) - 让您提交关于核心的各种元素的结构化查询。
    - 流 - 允许您提交流表达式并查看结果和解析解释。
    - 模式浏览器 - 在浏览器窗口中显示模式数据。
