## Solr空间查询 
<div class="content-intro view-box ">Solr 具有复杂的地理空间支持，包括在给定位置（或边界框内）的指定距离范围内搜索，按距离排序，甚至可以通过距离来提高结果。  
  
我们在练习1中编入索引的一些示例 techproducts 文档具有与它们相关联的位置以说明空间功能。要重新索引这些数据，请参阅[练习1](https://www.w3cschool.cn/solr_doc/solr_doc-2gbo2fsg.html)。  
空间查询可以与任何其他类型的查询结合使用，例如在旧金山10公里范围内查询 “ipod” 的例子：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171122/1511333735241241.png)  
这是来自 Solr 的示例搜索 UI（调用 /browse），它具有很好的功能来显示每个项目的映射，并允许轻松选择附近的搜索位置。您可以通过查看浏览器中的：http://localhost:8983/solr/techproducts/browse?q=ipod&amp;pt=37.7752%2C-122.4232&amp;d=10&amp;sfield=store&amp;fq=%7B%21bbox%7D&amp;queryOpts=spatial&amp;queryOpts=spatial  
  
要了解有关 Solr 的空间功能的更多信息，请参阅空间搜索部分。  

## 结束教程

如果您已经在本 Solr 快速入门指南中运行完整的命令集，则已经完成了以下操作：  

    - 将 Solr 推出到 SolrCloud 模式，两个节点，两个集合，包括分片（shards）和副本（replicas）
    - 索引了几种类型的文件
    - 使用模式 API 来修改您的模式
    - 打开管理控制台，使用其查询界面获取结果
    - 打开 /browse 界面，在更友好和熟悉的界面中浏览 Solr 的功能

## 重置

在您完成本教程的过程中，您可能需要停止 Solr 并将环境重置为起点。以下命令行将停止 Solr 并删除在[练习1](https://www.w3cschool.cn/solr_doc/solr_doc-2gbo2fsg.html)中完全创建的两个节点中的每个节点的目录：  
```
bin/solr stop -all ; rm -Rf example/cloud
```
