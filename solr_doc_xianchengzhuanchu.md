## Solr线程转储 
<div class="content-intro view-box ">Solr 的线程转储界面允许您检查服务器上当前活动的线程。  
  
它将列出每个线程，并在适用的情况下访问堆栈跟踪（stacktraces）。在该线程界面中，左边的图标表示线程的状态：例如，绿色圆圈中带有绿色复选标记的线程处于 “RUNNABLE（可运行）” 状态。在线程名称的右侧，向下箭头表示您可以展开以查看该线程的堆栈跟踪。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510209648816880.png)  
当您将光标移到线程名称上时，一个框将浮动在该线程的状态名称上。线程状态可以是：   
  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">状态</th>
            <th style="text-align: center;">含义</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center; ">NEW  
  
            </td>
            <td>
                <p style="text-align: center;">尚未开始的线程  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">RUNNABLE  
            </td>
            <td>
                <p style="text-align: center; ">在 Java 虚拟机中执行的线程  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">BLOCKED  
            </td>
            <td>
                <p style="text-align: center;">被阻塞的线程正在等待显示器锁定  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center; ">WAITING  
  
            </td>
            <td>
                <p style="text-align: center;">无限期等待另一个线程执行特定操作的线程  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TIMED_WAITING  
            </td>
            <td>
                <p style="text-align: center; ">正在等待另一个线程执行<span style="background-color: transparent;">一个由指定的等待时间完成的操作的线程</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TERMINATED  
            </td>
            <td>
                <p style="text-align: center;">已退出的线程  
            </td>
        </tr>
    </tbody>
</table>
当你点击一个可以扩展的线程时，你会看到堆栈跟踪，如下面的示例所示：  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171109/1510210006390799.png)  

    您还可以选中 "显示所有 Stacktraces（Show all Stacktraces）" 按钮以自动启用所有线程的扩展。  
