## 配置ReplicationHandler 
<div class="content-intro view-box ">
## 配置ReplicationHandler
除了特定于主/从角色的ReplicationHandler配置选项之外，还有一些通常支持的特殊配置选项（即使在使用SolrCloud时也是如此）。  
  
- maxNumberOfBackups是一个整数值，表示这个节点在接收backup命令时将保留在磁盘上的最大备份数量。
- 与Solr中的大多数其他请求处理程序类似，您可以配置一组默认值、不变量或追加与ReplicationHandlerwhen 处理命令支持的任何请求参数相对应的参数。
### 在主服务器上配置复制RequestHandler
在运行复制之前，应该在处理程序初始化时设置以下参数：  
**replicateAfter**
字符串指定在哪个复制发生之后的操作。有效值是提交，优化或启动。这个参数可以有多个值。如果您使用“启动”，如果您希望在将来的提交或优化时触发复制，则还需要“提交”或“优化”条目。  

backupAfter 字符串指定后面应该发生备份的操作。有效值是提交，优化或启动。这个参数可以有多个值。它不需要复制，它只是做一个备份。  
maxNumberOfBackups是整数，指定要保留多少备份。这可以用来删除除最近N个备份以外的所有备份。  
**confFiles**
要复制的配置文件，用逗号分隔。  
**commitReserveDuration**
如果您的提交非常频繁且您的网络速度较慢，则可以调整此参数以增加从主设备下载5Mb到从设备所需的时间。默认值是10秒。  

下面的示例显示了一个可能的“主”配置ReplicationHandler，包括固定数量的备份和maxWriteMBPerSec请求参数的不变设置，以防止从设备饱和其网络接口：  
```
&lt;requestHandler name="/replication" class="solr.ReplicationHandler"&gt;
  &lt;lst name="master"&gt;
    &lt;str name="replicateAfter"&gt;optimize&lt;/str&gt;
    &lt;str name="backupAfter"&gt;optimize&lt;/str&gt;
    &lt;str name="confFiles"&gt;schema.xml,stopwords.txt,elevate.xml&lt;/str&gt;
    &lt;str name="commitReserveDuration"&gt;00:00:10&lt;/str&gt;
  &lt;/lst&gt;
  &lt;int name="maxNumberOfBackups"&gt;2&lt;/int&gt;
  &lt;lst name="invariants"&gt;
    &lt;str name="maxWriteMBPerSec"&gt;16&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
#### 复制solrconfig.xml
在主服务器上的配置文件中，包含如下所示的行：  
```
&lt;str name="confFiles"&gt;solrconfig_slave.xml:solrconfig.xml,x.xml,y.xml&lt;/str&gt;
```
这确保本地配置solrconfig_slave.xml将被作为solrconfig.xml保存在从属。所有其他文件将以原始名称保存。  
  
在主服务器上，只要名称在confFiles字符串中正确标识，则从属配置文件的文件名就可以是任何内容，那么它将被保存为冒号“：”之后出现的任何文件名。  
### 在从属服务器上配置复制RequestHandler
下面的代码显示了如何在从属上配置ReplicationHandler。  
```
&lt;requestHandler name="/replication" class="solr.ReplicationHandler"&gt;
  &lt;lst name="slave"&gt;
    &lt;!-- fully qualified url for the replication handler of master. It is
         possible to pass on this as a request param for the fetchindex command --&gt;
    &lt;str name="masterUrl"&gt;http://remote_host:port/solr/core_name/replication&lt;/str&gt;
    &lt;!-- Interval in which the slave should poll master.  Format is HH:mm:ss .
         If this is absent slave does not poll automatically.
         But a fetchindex can be triggered from the admin or the http API --&gt;
    &lt;str name="pollInterval"&gt;00:00:20&lt;/str&gt;
    &lt;!-- THE FOLLOWING PARAMETERS ARE USUALLY NOT REQUIRED--&gt;
    &lt;!-- To use compression while transferring the index files. The possible
         values are internal|external.  If the value is 'external' make sure
         that your master Solr has the settings to honor the accept-encoding header.
         See here for details: http://wiki.apache.org/solr/SolrHttpCompression
         If it is 'internal' everything will be taken care of automatically.
         USE THIS ONLY IF YOUR BANDWIDTH IS LOW.
         THIS CAN ACTUALLY SLOWDOWN REPLICATION IN A LAN --&gt;
    &lt;str name="compression"&gt;internal&lt;/str&gt;
    &lt;!-- The following values are used when the slave connects to the master to
         download the index files.  Default values implicitly set as 5000ms and
         10000ms respectively. The user DOES NOT need to specify these unless the
         bandwidth is extremely low or if there is an extremely high latency --&gt;
    &lt;str name="httpConnTimeout"&gt;5000&lt;/str&gt;
    &lt;str name="httpReadTimeout"&gt;10000&lt;/str&gt;
    &lt;!-- If HTTP Basic authentication is enabled on the master, then the slave
         can be configured with the following --&gt;
    &lt;str name="httpBasicAuthUser"&gt;username&lt;/str&gt;
    &lt;str name="httpBasicAuthPassword"&gt;password&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```
