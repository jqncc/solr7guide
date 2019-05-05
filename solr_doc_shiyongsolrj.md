## 使用SolrJ 
<div class="content-intro view-box ">
## 使用SolrJ
SolrJ是一个使Java应用程序可以轻松与Solr对话的API。SolrJ隐藏了许多连接到Solr的细节，并允许您的应用程序通过简单的高级方法与Solr进行交互。  
  
SolrJ的中心是org.apache.solr.client.solrj包，它只包含五个主要的类。首先创建一个SolrClient代表你想要使用的Solr实例。然后发送SolrRequests或SolrQuerys找回SolrResponses。  
SolrClient是抽象的，所以要连接到远程Solr的情况下，您将实际创建一个 HttpSolrClient 或 CloudSolrClient 的实例。两者都通过HTTP与Solr进行通信，不同之处在于HttpSolrClient使用明确的Solr URL进行配置，而CloudSolrClient是使用 SolrCloud 群集的 zkHost 字符串配置的。  
单节点Solr客户端：  
```
String urlString = "http://localhost:8983/solr/techproducts";
SolrClient solr = new HttpSolrClient.Builder(urlString).build();
```
SolrCloud客户端：  
```
// Using a ZK Host String
String zkHostString = "zkServerA:2181,zkServerB:2181,zkServerC:2181/solr";
SolrClient solr = new CloudSolrClient.Builder().withZkHost(zkHostString).build();
// Using already running Solr nodes
SolrClient solr = new CloudSolrClient.Builder().withSolrUrl("http://localhost:8983/solr").build();
```
一旦你有一个SolrClient，你可以通过调用类似的方法使用它，例如query()，add()以及commit()。  
## 构建和运行SolrJ应用程序
SolrJ API包含在Solr中，所以您不必下载或安装其他任何东西。但是，为了构建和运行使用SolrJ的应用程序，必须将一些库添加到类路径中。  
  
在构建时，本节提供的示例要求在类路径中使用solr-solrj-x.y.z.jar。  
在运行时，本节中的示例需要在“dist / solrj-lib”目录中找到的库。  
与这些部分的示例捆绑在一起的Ant脚本在构建和运行时适当地包含这些库。  
你可以通过使用Maven而不是Ant来避开大量的JAR文件。在应用程序中包含 SolrJ 所需做的所有工作是将以下依赖项放在项目的 pom. xml 中：  
```
&lt;dependency&gt;
  &lt;groupId&gt;org.apache.solr&lt;/groupId&gt;
  &lt;artifactId&gt;solr-solrj&lt;/artifactId&gt;
  &lt;version&gt;x.y.z&lt;/version&gt;
&lt;/dependency&gt;
```
如果担心SolrJ库扩展客户端应用程序的大小，则可以使用ProGuard之类的代码混淆器来删除不使用的API。  
## 指定Solr基本URL
大多数SolrClient实现（例外情况：CloudSolrClient）要求用户指定一个或多个Solr基本URL，然后客户端将其用于向Solr发送HTTP请求。用户在其提供的基本URL上包含的路径会影响从此时创建的客户端的行为。  
  
1 <li>指向特定核心或集合的路径（例如：http://hostname:8983/solr/core1）。当在基本URL中指定核心或集合时，则使用该客户端的后续请求不需要重新指定受影响的集合。但是，客户端仅限于向该核心/集合发送请求，而不能将请求发送给其他任何人。</li>2 <li>具有指向根Solr路径的通用路径的URL（例如，http://hostname:8983/solr）。如果在基本URL中未指定核心或集合，则可以向任何核心/集合发出请求，但必须在所有请求上指定受影响的核心/集合。</li>
## 设置XMLResponseParser
SolrJ使用二进制格式而不是XML作为默认的响应格式。如果您正在尝试将 Solr 和 SolrJ 版本混合在一起，其中一个版本是1.x，另一个是3.x或更高版本，那么您必须使用XML响应解析器。二进制格式在3.x中改变，两个javabin版本完全不兼容。以下代码将进行此更改：  
  
```
solr.setParser(new XMLResponseParser());
```
## 执行查询
使用query()有Solr的搜索结果。您必须传递一个描述查询的SolrQuery对象，然后您将返回QueryResponse（从org.apache.solr.client.solrj.response包中）。  
  
SolrQuery有一些方法可以方便地添加参数来选择请求处理程序并向其发送参数。这是一个非常简单的例子，它使用默认的请求处理程序并设置查询字符串：  
```
SolrQuery query = new SolrQuery();
query.setQuery(mQueryString);
```
要选择不同的请求处理程序，SolrJ 4.0及更高版本中提供了一个特定的方法：  
```
query.setRequestHandler("/spellCheckCompRH");
```
您也可以在查询对象上设置任意参数。下面的前两行代码相互等价，第三行显示如何使用任意参数q来设置查询字符串：  
```
query.set("fl", "category,title,price");
query.setFields("category", "title", "price");
query.set("q", "category:books");
```
完成SolrQuery设置后，请将其提交给query()：  
```
QueryResponse response = solr.query(query);
```
客户端建立网络连接并发送查询。Solr处理查询，并将响应发送并解析为一个QueryResponse。  
  
QueryResponse是一组满足查询参数的文档。您可以使用 getResults () 直接检索文档，并且您可以调用其他方法来查找有关突出显示或facet的信息。  
```
SolrDocumentList list = response.getResults();
```
## 索引文件
其他操作也很简单。如果要索引（补充）的文件，所有你需要做的就是创建一个SolrInputDocument并将其传递到SolrClient的add()方法。这个例子假定已经根据前面的例子创建了一个名为'solr'的SolrClient对象。  
```
SolrInputDocument document = new SolrInputDocument();
document.addField("id", "552199");
document.addField("name", "Gouda cheese wheel");
document.addField("price", "49.99");
UpdateResponse response = solr.add(document);
// Remember to commit your changes!
solr.commit();
```
### 以XML或二进制格式上传内容
SolrJ允许您以二进制格式上传内容，而不是默认的XML格式。使用以下代码以二进制格式上传，这与SolrJ用于获取结果的格式相同。如果您尝试混合Solr和SolrJ版本，其中一个版本是1.x，另一个是3.x或更高版本，那么您必须使用XML请求编写器。二进制格式在3.x中改变，两个javabin版本完全不兼容。  
```
solr.setRequestWriter(new BinaryRequestWriter());
```
### 使用ConcurrentUpdateSolrClient
当实现一次性批量加载大量文档的Java应用程序时，可以考虑ConcurrentUpdateSolrClient替代使用HttpSolrClient。该ConcurrentUpdateSolrClient缓存所有添加的文档，并将其写入到打开的HTTP连接。这个类是线程安全的。尽管任何SolrClient请求都可以用这个实现来完成，但是建议只对ConcurrentUpdateSolrClientfor 使用/update请求。  
  
## EmbeddedSolrServer
本EmbeddedSolrServer类提供SolrClient客户端API的实现，它直接与在您的 Java 应用程序中直接运行的 Solr 的微实例进行对话。这种嵌入式方法在大多数情况下不被推荐，并且在它所支持的一组功能上相当有限 - 特别是它不能与SolrCloud或索引复制一起使用。EmbeddedSolrServer的存在主要是为了帮助促进测试。  
  
有关如何使用EmbeddedSolrServer的信息，请查看Solr源代码版本的org.apache.solr.client.solrj.embedded包中的SolrJ JUnit测试。  
