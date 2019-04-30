## Solr JDBC驱动程序-SQuirreL SQL 
<div class="content-intro view-box ">对于[SQuirreL SQL](http://squirrel-sql.sourceforge.net/)，您需要为Solr创建一个新的驱动程序。这会将几个SolrJ客户端.jars添加到SQuirreL SQL类路径中。所需的文件是：  
  

    - 所有在$SOLR_HOME/dist/solrj-lib 中找到的.jars
    - 在 $SOLR_HOME/dist/solr-solrj-&lt;version&gt;.jar 中发现的SolrJ .jar

一旦驱动程序被创建，您就可以使用通用部分中概述的连接字符串格式创建一个到Solr的连接，并使用编辑器发出查询。  

## 添加Solr JDBC驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#add-solr-jdbc-driver"/>

### 打开SQuirreL SQL驱动程序

### <a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#open-drivers"/>

打开SQuirreL SQL驱动程序的操作如下图所示：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545335450687.png)  
  

### 添加驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#add-driver"/>

添加SQuirreL SQL驱动程序的界面如下：  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545401818991.png)  

### 命名驱动程序<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#name-the-driver"/>

为驱动程序提供一个名称，并提供URL格式：jdbc:solr://&lt;zk_connection_string&gt;/?collection=&lt;collection&gt;。不要填写变量“zk_connection_string”和“ collection”的值，稍后在配置与Solr的连接时定义这些值。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545440668381.png)
  

### 将Solr JDBC jar添加到Classpath<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#add-solr-jdbc-jars-to-classpath"/>

<p style="text-align: center;">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545466851137.png)
  
<p style="text-align: center;">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545495550396.png)
  
<p style="text-align: center;">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545545299721.png)
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545593876842.png)  
  

### 添加Solr JDBC驱动程序类名称<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#add-the-solr-jdbc-driver-class-name"/>

添加.jars之后，您将需要额外定义类名称：org.apache.solr.client.solrj.io.sql.DriverImpl。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545622826391.png)
  

## 创建一个别名<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#create-an-alias"/>

要定义JDBC连接，您必须定义一个别名。  

### 打开别名<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#open-aliases"/>

<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545654715789.png)  

### 添加别名<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#add-an-alias"/>

<p style="text-align: left;">以下是添加别名的操作界面：  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545675600548.png)  

### 配置别名<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#configure-the-alias"/>

<p style="text-align: left;">配置别名的界面如下：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545708983957.png)
  

### 连接到别名<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#connect-to-the-alias"/>

<p style="text-align: left;">连接到别名的界面如下：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545734316654.png)
  

## 查询<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-squirrel-sql.html#querying"/>

一旦成功连接到Solr，就可以使用SQL接口输入查询并处理数据。  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171206/1512545763202431.png)  
