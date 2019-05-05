# Solr线程转储

Solr 的线程转储界面允许您检查服务器上当前活动的线程。

它将列出每个线程，并在适用的情况下访问堆栈跟踪（stacktraces）。在该线程界面中，左边的图标表示线程的状态：例如，绿色圆圈中带有绿色复选标记的线程处于 “RUNNABLE（可运行）” 状态。在线程名称的右侧，向下箭头表示您可以展开以查看该线程的堆栈跟踪。  
![solr](http://lucene.apache.org/solr/guide/7_0/images/thread-dump/thread_dump_1.png)  
当您将光标移到线程名称上时，一个框将浮动在该线程的状态名称上。线程状态可以是：   
  
状态|含义
---|--
NEW|尚未开始的线程  
RUNNABLE|在 Java 虚拟机中执行的线程
BLOCKED|被阻塞的线程正在等待显示器锁定  
WAITING|无限期等待另一个线程执行特定操作的线程  
TIMED_WAITING|正在等待另一个线程执行一个由指定的等待时间完成的操作的线程
TERMINATED|已退出的线程  

当你点击一个可以扩展的线程时，你会看到堆栈跟踪，如下面的示例所示：  
![solr thread_dump](http://lucene.apache.org/solr/guide/7_0/images/thread-dump/thread_dump_2.png)  

您还可以选中 "显示所有 Stacktraces（Show all Stacktraces）" 按钮以自动启用所有线程的扩展。
