# Collections / Core 管理

"Collections"界面提供了一些管理集合的基本功能，由Collections API提供支持。

>如果您正在运行单个节点Solr实例，则不会在Admin UI的左侧导航菜单中看到“集合”选项。
>您将看到一个“Core Admin”屏幕，它通过CoreAdmin API支持一些类似的Core级别信息和操作。

此页面的主显示提供了群集中存在的集合列表。单击集合名称可提供有关如何定义集合及其当前分片和副本的一些基本元数据，以及用于添加和删除单个副本的选项。

通过界面顶部的按钮，您可以对群集进行各种集合级别更改，从添加新集合或别名到重新加载或删除单个集合。

![solr collection-admin](http://lucene.apache.org/solr/guide/7_0/images/collections-core-admin/collection-admin.png)
单击副本名称旁边的红色“X”可以删除副本。

如果分片处于非活动状态，例如在SPLITSHARD操作之后，删除分片的选项将在分片名称旁边显示为红色“X”。

![solr DeleteShard](http://lucene.apache.org/solr/guide/7_0/images/collections-core-admin/DeleteShard.png)