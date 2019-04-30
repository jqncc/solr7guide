## Solr核心和solr.xml 
<div class="content-intro view-box ">在Solr中，术语"core"用于指单个索引和关联的事务日志和配置文件（包括solrconfig.xml和Schema文件等）。如果需要，您的Solr安装可以具有多个核心，这使您可以在同一台服务器中为不同结构的数据建立索引，并且更好地控制数据如何呈现给不同的受众。在 SolrCloud 模式中，您将更熟悉术语“collection”。在幕后，一个集合由一个或多个核心组成。  
  
可以使用bin/solr脚本创建内核，也可以使用API​​创建SolrCloud集合的一部分。特定于核心的属性（如用于索引或配置文件的目录，核心名称和其他选项）在core.properties文件中定义。您的Solr安装目录中的core.properties文件（或者solr_home定义的目录下）都将被Solr找到，定义的属性将用于文件中指定的核心。  
在独立模式下，solr.xml必须驻留在solr_home。在SolrCloud模式下，将从ZooKeeper加载solr.xml（如果它存在），回退到solr_home。  
注意：在较旧版本的Solr中，核心必须在<code>solr.xml</code>中预定义为<code>&lt;core&gt;</code>标签。为了让Solr了解它们。现在，Solr支持自动发现核心，它们不再需要显式定义。推荐的方法是使用api动态创建内核/集合。  
以下各节将更详细地介绍这些选项。  

    - solr.xml的格式：关于如何定义solr.xml的细节，包括solr.xml文件的可接受参数。
    - 定义core.properties：有关放置core.properties和可用属性选项的详细信息。
    - CoreAdmin API：使用REST API进行核心管理的工具和命令。
    - 配置集：如何使用配置集来避免在定义新核心时重复工作。
