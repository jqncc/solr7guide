## CDCR监控 
<div class="content-intro view-box ">
## <span style="font-size: 14px;">CDCR</span>监控<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#monitoring-2"/>

1 <li>网络和磁盘空间监控是必不可少的。如果Source和Target之间断开连接，请确保系统有足够的可用存储来排队更改。两个数据中心之间的网络中断可能会导致您的磁盘使用率增加。<ul><li>提示：为磁盘设置监视器，以便在磁盘超过特定百分比（例如，70％）时发送警报。</li>2 <li>提示：运行测试。通过适度的索引，在磁盘空间不足之前系统队列可以多久变化？</li></li>
    <li>创建一个简单的方法来检查Source和Target之间的计数。- 请记住，如果索引正在运行，则Source文件和Target文件可能不匹配。如果差异大于整体cloud 大小的某个百分比，则设置警报以触发。
</li></ol>
