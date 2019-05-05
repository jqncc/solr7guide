## Solr JDBC驱动程序：Apache Zeppelin 
<div class="content-intro view-box ">
## Apache Zeppelin
Solr JDBC驱动程序可以支持[Apache Zeppelin](http://zeppelin.apache.org/)。  
  
这需要包含JDBC解释器的Apache Zeppelin 0.6.0版本或更高版本。  
要在Solr中使用 Apache Zeppelin，您需要为Solr创建一个JDBC解释器。这会将SolrJ添加到解释程序类路径中。一旦解释程序被创建，您可以创建一个Notebook来发出查询。在Apache Zeppelin JDBC解释文件提供了有关JDBC前缀和其他功能的附加信息。  

## 创建Apache Solr JDBC解释程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-apache-zeppelin.html#create-the-apache-solr-jdbc-interpreter"/>

创建Apache Zeppelin JDBC解释程序的步骤如下所示：  
1、单击界面顶部导航中的“Interpreter”：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512626114221571.png)
  
2、然后点击“Create”：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512626147940785.png)
  
3、输入关于Solr安装的信息：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512626236140217.png)
  
对于大多数安装, Apache Zeppelin将PostgreSQL配置为JDBC解释程序默认驱动程序。如上面所述, 默认驱动程序可以由 Solr 驱动程序替换, 也可以添加一个单独的JDBC解释程序前缀，如Apache Zeppelin JDBC解释器文档中所述。  

## 创建一个Notebook<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-apache-zeppelin.html#create-a-notebook"/>

如果您要在Apache Zeppelin中新建一个Notebook，请执行下列操作：  
1、点击Notebook，选择创建新笔记：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512627173324121.png)
  
2、提供一个名字并点击“Create Note”  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512627195200200.png)  

## 用Notebook查询<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-apache-zeppelin.html#query-with-the-notebook"/>

对于某些笔记本，默认情况下，JDBC解释程序将不会绑定到笔记本。请参考[将JDBC解释器绑定到Notebook](https://zeppelin.apache.org/docs/latest/interpreter/jdbc.html#bind-to-notebook)的说明。  
以下是Solr查询的结果：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171207/1512627459835831.png)  
下面的代码块假定Apache Solr驱动程序被设置为默认的JDBC解释器驱动程序。如果情况并非如此，则可以获得[使用不同前缀的说明](https://zeppelin.apache.org/docs/latest/interpreter/jdbc.html#how-to-use)。  
```
%jdbc
select fielda, fieldb, from test limit 10
```
