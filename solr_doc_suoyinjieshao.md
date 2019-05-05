## Solr索引介绍 
<div class="content-intro view-box ">本节介绍 Solr 索引的过程：将内容添加到 Solr 索引中，并在必要时修改该内容或将其删除。  
  
通过向索引添加内容，我们可以通过 Solr 进行搜索。  
Solr 索引可以接受来自许多不同来源的数据，包括 XML 文件、逗号分隔值（CSV）文件、从数据库表格中提取的数据以及常用文件格式（如 Microsoft Word 或 PDF）中的文件。  
以下是将数据加载到 Solr 索引中的三种最常见的方法：  

    - 使用基于 Apache Tika 构建的 Solr Cell 框架来获取二进制文件或结构化文件，如 Office、Word、PDF 和其他专有格式。
    - 通过向任何可以生成此类请求的环境发送 HTTP 请求到 Solr 服务器来上传 XML 文件。
    - 编写自定义 Java 应用程序以通过 Solr 的 Java Client API（在客户端 API 中更详细地描述）来获取数据。如果您正在使用提供 Java API 的应用程序（如内容管理系统（CMS）），则使用 Java API 可能是最佳选择。

不管用于提取数据的方法如何，都有一个共同的基本数据结构，用于将数据输入到 Solr 索引中：一个包含多个字段的文档，每个字段都有一个名称并包含内容，可能是空的。其中一个字段通常被指定为唯一的 ID 字段（类似于数据库中的主键），尽管 Solr 并不要求使用唯一的 ID 字段。  
  
如果在与索引关联的架构中定义了字段名称，那么当该内容被标记时，与该字段相关联的分析步骤将被应用于其内容。如果存在与字段名称匹配的字段，则在架构中未明确定义的字段将被忽略或映射到动态字段定义（请参阅文档：字段和架构设计）。  
有关 Solr 索引的更多信息，请参阅 Solr Wiki。  

## Solr 示例目录

当使用 “-e” 选项启动 Solr 时，该 example/ 目录将被用作所创建的示例 Solr 实例的基本目录。该目录还包含一个 example/exampledocs/ 子目录，其中包含各种格式的示例文档，您可以使用这些格式对各种示例中的索引进行测试。  
  

## 用于传输文件的 curl 实用程序

本节中的许多说明和示例都使用该 curl 实用程序通过 URL 传输内容。curl 通过 HTTP，FTP 和许多其他协议发布和检索数据。大多数 Linux 发行版都包含一份 curl 的副本。您可以在 http://curl.haxx.se/download.html 找到适用于 Linux、Windows 和许多其他操作系统的 curl 下载。curl 文档可在这里找到：http : 
    //curl.haxx.se/docs/manpage.html。  
```
Tip：使用 curl 或其他命令行工具发布数据对于示例或测试来说是很好的，但是这不是在生产环境中实现最佳性能更新的推荐方法。使用 Solr Cell 或本节中介绍的其他方法可以获得更好的性能。您可以使用 GNU wget（http://www.gnu.org/software/wget/）来代替 curl，或者使用 Perl 来管理 GETs 和 POSTS，虽然命令行选项会有所不同。
```
