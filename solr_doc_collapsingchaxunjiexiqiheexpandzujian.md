## Collapsing查询解析器和Expand组件 
<div class="content-intro view-box ">Collapsing 查询解析器和 Expand 组件结合起来，形成了在 Solr 搜索结果中为字段折叠而对文档进行分组的方法。   
Collapsing 查询解析器根据您的参数对文档进行分组（折叠结果集），而 Expand 组件则提供对折叠组中的文档的访问，以便在结果显示或客户端应用程序的其他处理中使用。Collapse 和Expand 可以共同执行之前的 Result Grouping（group=true）对大多数用例都适用但不是全部用例。一般来说，您应该选择折叠和扩展。  
为了在 SolrCloud 中使用这些功能，文档必须位于同一个分片上。为确保文档在同一地点，您可以在创建集合时定义<code>router.name</code>参数作为 compositeId。有关此选项的详细信息, 请参阅文档路由一节。   
<h2>Collapsing查询解析器  
</h2>
CollapsingQParser 实际上是一个 post 过滤器，当结果集中的不同组的数量很多时，提供比 Solr 标准方法更高的性能字段折叠。在将结果集转发给其余搜索组件之前，此解析器会将结果集折叠为每个组的单个文档。所以所有的下游组件（faceting，highlighting等）都可以与折叠的结果集一起工作。
      
  
CollapsingQParser 接受以下本地参数：  
- field 参数  

   
        正在折叠的字段。该字段必须是单值 String、Int 或 Float 类型的字段。  
    
- min 或 max 参数  

        根据哪个文档具有指定数字字段或函数查询的最小或最大值，为每个组选择组头文档。  
        
            至多只能指定一个<code>min</code>，<code>max</code>或<code>sort</code>（见下文）参数。  
          
        
            如果没有指定，则将根据该组中的最高评分文档来选择每个组的组长文档。默认值是“none”。  
          
    
- sort 参数  

   
        根据指定的 sort 字符串，根据哪个文档最先出现，为每个组选择组头文档。  
        
            至多只能指定一个<code>min</code>，<code>max</code>，（见上文）或<code>sort</code>参数。  
          
        
            如果没有指定，则将根据该组中的最高评分文档来选择每个组的组长文档。默认值是“none”。  
          
    
- nullPolicy 参数  

   
        有三个可用的空策略：
              
          
   
        
            <ul>
                <li>
                    <code>ignore</code>：在折叠字段中删除具有 null 值的文档。这是默认的。  
                
                - 
                    <code>expand</code>：将折叠字段中的每个文档的 null 值作为单独的组处理。  
                
                - 
                    <code>collapse</code>：使用最高分或最小/最大值将所有具有 null 值的文档合并为一个组。  
                    
                        默认是<code>ignore</code>。  
                      
                
            
          
    </li><li>hint 参数  

   
        目前只有一个提示可用：<code>top_fc</code>，它代表顶级 FieldCache。  
        
            该<code>top_fc</code>提示仅在折叠字符串字段时可用。<code>top_fc</code>通常会提供最佳的查询时间速度，但在启动或提交后需要最长的时间来预热。<code>top_fc</code>还将导致折叠字段在内存中缓存两次，如果它用于分面或排序。对于非常高的基数（高分数）的字段，<code>top_fc</code>可能不那么好。  
          
        
            默认值是“none”。  
          
    </li><li>size 参数  

 
        仅在折叠数值字段时设置折叠数据结构的初始大小。  
        
            用于折叠的数据结构在数值字段上折叠时动态增长。将大小设置为结果集中预期的结果数量将会消除调整大小的成本。  
          
        
            默认值是“100,000”。  
          
    </li>
</ul>

### 示例语法

折叠 group_field 选择每个组中具有最高分数文档的文档:：  
```
fq={!collapse field=group_field}
```

折叠 group_field 选择每个组中的文档，其中最小值为 numeric_field:  
```
fq={!collapse field=group_field min=numeric_field}
```

折叠 group_field 选择每个组中的文档，其中最大值为 numeric_field：  
```
fq={!collapse field=group_field max=numeric_field}
```

折叠 group_field 在每个组中选择一个函数的最大值的文档。请注意，cscore () 函数可与 "min/max" 选项一起使用，以使用当前正在折叠的文档的分数。  
```
fq={!collapse field=group_field max=sum(cscore(),numeric_field)}
```

使用空策略在 group_field 上折叠，以便在 group_field 中没有值的所有文档都将被视为单个组。对于每个组，所选文档将首先基于 numeric_field，但关系将按分数中断:  
```
fq={!collapse field=group_field nullPolicy=collapse sort='numeric_field asc, score desc'}
```

在 group_field 上折叠，并提示使用顶层字段缓存:  
```
fq={!collapse field=group_field hint=top_fc}
```

CollapsingQParserPlugin 完全支持 QueryElevationComponent。  

## Expand组件

ExpandComponent 可用于展开 CollapsingQParserPlugin 折叠的组。  
CollapsingQParserPlugin的使用示例：  
```
q=foo&amp;fq={!collapse field=ISBN}
```

在上面的查询中，CollapsingQParserPlugin 会折叠 ISBN 字段上的搜索结果。主要搜索结果将包含每本书中排名最高的文档。  
现在可以使用 ExpandComponent 扩展结果，以便可以看到按 ISBN 分组的文档。例如：  
```
q=foo&amp;fq={!collapse field=ISBN}&amp;expand=true
```

“expand = true” 参数打开 ExpandComponent。ExpandComponent 将新节添加到标记为 “expanded” 的搜索输出中。  
在展开的部分中，有一个地图，每个组头都指向组内的展开文档。当应用程序遍历主要折叠结果集时，他们可以访问展开的地图来检索展开的组。  
ExpandComponent 具有以下参数：  
- expand.sort 参数  

        在展开的组中对文档进行排序。默认是<code>score desc</code>。  
    
- expand.rows 参数  

    
        每个组中显示的行数。默认值是5行。  
    
- expand.q 参数  

    
        重写主查询（<code>q</code>），确定要在主组中包含哪些文档。默认是使用主查询。  
    
- expand.fq 参数  

  
        重写主过滤器查询（<code>fq</code>），确定要在主组中包含哪些文档。默认是使用主要的过滤器查询。  
    

