## Solr：Post工具 
<div class="content-intro view-box ">Solr 包含一个简单的命令行工具，即 Post 工具（bin/post 工具），用于将各种类型的内容发布到 Solr 服务器。  
  
bin/post 工具是一个 Unix shell 脚本；对于 Windows（非 Cygwin）使用情况，请参阅下面的 “Post 工具 windows 支持”一节。  
要运行它，请打开一个窗口并输入：  
```
bin/post -c gettingstarted example/films/films.json
```
这将与服务器在 localhost:8983 联系。指定 collection/core name 是必需的。该 -help（或简称 -h）选项将输出有关其用法的信息（即：bin/post -help)。  

## 使用 bin / post 工具<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#using-the-bin-post-tool"/>

在使用 bin/post 时，指定任一 collection/core name 或完整更新 url 是必须的。  
  
bin/post 的基本用法是：  
```
$ bin/post -h
Usage: post -c &lt;collection&gt; [OPTIONS] &lt;files|directories|urls|-d ["...",...]&gt;
    or post -help
   collection name defaults to DEFAULT_SOLR_COLLECTION if not specified
OPTIONS
=======
  Solr options:
    -url &lt;base Solr update URL&gt; (overrides collection, host, and port)
    -host &lt;host&gt; (default: localhost)
    -p or -port &lt;port&gt; (default: 8983)
    -commit yes|no (default: yes)
    -u or -user &lt;user:pass&gt; (sets BasicAuth credentials)
  Web crawl options:
    -recursive &lt;depth&gt; (default: 1)
    -delay &lt;seconds&gt; (default: 10)

  Directory crawl options:
    -delay &lt;seconds&gt; (default: 0)
  stdin/args options:
    -type &lt;content/type&gt; (default: application/xml)

  Other options:
    -filetypes &lt;type&gt;[,&lt;type&gt;,...] (default: xml,json,csv,pdf,doc,docx,ppt,pptx,xls,xlsx,odt,odp,ods,ott,otp,ots,rtf,htm,html,txt,log)
    -params "&lt;key&gt;=&lt;value&gt;[&amp;&lt;key&gt;=&lt;value&gt;...]" (values must be URL-encoded; these pass through to Solr update request)
    -out yes|no (default: no; yes outputs Solr response to console)
...
```

## 使用 bin / post 示例<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#examples-using-bin-post"/>

有几种方法可以使用 bin/post。本节介绍几个例子。  
  

### 索引 XML<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#indexing-xml"/>

将文件扩展名为 .xml 的所有文档添加到命名为 gettingstarted 的集合或核心中。  
```
bin/post -c gettingstarted *.xml
```
将所有带有文件扩展名为 .xml 的文档添加到在端口 8984 上运行的 Solr 上的 gettingstarted 集合/内核。  
```
bin/post -c gettingstarted -p 8984 *.xml
```
发送 XML 参数以从 gettingstarted 中删除文档。  
```
bin/post -c gettingstarted -d '&lt;delete&gt;&lt;id&gt;42&lt;/id&gt;&lt;/delete&gt;'
```

### 索引 CSV<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#indexing-csv"/>

将所有 CSV 文件索引到 gettingstarted：  
  
```
bin/post -c gettingstarted *.csv
```
将制表符分隔的文件索引到 gettingstarted：  
```
bin/post -c signals -params "separator=%09" -type text/csv data.tsv
```
内容类型（-type）参数是需要将文件视为正确的类型，否则将被忽略，并记录一个警告，因为它不知道 .tsv 文件是什么类型的内容。该 CSV 处理器支持 separator 参数，并通过使用 -params 设置传递。  

### 索引 JSON<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#indexing-json"/>

将所有 JSON 文件编入索引 gettingstarted。  
```
bin/post -c gettingstarted *.json
```

### 索引丰富的文档（PDF、Word、HTML等）<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#indexing-rich-documents-pdf-word-html-etc"/>

将 PDF 文件索引到 gettingstarted。  
```
bin/post -c gettingstarted a.pdf
```
自动检测文件夹中的内容类型，并对其进行递归扫描，以便为编入 gettingstarted 的文档进行索引。  
```
bin/post -c gettingstarted afolder/
```
自动检测文件夹中的内容类型，但将其限制为 PPT 和 HTML 文件并将其索引到 gettingstarted。  
```
bin/post -c gettingstarted -filetypes ppt,html afolder/
```

### 索引到受密码保护的 Solr（基本身份验证）<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#indexing-to-a-password-protected-solr-basic-auth"/>

索引一个 PDF 作为用户 solr 使用密码 SolrRocks：  
  
```
bin/post -u solr:SolrRocks -c gettingstarted a.pdf
```

## 发布工具 Windows 支持<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#post-tool-windows-support"/>

bin/post 目前仅作为 Unix shell 脚本存在，但是它将其工作委派给了一个具有跨平台能力的 Java 程序。该 SimplePostTool 可以直接在支持的环境，包括 Windows 上运行。  

## SimplePostTool<a href="http://lucene.apache.org/solr/guide/7_0/post-tool.html#simpleposttool"/>

该 bin/post 脚本目前委托给一个名为 SimplePostTool 的独立 Java 程序。  
  
捆绑到可执行 JAR 中的这个工具可以直接运行 java -jar example/exampledocs/post.jar。请参阅 "帮助" 输出，并从那里获取文件、递归网站或文件系统文件夹，或直接发送命令到 Solr 服务器。  
```
$ java -jar example/exampledocs/post.jar -h
SimplePostTool version 5.0.0
Usage: java [SystemProperties] -jar post.jar [-h|-] [&lt;file|folder|url|arg&gt; [&lt;file|folder|url|arg&gt;...]]
.
.
.
```
