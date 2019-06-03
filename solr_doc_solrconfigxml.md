# 配置solrconfig.xml文件

在Solr中solrconfig.xml文件是影响Solr本身参数最多的配置文件。

在配置Solr时，您会经常使用到solrconfig.xml，不论是直接或通过Config API创建“配置覆盖”（configoverlay.json）以覆盖solrconfig.xml中的值。  
在solrconfig.xml中，您需要配置下述的一些重要的功能，如：

- request handlers相关配置，处理对Solr的请求，例如向索引添加文档的请求或返回查询结果的请求
- listener，“监听”特定查询相关事件的过程; 侦听器可用于触发特殊代码的执行，例如调用一些常见查询来预热缓存
- 管理HTTP通信的请求调度程序配置
- 管理Web界面相关配置
- 为分布式的复制配置相关的参数（这些参数在Legacy Scaling and Distribution中详细介绍）

solrconfig.xml文件位于每个core的conf/目录中。在server/solr/configsets/目录中可以找到几个不同配置的示例文件。  

本章将介绍solrconfig.xml的以下部分：

- [DataDir和DirectoryFactory](solr_doc_solrconfigdatadir.md)
- [lib元素](solr_doc_solrconfiglib.md)
- [Schema Factory Definition](solr_doc_solrconfig_schemafactory.md)
- [IndexConfig](solr_doc_solrconfigindexconfig.md)
- RequestHandlers和SearchComponents
- InitParams
- UpdateHandlers 索引更新操作相关配置
- Query Settings 查询组件设置
- RequestDispatcher 请求处理器相关配置
- Update Request Processors
- Codec Factory编解码器工厂

## 配置文件中的变量属性

Solr支持在配置文件中属性值的变量替换，允许在solrconfig.xml中使用各种配置选项的运行时规范语法是：${propertyname[:option default value]}。这允许定义在Solr启动时可以覆盖的默认值。如果未指定默认值，则必须在运行时指定该属性，否则配置文件在解析时将生成错误。

在下面的内容中，有多种方法可以用于指定可以在配置文件中使用的属性。其中，强烈认为“配置覆盖”作为首选方法，因为它保持本地配置集，而且它很容易修改。

### JVM系统属性

在启动JVM时，通常使用-D标志指定的任何 JVM 系统属性都可用作 Solr 中任何 XML 配置文件中的变量。  

例如，在示例solrconfig.xml文件中，您将看到这个定义要使用的锁定类型的值：

`<lockType>${solr.lock.type:native}</lockType>`

这意味着锁定类型默认为“native”，但是当启动Solr时，可以通过启动Solr来使用JVM系统属性来覆盖它：

```shell
bin/solr start -Dsolr.lock.type=none
```

通常，您要设置的任何Java系统属性都可以使用标准-Dproperty=value语法通过bin/solr脚本传递。或者，您可以将通用的系统属性添加到Solr包含文件（bin/solr.in.sh或bin/solr.in.cmd）中定义的SOLR_OPTS环境变量。

### 通过Config API修改solrconfig.xml

Config API允许您使用一个API来修改Solr的配置，特别是用户定义的属性。使用此API进行的更改存储在名为configoverlay.json的文件中。这个文件只能用API进行编辑，看起来像下面这个例子：

```json
{"userProps":{
    "dih.db.url":"jdbc:oracle:thin:@localhost:1521",
    "dih.db.user":"username",
    "dih.db.pass":"password"}}
```

### solrcore.properties

如果Solr core的配置目录包含一个名为solrcore.properties的文件，则可以使用Java标准属性文件格式包含任意用户定义的属性名称和值，并且这些属性可以在该Solr core的XML配置文件中用作变量。  

例如，可以使用一个示例配置在集合的conf/目录中创建以下solrcore.properties文件，以覆盖所使用的lockType。

```
#conf/solrcore.properties
solr.lock.type=none
```

solrcore.properties 属性不会在 SolrCloud 模式下工作（不从ZooKeeper读取）。将来可能会删除此功能。相反，可以使用另一种机制，如配置叠加。  
solrcore.properties 文件的路径和名称可以使用 core.properties 中的 properties 属性重写。

### core.properties中的用户定义的属性

每个Solr core都有一个core.properties文件，在使用API​​时自动创建。在创建SolrCloud集合时，可以通过自定义参数传入每个将创建的core.properties，只需使用“property”作为参数名称前缀即可，比如URL参数，示例如下：  

```
http://localhost:8983/solr/admin/collections?action=CREATE&amp;name=gettingstarted&amp;numShards=1&amp;property.my.custom.prop=edismax
```

这将创建一个core.properties文件，至少具有以下属性（为简洁起见，省略了其他文件）：
```
#core.properties
name=gettingstarted
my.custom.prop=edismax
```

该my.custom.prop属性可以被用作一个变量，例如在solrconfig.xml中：

```xml
<requestHandler name="/select">
  <lst name="defaults">
    <str name="defType">${my.custom.prop}</str>
  </lst>
</requestHandler>
```

### 隐含的核心属性

Solr 核心的几个属性可以用作变量替换中使用的“隐式”属性，独立于初始化值的位置或方式。例如：不管特定Solr核心的名称是在core.properties实例目录的名称中显式配置还是从实例目录中推断出来的，隐式属性solr.core.name都可用作该核心配置文件中的变量。 

```xml
<requestHandler name="/select">
  <lst name="defaults">
    <str name="collection_name">${solr.core.name}</str>
  </lst>
<requestHandler>
```

所有隐式属性都使用solr.core.名称前缀，并反映等效core.properties属性的运行时值：

- solr.core.name
- solr.core.config
- solr.core.schema
- solr.core.dataDir
- solr.core.transient
- solr.core.loadOnStartup
