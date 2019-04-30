## Solr隐式RequestHandlers 
<div class="content-intro view-box ">Solr附带了很多现成的RequestHandler，因为它们没有在solrconfig.xml中配置，所以被称为隐式的。  
  
这些处理程序已经预先定义的默认参数，被称为paramsets，其可以根据需要进行修改。  

## 隐式可用端点列表

<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">端点</th>
            <th style="text-align: center;">请求处理程序类</th>
            <th style="text-align: center;">Paramset</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/file</code>
                  
            </td>
            <td>
                <p style="text-align: center;">ShowFileRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_FILE</code>
                  
            </td>
            <td>
                <p style="text-align: left;">返回<code>${solr.home}</code><code>/conf/</code>中文件的内容  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/logging</code>
                  
            </td>
            <td>
                <p style="text-align: center;">LoggingHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_LOGGING</code>
                  
            </td>
            <td>
                <p style="text-align: center;">检索/修改已注册的记录器。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/luke</code>
                  
            </td>
            <td>
                <p style="text-align: center;">LukeRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_LUKE</code>
                  
            </td>
            <td>
                <p style="text-align: center;">公开内部lucene索引。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/mbeans</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SolrInfoMBeanHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_MBEANS</code>
                  
            </td>
            <td>
                <p style="text-align: center;">提供有关所有注册的SolrInfoMBeans的信息。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/ping</code>
                  
            </td>
            <td>
                <p style="text-align: center;">PingRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_PING</code>
                  
            </td>
            <td>
                <p style="text-align: center;">安全检查。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/plugins</code>
                  
            </td>
            <td>
                <p style="text-align: center;">PluginInfoHandler
                  
            </td>
            <td>
                <p style="text-align: center;">N / A  
            </td>
            <td>
                <p style="text-align: center;">返回关于所有注册插件的信息。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/properties</code>
                  
            </td>
            <td>
                <p style="text-align: center;">PropertiesRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_PROPERTIES</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回JRE系统属性。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/segments</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SegmentsInfoRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_SEGMENTS</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回上次提交生成Lucene索引段的信息。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/system</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SystemInfoHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_SYSTEM</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回服务器统计和设置  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/admin/threads</code>
                  
            </td>
            <td>
                <p style="text-align: center;">ThreadDumpHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_THREADS</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回所有JVM线程的信息。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/analysis/document</code>
                  
            </td>
            <td>
                <p style="text-align: center;">DocumentAnalysisRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ANALYSIS_DOCUMENT</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回给定文档分析过程的细目。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/analysis/field</code>
                  
            </td>
            <td>
                <p style="text-align: center;">FieldAnalysisRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ANALYSIS_FIELD</code>
                  
            </td>
            <td>
                <p style="text-align: center;">返回给定字段/字段类型的索引和查询时间分析。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/config</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SolrConfigHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_CONFIG</code>
                  
            </td>
            <td>
                <p style="text-align: center;">检索/修改Solr配置。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/debug/dump</code>
                  
            </td>
            <td>
                <p style="text-align: center;">DumpRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_DEBUG_DUMP</code>
                  
            </td>
            <td>
                <p style="text-align: center;">将请求内容回送给客户端。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/export</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SearchHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_EXPORT</code>
                  
            </td>
            <td>
                <p style="text-align: center;">导出完整排序的结果集。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/get</code>
                  
            </td>
            <td>
                <p style="text-align: center;">RealTimeGetHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_GET</code>
                  
            </td>
            <td>
                <p style="text-align: center;">实时获取：低延迟检索文档的最新版本。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/graph</code>
                  
            </td>
            <td>
                <p style="text-align: center;">GraphHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_ADMIN_GRAPH</code>
                  
            </td>
            <td>
                <p style="text-align: left;">从<code style="text-align: left;">gather</code><code style="text-align: left;">Nodes</code><span style="background-color: transparent;">流表达式</span><span style="background-color: transparent;">返回</span><span style="background-color: transparent;">GraphML</span><span style="background-color: transparent;">格式化的输出。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/replication</code>
                  
            </td>
            <td>
                <p style="text-align: center;">ReplicationHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_REPLICATION</code>
                  
            </td>
            <td>
                <p style="text-align: left;"><span style="background-color: transparent;">复制SolrCloud恢复和</span><span style="background-color: transparent; text-align: left;">Master/Slave</span><span style="background-color: transparent;">索引分布的索引。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/schema</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SchemaHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_SCHEMA</code>
                  
            </td>
            <td>
                <p style="text-align: center;">检索/修改Solr模式。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/sql</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SQLHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_SQL</code>
                  
            </td>
            <td>
                <p style="text-align: center;">并行SQL接口的前端。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/stream</code>
                  
            </td>
            <td>
                <p style="text-align: center;">StreamHandler中
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_STREAM</code>
                  
            </td>
            <td>
                <p style="text-align: center;">分布式流处理。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/terms</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SearchHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_TERMS</code>
                  
            </td>
            <td>
                <p style="text-align: left;">返回一个字段的索引条款和包含每个条款的文档数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/update</code>
                  
            </td>
            <td>
                <p style="text-align: center;">UpdateRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_UPDATE</code>
                  
            </td>
            <td>
                <p style="text-align: left;">添加，删除和更新格式为SolrXML，CSV，SolrJSON或javabin的索引文档。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/update/csv</code>
                  
            </td>
            <td>
                <p style="text-align: center;">UpdateRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_UPDATE_CSV</code>
                  
            </td>
            <td>
                <p style="text-align: center;">添加和更新CSV格式的文档。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/update/json</code>
                  
            </td>
            <td>
                <p style="text-align: center;">UpdateRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_UPDATE_JSON</code>
                  
            </td>
            <td>
                <p style="text-align: center;">添加，删除和更新SolrJSON格式的文档。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>/update/json/docs</code>
                  
            </td>
            <td>
                <p style="text-align: center;">UpdateRequestHandler
                  
            </td>
            <td>
                <p style="text-align: center;"><code>_UPDATE_JSON_DOCS</code>
                  
            </td>
            <td>
                <p style="text-align: center;">添加和更新自定义JSON格式的文档。  
            </td>
        </tr>
    </tbody>
</table>
## 如何查看配置

您可以通过Config API查看所有请求处理程序的配置，包括隐式请求处理程序。对于gettingstarted集合：  
```
curl http://localhost:8983/solr/gettingstarted/config/requestHandler
```
要将结果限制为特定请求处理程序的配置，请使用componentName请求参数。以下是仅查看/export请求处理程序的配置：  
```
curl "http://localhost:8983/solr/gettingstarted/config/requestHandler?componentName=/export"
```
要在响应中包含扩展参数集以及将参数集参数与内置参数合并的有效参数，请使用expandParams请求参数。对于/export请求处理程序，可以这样提出请求：  
```
curl "http://localhost:8983/solr/gettingstarted/config/requestHandler?componentName=/export&amp;expandParams=true"
```

## 如何编辑配置

因为隐式请求处理程序不存在于solrconfig.xml，因此可以使用上表中列出的 paramset 通过请求参数 API 编辑其关联的default、invariant和appends参数的配置。但是，其他参数（包括SearchHandler 组件）可能不会被修改。隐式配置中指定的不变量和附加值不能被覆盖。  
