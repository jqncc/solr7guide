# (Core Admin)Solr核心专用工具

Solr特定于core的工具是一组 UI 界面，允许您查看solr core的信息。  
  
在左侧的导航栏中，您将看到“核心选择器（Core Selector）”的下拉菜单。单击该菜单将显示此 Solr 节点上托管的 Solr 核心列表，其中包含可用于按名称查找特定核心的搜索框。

当您从下拉列表中选择一个核心时，页面的主要显示将显示关于所选core的一些基本元数据，而在左侧导航栏中将出现一个二级菜单，其中包含指向其他核心特定管理屏幕的链接。
您还可以定义一个名为 admin-extra.html 的配置文件，其中包含您希望在此主界面的 “Admin Extra” 部分中显示的链接或其他信息。  
![solr core_dashboard](http://lucene.apache.org/solr/guide/7_0/images/core-specific-tools/core_dashboard.png)
下面介绍各工具使用

## Ping请求功能

点击所选core下的Ping菜单会发出一个 ping 请求来检查核心是否启动并响应请求。  
![solr ping menu](http://lucene.apache.org/solr/guide/7_0/images/ping/ping.png)
  
由 Ping 执行的搜索是使用请求参数 API 进行配置的。请参阅 Implicit RequestHandlers，以了解用于 /admin/ping 端点的参数集。  

Ping 选项不打开页面，但是在点击集合名称时显示的核心概览页面上可以看到请求的状态。请求的时间长度显示在 Ping 选项旁边，以毫秒为单位。  

### API 示例

虽然在 UI 界面上可以很容易地看到 ping 响应时间，但是当由远程监视工具执行时，底层 ping 命令会更加有用：

输入：

```
http://localhost:8983/solr/<core-name>/admin/ping
```

这个命令将 ping 一个响应的核心名称。  
输入：  

```
http://localhost:8983/solr/<collection-name>/admin/ping?distrib=true
```

此命令将为响应 ping 给定集合名称的所有副本。  
示例输出：

```xml
<response>
   <lst name="responseHeader"></lst>
      <int name="status">0</int>
      <int name="QTime">13</int>
      <lst name="params">
         <str name="q">{!lucene}*:*</str>
         <str name="distrib">false</str>
         <str name="df">_text_</str>
         <str name="rows">10</str>
         <str name="echoParams">all</str>
      </lst>
   </lst>
   <str name="status">OK</str>
</response>
```

这两个 API 调用都有相同的输出。status=OK 表示节点正在响应。  
SolrJ 示例：

```
SolrPing ping = new SolrPing();
ping.getParams().add("distrib", "true"); //To make it a distributed request against a collection
rsp = ping.process(solrClient, collectionName);
int status = rsp.getStatus();
```

## 插件/统计（Plugins/Stats）功能

Solr Plugins/Stats界面显示有关每个Solr核心中运行的各种插件的状态和性能的信息和统计信息。您可以找到有关Solr缓存性能，Solr搜索器状态以及请求处理程序和搜索组件配置的信息。

在右侧选择感兴趣的区域，然后通过单击窗口中央部分中显示的名称之一深入查看更多细节。在这个例子中，我们选择从Core区域查看Searcher统计数据：
![solr plugin-searcher](http://lucene.apache.org/solr/guide/7_0/images/plugins-stats-screen/plugin-searcher.png)  
该显示是在页面加载时拍摄的快照。您可以通过选择 “观察更改” 或 “刷新值” 来获取更新的状态。观察这些更改将突出显示已更改的区域，而刷新值将使用更新的信息重新加载页面。  

## 主从复制（Replication）

"Replication"功能显示您指定的核心的当前主从复制状态。SolrCloud取代了大部分此功能，但如果您仍在使用Master-Slave索引复制，则可以使用此屏幕：

1. 查看可复制的索引状态。（在主节点上）
2. 查看当前复制状态（在从属节点上）
3. 禁用复制。（在主节点上）


## 细分信息（Segments Info）

通过“Segments Info”界面，您可以查看此Core的基础Lucene索引中各个细分的可视化，其中包含有关每个细分的大小的信息 - 包括字节数和文档数 - 以及有关这些细分的其他基本元数据。最明显的是已删除文档的数量，也可以将鼠标悬停在段上以查看其他数字详细信息。
![solr segments info screen](http://lucene.apache.org/solr/guide/7_0/images/segments-info/segments_info.png)


如果您正在运行 Solr 的单个节点实例，则通常在每个集合基础上显示的其他 UI 界面也将被列出：

- 分析（Analysis） - 让您分析在特定字段中找到的数据。
- 导入（Dataimport） - 显示有关数据导入处理程序的当前状态的信息。
- 文档（Documents） - 提供了一个简单的表单，允许您直接从浏览器执行各种 Solr 索引命令。
- 文件（Files） - 显示当前的核心配置文件，如：solrconfig.xml。
- 查询（Query） - 让您提交关于核心的各种元素的结构化查询。
- 流（Stream） - 允许您提交流表达式并查看结果和解析解释。
- 模式浏览器（Schema Browser） - 在浏览器窗口中显示架构数据。
