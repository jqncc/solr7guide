## 配置Solr日志记录 
<div class="content-intro view-box ">Solr日志是了解系统中发生的情况的关键方法。有几种方法可以调整默认的日志记录配置。  
  
除了下面介绍的日志记录选项外，还有一种方法可以配置哪些请求参数（如作为查询的一部分发送的参数），并使用附加的名为<code>logParamsList</code>的请求参数进行记录。有关更多信息，请参阅常见查询参数一节。  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">临时记录设置</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/configuring-logging.html#temporary-logging-settings"/>
您可以使用Admin Web界面来控制Solr中的日志输出量。选择LOGGING链接。请注意，此页面只允许您更改正在运行的系统中的设置，并不会保存在下一次​​运行中。（有关Admin Web界面的更多信息，请参阅[使用Solr管理用户界面](https://www.w3cschool.cn/solr_doc/solr_doc-fbc32fuv.html)。）  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171221/1513836203726867.png)  
Admin Web界面的这一部分允许您为许多不同的日志类别设置日志记录级别。幸运的是，任何未设置的类别都将具有其父级的日志记录级别。这样就可以通过调整其父级的日志级别来一次更改多个类别。  
  
当您选择“级别”时，您会看到以下菜单：  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171221/1513836559385107.png)  
目录显示当前的日志记录级别。日志级别菜单浮动在这些上面。要为特定目录设置日志级别，请选择它并单击相应的日志级别按钮。  
  
日志级别设置如下：  
<table class=""><colgroup><col/><col/></colgroup><thead><tr><th style="text-align: center;">级别</th><th style="text-align: center;">结果</th></tr></thead><tbody><tr><td><p style="text-align: center;">FINEST  
</td><td><p style="text-align: center;">报告一切  
</td></tr><tr><td><p style="text-align: center; "><span style="text-align: left;">FINE</span>  
  
</td><td><p style="text-align: center;">报告一切，但都是最不重要的信息  
</td></tr><tr><td><p style="text-align: center;">CONFIG  
</td><td><p style="text-align: center;">报告配置错误  
</td></tr><tr><td><p style="text-align: center;">INFO  
  
</td><td><p style="text-align: center;">报告一切，但是正常状态的  
</td></tr><tr><td><p style="text-align: center;">WARN  
  
</td><td><p style="text-align: center;">报告所有警告  
</td></tr><tr><td><p style="text-align: center;">SEVERE  
  
</td><td><p style="text-align: center;">只报告最严重的警告  
</td></tr><tr><td><p style="text-align: center;">OFF  
  
</td><td><p style="text-align: center;">关闭日志记录  
</td></tr><tr><td><p style="text-align: center;">UNSET  
</td><td><p style="text-align: center;">删除以前的日志设置  
</td></tr></tbody></table>
```
  
Note：一次可以允许有多个设置。
```
### 日志级别的API<a href="http://lucene.apache.org/solr/guide/7_0/configuring-logging.html#log-level-api"/>
还有一种方法可以将 REST 命令发送到日志记录端点以执行相同的功能。例如:  
```
# Set the root logger to level WARN
curl -s http://localhost:8983/solr/admin/info/logging --data-binary "set=root:WARN"
```
## 在启动时选择日志级别<a href="http://lucene.apache.org/solr/guide/7_0/configuring-logging.html#choosing-log-level-at-startup"/>
您可以在启动Solr时暂时选择不同的日志记录级别。有两种方法：  
第一种方法是在启动Solr之前设置SOLR_LOG_LEVEL环境变量，或者将相同的变量放在bin/solr.in.sh或bin/solr.in.cmd中。变量必须包含支持日志级别的大写字符串（请参见上文）。  
第二种方法是使用-v或-q选项启动Solr，有关详细信息，请参阅[Solr控制脚本参考](https://www.w3cschool.cn/solr_doc/solr_doc-m13y2fs3.html)。例子：  
```
# Start with verbose (DEBUG) looging
bin/solr start -f -v
# Start with quiet (WARN) logging
bin/solr start -f -q
```
## 永久记录设置<a href="http://lucene.apache.org/solr/guide/7_0/configuring-logging.html#permanent-logging-settings"/>
Solr 使用 Log4J 版本1.2 进行日志记录，它是使用server/resources/log4j.properties进行配置。花点时间检查log4j.properties文件的内容，以便熟悉其结构。默认情况下，Solr日志消息将被写入SOLR_LOGS_DIR/solr.log。  100000 100  
  
准备在生产环境中部署Solr时，请将变量 SOLR_LOGS_DIR 设置为希望 Solr 写入日志文件的位置，例如：/var/solr/logs。您可能也想调整log4j.properties。请注意，如果您使用Solr生产中提供的说明将Solr作为服务安装，则请参阅/var/solr/log4j.properties，而不是默认的server/resources版本。  
当在前台（-f选项）启动Solr时，除了solr.log，所有日志将被发送到控制台。当在后台启动Solr时，它会将所有的stdout和stderr输出写入到solr-&lt;port&gt;-console.log的一个日志文件中，并自动禁用log4j.properties配置的CONSOLE记录器，其效果与您从rootLogger手动删除CONSOLE appender效果相同。  
此外，log4j.properties默认的日志旋转大小阈值4MB 对生产服务器来说很可能太小，应该增加到更大的值（例如100MB或更多）。  
```
log4j.appender.file.MaxFileSize=100MB
```
当垃圾收集日志大小达到20M时，Java垃圾收集日志被JVM旋转，最多9代。旧的GC日志被移动到SOLR_LOGS_DIR/archived。这些设置只能通过编辑start脚本来更改。  
  
在Solr的每次启动时，start脚本将清理旧日志并旋转主solr.log文件。如果在log4j.properties中更改了log4j.appender.file.MaxBackupIndex设置，则还需要更改 start 脚本中的相应-rotate_solr_logs 9 设置。  
您可以通过更改设置在启动时禁用自动登录旋转，通过在bin/solr.in.sh或bin/solr.in.cmd中发现的SOLR_LOG_PRESTART_ROTATION设置为false。  
## 记录慢速查询<a href="http://lucene.apache.org/solr/guide/7_0/configuring-logging.html#logging-slow-queries"/>
对于大容量的搜索应用程序，记录每个查询可能会生成大量日志，并且根据卷的大小，可能会影响性能。如果您挖掘这些日志以获取对应用程序的更多见解，那么记录每个查询请求可能会有用。  
  
另一方面，如果您只关心与请求相关的警告和错误消息，则可以将日志详细程度设置为WARN。但是，这会造成潜在的问题，因为您不知道是否有任何查询缓慢，因为缓慢的查询仍然记录在INFO级别。  
Solr提供了一种将日志详细度阈值设置为WARN的方法，并且可以设置一个延迟阈值，在该阈值之上，请求被视为“slow”，并在WARN级别记录该请求，以帮助您识别应用程序中的缓慢查询。要启用此行为，请在solrconfig.xml 的查询部分配置该&lt;slowQueryThresholdMillis&gt;元素：  
```
&lt;slowQueryThresholdMillis&gt;1000&lt;/slowQueryThresholdMillis&gt;
```
超过指定阈值的任何查询将在WARN级别被记录为“slow”查询。  
