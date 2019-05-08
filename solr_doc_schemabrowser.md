# Solr管理界面：Schema Browser

使用Solr架构浏览界面，您可以在Schema Browser窗口中查看Schema数据。  

如果从[分析界面](solr_doc_fenxijiemian.md)访问了此窗口，则它将打开到特定字段、动态字段规则或字段类型。如果未选择任何选项，请使用下拉菜单选择字段或字段类型。
![schema browser](http://lucene.apache.org/solr/guide/7_0/images/schema-browser-screen/schema_browser_terms.png)

Schema Browser界面提供了有关 Schema 中每个特定字段和字段类型的大量有用信息，而且（如果已启用Schema API可以在界面上快速的添加字段或字段类型 。在上面的例子中，我们选择了这个 cat 字段。在主视图窗口的左侧，我们看到字段名称，它被复制到_text_（由于copyField规则），并使用 strings 字段类型。单击这些字段或字段类型名称之一，您可以看到相应的定义。  
在主视图的右侧部分，我们看到 cat 字段定义的具体属性- 通过字段类型显式或隐式地定义，以及填充此字段的文档数量。然后我们看到用于索引和查询处理的分析器。点击其中任何一个的左侧的图标，您将看到所使用的标记化器和/或过滤器的定义。这些过程的输出是在[分析界面](solr_doc_fenxijiemian.md)上测试特定字段的内容处理方式时所看到的信息。

在分析器信息下按钮"Load Term Info",单击该按钮将显示该字段的示例分片中的前N个项，以及显示具有各种频率的Term的数量的直方图。点击Term，将打开[查询界面](solr_doc_chaxunjiemian.md)查看该字段中该Term的查询结果。如果要始终查看某个字段的Term信息，请选择“ 自动加载”，并且在字段有Term时将始终显示。直方图显示了字段中具有给定频率的Term的数量。
