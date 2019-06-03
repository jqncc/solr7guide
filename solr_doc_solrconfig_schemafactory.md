# Schema Factory Definition

Solr的Schema API使远程客户端能够通过REST接口访问模式信息并进行模式修改

其他特性，如Solr的Schemaless Mode，也可以在运行时通过编程方式进行架构修改。
>使用托管架构（Managed Schema）需要能够使用Schema API来修改您的架构。然而，使用托管架构本身并不意味着您也在无架构模式(或“架构猜测”模式)中使用Solr。无架构模式需要启用托管架构，但是完全模式猜测需要额外配置，如“无模式模式”部分中所述

尽管Schema API的“读取”特性对于所有架构类型都是支持的，但是支持以编程方式进行架构修改依赖于正在使用的&lt;schemaFactory/&gt;。

## Solr默认使用托管模式

当&lt;schemaFactory/&gt;没有在solrconfig.xml文件中显式声明时，Solr隐式使用a ManagedIndexSchemaFactory，默认情况下"mutable"将架构信息保存在managed-schema文件中。

```xml
 <!-- An example of Solr's implicit default behavior if no
      no schemaFactory is explicitly defined.
 -->
  <schemaFactory class="ManagedIndexSchemaFactory">
    <bool name="mutable">true</bool>
    <str name="managedSchemaResourceName">managed-schema</str>
  </schemaFactory>
```

如果您希望显式配置 ManagedIndexSchemaFactory，下列选项可用：

- mutable - 控制是否可以对Schema数据进行更改。这必须设置为true，以允许使用Schema API进行编辑。
- managedSchemaResourceName - 一个可选参数，默认为“managed-schema”，并为架构文件定义一个新的名称，该名称可以是除“schema.xml”以外的任何其他名称。

使用上面显示的默认配置，您可以使用Schema API根据需要修改架构，如果您希望“锁定”架构并防止将来更改，则稍后将值更改mutable为false。

## 经典schema.xml

使用托管架构的替代方法是显式配置一个ClassicIndexSchemaFactory。ClassicIndexSchemaFactory 需要使用schema.xml配置文件，并且不允许在运行时对架构进行任何编程式更改。该schema.xml文件必须手动编辑，仅在加载集合时才加载。

```xml
<schemaFactory class="ClassicIndexSchemaFactory"/>
```

### 从schema.xml切换到托管架构

如果您有一个现有的Solr集合使用了ClassicIndexSchemaFactory，并且您希望转换为使用托管模式，则可以简单地修改solrconfig.xml以指定使用ManagedIndexSchemaFactory。

一旦Solr重新启动，它检测到schema.xml文件存在，但managedSchemaResourceName文件（即：“managed-schema”）不存在，现有的schema.xml文件将被重命名为schema.xml.bak，内容被重新写入托管的架构文件。如果您查看生成的文件，您会在页面顶部看到这个：

```
<!-- Solr managed schema - automatically generated - DO NOT EDIT -->
```

您现在可以随意使用Schema API，只需要进行更改，然后删除schema.xml.bak。  

### 从托管架构切换到手动编辑的schema.xml

如果您启动了Solr并启用了托管架构，并且想要切换到手动编辑schema.xml文件，则应执行以下步骤：

1. 将managed-schema文件重命名为schema.xml
2. 修改solrconfig.xml以替换schemaFactory类。
    - 删除任何ManagedIndexSchemaFactory定义(如果存在)
    - 添加一个ClassicIndexSchemaFactory，如上所示的定义。
3. 重新加载核心。

如果您正在使用SolrCloud，则可能需要通过ZooKeeper修改文件。该bin/solr脚本提供了一种简单的方法来从ZooKeeper下载文件并在编辑之后将其上传。有关更多信息，请参阅ZooKeeper操作部分。
>要完全控制schema.xml文件，您可能还需要禁用架构猜测，这可以在编制索引期间将未知字段添加到架构中。在“无架构模式”一节中讨论了启用此功能的属性。
