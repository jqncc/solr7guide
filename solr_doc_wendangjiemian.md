# Solr文档界面

文档界面提供了一个简单的表单，允许您直接从浏览器以各种格式执行各种Solr索引命令。  
  
![solr documents_add](http://lucene.apache.org/solr/guide/7_0/images/documents-screen/documents_add_screen.png)

Solr 文档界面允许您：

- 以 JSON、CSV 或 XML 格式复制文档并将其提交给索引；
- 上传文件（使用 JSON、CSV 或 XML 格式）
- 通过选择字段和字段值来构建文档

>还有一些其他的方法来加载数据，您可以参考下列章节：
>- 使用索引处理程序上传数据
>- 使用 Apache Tika 上传 Solr Cell 数据

第一步是定义 RequestHandler 以使用（aka，qt）。默认情况下 /update 会被定义。例如，要使用 Solr Cell，请将请求处理程序更改为 /update/extract。

然后选择“文档类型”来定义要加载的文档的类型。其余参数将根据所选的文件类型而改变。

## JSON 文档

使用 JSON 文档类型时，其功能与在命令行上使用 requestHandler 类似。不是将文档放在 curl 命令中，而是将其输入到 “文档” 输入框中。文档结构仍应采用适当的 JSON 格式。  
然后，您可以选择何时将文档添加到索引（Commit Within）中，以及是否应该用具有相同 ID 的传入文档覆盖现有文档（如果不是 true，则传入文档将被丢弃）。  
这个选项只会添加或覆盖文件到索引中，对于其他更新任务，请参阅 Solr 命令选项。 

## CSV 文档

使用 CSV 文档类型时，其功能与在命令行上使用 requestHandler 类似。不是将文档放在 curl 命令中，而是将其输入到 “文档” 输入框中。文档结构仍然应该是正确的 CSV 格式：带有列分隔符和一行文档。  
然后，您可以选择何时将文档添加到索引（Commit Within）中，以及是否应该用具有相同 ID 的传入文档覆盖现有文档（如果不是 true，则传入文档将被丢弃）。  

## 文档生成器

文档生成器提供了一个类似于向导的界面，用于输入文档的字段。  

## 上传文件

文件上传选项允许选择一个准备好的文件并将其上传。如果仅将 /update 选项用于请求处理程序，则您将被限制为 XML、CSV 和 JSON。  
但是，要使用 ExtractingRequestHandler（又名 Solr Cell），您可以将 Request-Handler 修改为 /update/extract。您必须在您的 solrconfig.xml 文件中定义您所需的默认值。您还应该添加 &amp;literal.id，是其显示在“提取需求处理程序参数（Extracting Req. Handler Params）”字段中，以便选择的文件具有唯一的 ID。
然后，您可以选择何时将文档添加到索引（Commit Within）中，以及是否应该用具有相同 ID 的传入文档覆盖现有文档（如果不是 true，则传入文档将被丢弃）。  

## Solr 命令

Solr 命令选项允许您使用 XML 或 JSON 对文档执行特定的操作，例如定义要添加或删除的文档，只更新文档的某些字段，或提交和优化索引上的命令。  
  
这些文档的结构应该像 /update 在命令行中使用一样。  

## XML 文档

使用 XML 文档类型时，其功能与在命令行上使用 requestHandler 类似。不是将文档放在 curl 命令中，而是将其输入到 “文档” 输入框中。文档结构仍应采用适当的 Solr XML 格式，每个文档由 &lt;doc&gt; 标签分隔，并且每个字段被定义。  
然后，您可以选择何时将文档添加到索引（Commit Within）中，以及是否应该用具有相同 ID 的传入文档覆盖现有文档（如果不是 true，则传入文档将被丢弃）。  
这个选项只会添加或覆盖文件到索引；对于其他更新任务，请参阅 Solr 命令选项。  
