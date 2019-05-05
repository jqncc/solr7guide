## Solr日志记录 
<div class="content-intro view-box ">在日志记录页面显示此 Solr 节点记录的最近消息。
      
  
当您单击 "日志记录" 的链接时，将显示一个类似于下面的页面：  
<p style="text-align: center; "> ![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510199647643943.png)  
在上述的 Solr 日志记录界面中，包含了由客户端发送的错误文档导致的错误示例。  
虽然本示例仅显示一个核心的记录消息，但如果您在单个实例中有多个核心，则它们将分别列出每个核心的级别。  

## Solr 选择记录级别<a href="http://lucene.apache.org/solr/guide/7_0/logging.html#selecting-a-logging-level"/>

当您选择左侧的 "级别" 链接时，您会看到您的实例的类路径和类名的层次结构。以黄色突出显示的行表示该类具有日志记录功能。点击突出显示的行，将出现一个菜单，允许您更改该类的日志级别。粗体中的字符表示该类不会受根级别更改的影响。  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510199852203024.png)  
有关各种日志记录级别的说明，请参阅配置日志记录。  
