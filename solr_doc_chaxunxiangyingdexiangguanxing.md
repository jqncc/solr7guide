## Solr查询响应的相关性 
<div class="content-intro view-box ">Solr 相关性是查询响应满足正在搜索信息的用户的程度。  
  
查询响应的相关性取决于执行查询的上下文。一个单独的搜索应用程序可能会被不同的需求和期望的用户在不同的环境中使用。例如，一个研究气候数据的搜索引擎可能在以下的场合被使用，例如：一个研究长期气候趋势的大学研究人员，一个有兴趣计算春季最后霜冻的可能日期的农民，一个对降雨模式和洪水频率感兴趣的土木工程师，以及一个大学生计划去一个地区度假，想知道要收拾什么。由于这些用户的动机不同，对查询的任何特定响应的相关性也会有所不同。  
查询响应应该有多全面？与一般意义上的相关性一样，这个问题的答案取决于搜索的上下文。在某些情况下，不响应查询找到特定文档的成本很高，例如响应于传票的法定 e-discovery 发现搜索，而在其他情况下相当低，例如在网络上搜索蛋糕配方，会出现几十个或几百个蛋糕食谱的网站。在配置 Solr 时，您应该权衡其他因素，如及时性和易用性。  
上述提到的两个例子：e-discovery 和菜谱实例，证明了与相关性相关的两个概念的重要性：  

    - 精度（precision）是返回结果中相关文档的百分比。
    - 召回（recall）一下系统中所有相关结果的相关结果的百分比。获得完美的召回是微不足道的：只需简单地将每个文档返回到每个查询的集合中。

回到上面的例子，一个 e-discovery 搜索应用程序有 100％ 召回返回与传票有关的所有文件是非常重要的。然而，一个菜谱应用程序提供这样的精确度就不那么重要了。在某些情况下，在不经意的情况下返回太多的结果可能会压倒用户。在某些情况下，返回较少的结果具有更高的相关性可能是最好的方法。  
  
使用精确度和召回的概念，可以将用户的相关性和对文档集合的查询进行量化。一个完美的系统对每个用户和每个查询都有100％的精度和100％的召回。换句话说，它将检索所有相关的文件，没有其他的。实际上，当谈到实际系统中的精确度和召回率时，通常关注的是精确度和召回率，在一定数量的结果中，最常见的(也是有用的)是10个结果。  
通过 faceting、查询过滤器和其他搜索组件，可以灵活配置 Solr 应用程序，以帮助用户对搜索进行微调，以便为用户返回最具有相关性的结果。也就是说，Solr 可以配置为平衡精确度和召回率，以满足特定用户群体的需求。  
Solr 应用程序的配置应该考虑到：  

    - 应用程序的各种用户的需求（除了严格的信息需求外，还包括易用性和响应速度）
    - 在不同的上下文中（如日期、产品类别或地区）对这些用户有意义的类别
    - 文档的任何固有的相关性（例如，确保官方产品说明或 FAQ 总是返回到搜索结果的顶部附近是有意义的）
    - 文件的日期是否重要（在某些情况下，最近的文件可能永远是最重要的）

记住所有这些因素，在Solr部署的规划阶段，通常会帮助您勾画出您认为搜索应用程序应该返回的示例查询的响应类型。一旦应用程序启动并运行，您就可以使用一系列测试方法，如焦点组、内部测试、TREC 测试和 A/B 测试来微调应用程序的配置，以最好地满足用户的需求。  
有关相关性的更多信息，请参阅 Grant Ingersoll 在  SearchHub.org 上提供的技术文章：[调试搜索应用程序相关性问题](https://lucidworks.com/2009/09/02/debugging-search-application-relevance-issues/)。  
