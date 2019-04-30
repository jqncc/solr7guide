## Solr快速教程 
<div class="content-intro view-box ">本教程包括获取和运行 Solr，将各种数据源摄入到多个集合中，并对 Solr 管理和搜索接口有一定的了解。
      
  
它被组织成三个部分，每个部分都建立前一个部分之上。本章的第一个练习将要求您启动 Solr，创建一个集合，索引一些基本文档，然后执行一些搜索。  
第二个练习使用了一组不同的数据，并通过数据集探索请求的方面。  
第三个练习鼓励您开始使用自己的数据进行工作，并开始执行计划。  
最后，我们将介绍空间搜索，并向您展示如何使您的 Solr 实例恢复到最初的状态。  

## Solr 教程要求<a href="http://lucene.apache.org/solr/guide/7_0/solr-tutorial.html#before-you-begin"/>

要遵循本教程，您将需要达到以下要求：
      
  
1 <li>满足如下的系统要求：  <ul><li>Apache Solr 在 Java 8 或更高的版本上运行。</li>2 <li>建议始终使用 Java VM 的最新更新版本，因为错误可能会影响到 Solr，您可以在 [http://wiki.apache.org/lucene-java/JavaBugs](http://wiki.apache.org/lucene-java/JavaBugs) 上找到已知的 JVM 错误的概述。</li>3 <li>CPU、磁盘和内存要求是基于在实现 Solr 时所做的许多选择 (文档大小、文档数量和检索到的名称数)。基准页面有一些与特定平台上的性能有关的信息。</li></li>
    <li>Apache Solr 版本[下载](http://lucene.apache.org/solr/downloads.html)。本教程是为 Apache Solr 7.0 设计的。</li>
</ol>
为了获得最佳效果，请运行显示本教程的浏览器和 Solr 服务器在同一台机器上，以便教程链接能够正确指向您的 Solr 服务器。  

## 解压缩 Solr

首先解压缩 Solr 版本，然后将工作目录更改为安装了 Solr 的目录。例如，使用 UNIX，Cygwin 或 MacOS 中的 shell：
      
  
```
~$ ls solr*
solr-7.0.0.zip
~$ unzip -q solr-7.0.0.zip
~$ cd solr-7.0.0/
```

如果您想了解更多关于 Solr 的目录布局，然后再转到第一个练习，请参阅[目录布局](https://www.w3cschool.cn/solr_doc/solr_doc-ltzn2fm4.html#mlbj)获取更多详细信息。  
