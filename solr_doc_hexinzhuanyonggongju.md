## Solr核心专用工具 
<div class="content-intro view-box ">Solr 特定于核心的工具是一组 UI 界面，允许您查看核心级别的信息。  
  
在左侧的导航栏中，您将看到一个名为“核心选择器（Core Selector）”的下拉菜单。单击该菜单将显示此 Solr 节点上托管的 Solr 核心列表，其中包含可用于按名称查找特定核心的搜索框。  
当您从下拉列表中选择一个核心时，页面的主要显示将显示关于核心的一些基本元数据，而在左侧导航栏中将出现一个二级菜单，其中包含指向其他核心特定管理屏幕的链接。  
您还可以定义一个名为 admin-extra.html 的配置文件，其中包含您希望在此主界面的 “Admin Extra” 部分中显示的链接或其他信息。  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510216957612427.png)  
下面列出了特定于核心的 UI 界面，并提供了指向本指南部分的链接以了解更多内容：  

    - [Ping](https://www.w3cschool.cn/solr_doc/solr_doc-l6u92fxe.html) - 让您 ping 一个已命名的核心，并确定核心是否处于活动状态。
    - 插件/统计（Plugins/Stats） - 显示插件和其他已安装组件的统计信息。  

    - 复制（Replication） - 显示核心的当前复制状态，并允许您启用/禁用复制。
    - 段信息（Segments Info） - 提供底层 Lucene 索引段的可视化。

如果您正在运行 Solr 的单个节点实例，则通常在每个集合基础上显示的其他 UI 界面也将被列出：  

    - 分析（Analysis） - 让您分析在特定字段中找到的数据。  

    - 导入（Dataimport） - 显示有关数据导入处理程序的当前状态的信息。  

    - 文档（Documents） - 提供了一个简单的表单，允许您直接从浏览器执行各种 Solr 索引命令。  

    - 文件（Files） - 显示当前的核心配置文件，如：solrconfig.xml。  

    - 查询（Query） - 让您提交关于核心的各种元素的结构化查询。  

    - 流（Stream） - 允许您提交流表达式并查看结果和解析解释。  

    - 模式浏览器（Schema Browser） - 在浏览器窗口中显示架构数据。
