## Solr练习3：索引自己的数据 
<div class="content-intro view-box ">本节是 Solr 教程的最后一个练习，对于最后一个练习，请使用您选择的数据集。这可以是本地硬盘上的文件、之前使用过的一组数据，也可以是您打算为生产应用程序索引到 Solr 的数据的样本。  
  
这个练习是为了让您思考您需要为您的应用程序做什么：  

    - 您需要索引哪些类型的数据？  

    - 您需要做些什么来为您的数据准备 Solr（例如，创建特定字段，设置复制字段，确定分析规则等）
    - 您希望向用户提供哪些类型的搜索选项？  

    - 您需要做多少测试才能确保一切按您期望的方式进行？

## 创建你自己的集合

在开始之前，创建一个新的集合，命名为任何您想要的名称。在这个例子中，集合将被命名为 “localDocs”；如果需要，请将该名称替换为您选择的任何名称。  
  
```
./bin/solr create -c localDocs -s 2 -rf 2
```
同样，正如我们从上面的[练习2](https://www.w3cschool.cn/solr_doc/solr_doc-frzu2g9u.html)中看到的，这将使用 _default configSet 和它提供的所有无架构功能。正如我们前面提到的，当我们索引数据时，这可能会导致问题。在获得正确的架构之前，您可能需要多次对索引进行迭代。  

### Solr 提供的方法

Solr 有很多方法来索引数据。选择下面的方法之一，并与您的系统一起试用：  
  

**带有 bin / post 的本地文件**
    
        如果您有文件的本地目录，则 Post Tool（<code>bin/post</code>）可以索引文件的目录。我们在[第一个练习](https://www.w3cschool.cn/solr_doc/solr_doc-2gbo2fsg.html)中看到了这一点。  
        
            我们在练习中只使用了 JSON、XML 和 CSV，但 Post Tool 也可以处理 HTML、PDF、Microsoft Office 格式（如 MS Word）、纯文本等等。  
          
        
            在这个例子中，假设在本地有一个名为 “Documents” 的目录。要索引它，我们会发出这样的命令（根据需要在<code>-c</code>参数后面纠正集合名称）：  
```
./bin/post -c localDocs ~/Documents
```
  
        在文档中工作时，可能会出现错误。这些可能是由字段猜测引起的，或者文件类型可能不被支持。像这样的索引内容表明需要为您的数据规划 Solr，这需要了解它，也许还需要一些尝试和错误。  
          
    
**DataImportHandler**
    
        Solr 包含一个称为数据导入处理程序（DIH）的工具，它可以连接到数据库（如果您有 jdbc 驱动程序），邮件服务器或其他结构化数据源。有几个例子包括 feeds 、GMail 和一个小的 HSQL 数据库。  
        
            <code><font color="#000000" face="Verdana, Arial, Helvetica, sans-serif"><span style="white-space: normal; background-color: rgb(255, 255, 255);">在</span></font></code>example/example-DIH 中的<code>README.txt</code>文件将为您提供有关如何开始使用此工具的详细信息。  
  
**SolrJ**
    
        SolrJ 是一个与 Solr 交互的基于 Java 的客户端。将 SolrJ 用于基于 JVM 的语言或其他 Solr 客户端，以编程方式创建要发送到 Solr 的文档。  
    
**文档屏幕**
    
        使用管理用户界面文档选项卡（位于 http：// localhost：8983 / solr /＃/ localDocs / documents）粘贴到要建立索引的文档中，或者从<code>Document Type</code>下拉列表中选择 Document Builder 以一次生成一个字段的文档。单击表单下方的“提交文档（Submit Document）”按钮以索引文档。  


## 更新数据

您可能会注意到，即使您不止一次在本教程中对内容进行索引，也不会重复找到的结果。这是因为 Solr 架构（名为 managed-schemaor 或者 schema.xml 的文件）的示例指定了一个名为 id 的 uniqueKey 字段。无论何时您将命令发送到 Solr 以添加具有与 uniqueKey 现有文档相同的值的文档，它就会自动替换为您的文档。  
  
通过查看 Solr 管理 UI 的核心特定概述部分中的 numDocs 和 maxDoc 的值，您可以看到发生了这种情况。  
  
numDocs 表示索引中可搜索文档的数量（由于某些文件包含多个文档，因此将大于 XML、JSON 或 CSV 文件的数量）。该 maxDoc 值可能会更大，因为 maxDoc 计数包括逻辑上已删除但尚未从索引中删除的文档。您可以反复张贴示例文件，只要您想，numDocs 永远不会增加，因为新文档将不断地替换旧的。  
继续并编辑任何现有的示例数据文件，更改一些数据，然后重新运行 PostTool（bin/post）。您将看到在后续搜索中反映的更改。  

## 删除数据

如果您需要迭代几次才能使架构正确，那么您可能需要删除文档以清除集合，然后重试。但请注意，仅删除文档并不会更改基础字段定义。实质上，这将允许您在根据需要更改字段之后重新索引数据。  
  
您可以通过向更新 URL 发布删除命令并指定文档的唯一键字段的值或与多个文档匹配的查询（请注意该字段！）来删除数据。如果我们正确地构建请求，我们也可以使用 bin 或者 post 删除文档。   
执行以下命令删除特定的文档：  
```
bin/post -c localDocs -d "&lt;delete&gt;&lt;id&gt;SP2514N&lt;/id&gt;&lt;/delete&gt;"
```
要删除所有文档，可以使用“删除查询”命令：  
```
bin/post -c localDocs -d "&lt;delete&gt;&lt;query&gt;*:*&lt;/query&gt;&lt;/delete&gt;"
```
您也可以修改上述内容，只删除与特定查询匹配的文档。  

### Solr 练习3总结

在这一点上，您已经准备好开始自己的工作了。  
  
当您准备好停止 Solr 并删除所有与之合作的例子并重新开始时，您就可以跳过整个包了。  
