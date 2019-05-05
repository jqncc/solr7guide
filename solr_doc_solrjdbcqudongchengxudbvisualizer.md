## Solr JDBC驱动程序-DbVisualizer 
<div class="content-intro view-box ">Solr的JDBC驱动程序支持用于查询Solr的DBVisualizer。  
  
对于[DbVisualizer](https://www.dbvis.com/)，您需要使用DbVisualizer驱动程序管理器为Solr创建一个新的驱动程序。这会将几个SolrJ客户端.jars添加到DbVisualizer类路径中。所需的文件是：  
- 所有在$SOLR_HOME/dist/solrj-lib 中找到的.jars
- 在 $SOLR_HOME/dist/solr-solrj-&lt;version&gt;.jar 中发现的SolrJ .jar
一旦创建了驱动程序，您就可以使用通用部分中概述的连接字符串格式创建一个到Solr的连接，并使用SQL Commander发出查询。  
## 安装驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#setup-driver"/>

### 打开驱动程序管理器<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#open-driver-manager"/>
从工具菜单中选择驱动程序管理器，添加一个驱动程序。  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512543518237493.png)  
### 创建一个新的驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#create-a-new-driver"/>
<p style="text-align: left;">创建新的Solr JDBC驱动程序的步骤如下图所示：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512543582442263.png)  
  
### 在Driver Manager中命名驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#name-the-driver-in-driver-manager"/>
为驱动程序提供一个名称，并提供URL格式：jdbc:solr://&lt;zk_connection_string&gt;/?collection=&lt;collection&gt;。不要填写变量“zk_connection_string”和“collection”的值，稍后在配置Solr连接时会提供这些值。添加驱动程序.jars时，驱动程序类也将自动添加。  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512543692296663.png)  
### 将驱动程序文件添加到类路径<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#add-driver-files-to-classpath"/>
要添加的驱动程序文件是：  
  
- 所有在$SOLR_HOME/dist/solrj-lib 中找到的.jars
- 在 $SOLR_HOME/dist/solr-solrj-&lt;version&gt;.jar 中发现的SolrJ .jar
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512543775762838.png)  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512543893751905.png)  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544080120036.png)  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544042787109.png)  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544155944706.png)  
  
<p style="text-align: center; ">  
  
### 检查并关闭驱动程序管理器<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#review-and-close-driver-manager"/>
一旦添加了驱动程序文件，您可以关闭驱动程序管理器。  
## 创建一个连接<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#create-a-connection"/>
接下来，使用刚刚创建的驱动程序创建一个到Solr的连接。  
### 如何使用连接向导<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#use-the-connection-wizard"/>
<p style="text-align: left;">使用连接向导的操作如下图所示：  
<p style="text-align: center;">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544227225147.png)  
<p style="text-align: center;">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544260574005.png)  
### 命名连接<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#name-the-connection"/>
为您创建的连接命名，如下所示：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544296417049.png)  
### 选择Solr驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#select-the-solr-driver"/>
选择一个合适的Solr JDBC驱动程序，如下：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544391330850.png)  
### 指定Solr URL<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#specify-the-solr-url"/>
使用ZooKeeper主机和端口以及集合来提供Solr URL。例如，jdbc:solr://localhost:9983?collection=test  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544892646688.png)  
  
## 打开并连接到Solr<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#open-and-connect-to-solr"/>
一旦建立了连接，双击它打开连接细节屏幕并连接到Solr。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544512767606.png)  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544547602739.png)  
## 打开SQL命令进入查询<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-dbvisualizer.html#open-sql-commander-to-enter-queries"/>
连接建立后，您可以使用SQL Commander发出查询和查看数据。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544579900416.png)  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512544606142755.png)  
