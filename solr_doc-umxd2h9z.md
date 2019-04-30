## Velocity响应编写器 
<div class="content-intro view-box ">VelocityResponseWriter（Velocity响应编写器）是 contrib/velocity 目录中可用的可选插件。当使用诸如 “_default”、“techproducts” 和 “example / files” 等配置时，它为浏览用户界面提供动力。
      
  
必须添加它的 JAR 和依赖项（通过&lt;lib&gt;或 solr/home lib 包含），并且必须在 solrconfig.xml 像这样注册：  
```
&lt;queryResponseWriter name="velocity" class="solr.VelocityResponseWriter"&gt;
  &lt;str name="template.base.dir"&gt;${velocity.template.base.dir:}&lt;/str&gt;
&lt;!--
  &lt;str name="init.properties.file"&gt;velocity-init.properties&lt;/str&gt;
  &lt;bool name="params.resource.loader.enabled"&gt;true&lt;/bool&gt;
  &lt;bool name="solr.resource.loader.enabled"&gt;false&lt;/bool&gt;
  &lt;lst name="tools"&gt;
    &lt;str name="mytool"&gt;com.example.MyCustomTool&lt;/str&gt;
  &lt;/lst&gt;
--&gt;
&lt;/queryResponseWriter&gt;
```

以上示例显示了 VelocityResponseWriter 使用的可选的初始化和自定义工具参数；下文详细介绍了这些内容。这些初始化参数仅在 solrconfig.xml 中的编写器注册中指定，而不是作为请求时间参数。请参阅下面的请求时间参数。
      
  

## 配置和使用


### VelocityResponseWriter 初始化参数

- template.base.dir   

    
        如果指定并作为文件系统目录存在，则将为此目录添加一个文件资源加载程序。此目录中的模板将覆盖 “solr” 资源加载程序模板。
              
          
    
- init.properties.file  

        指定一个属性文件名，必须存在于 Solr 的<code>conf/</code>目录（而不是在<code>velocity/</code>子目录中）或者 &lt;lib&gt; 的 JAR 文件的根中。
              
          
    
- params.resource.loader.enabled  

        “params” 资源加载程序允许在 Solr 请求参数中指定模板。例如：  
    
 
```
http://localhost:8983/solr/gettingstarted/select?q=\*:*&amp;wt=velocity&amp;v.template=custom&amp;v.template.custom=CUSTOM%3A%20%23core_name
```

        
            <code>v.template=custom</code>表示要呈现一个名为“自定义”的模板，其值<code>v.template.custom</code>是自定义模板。默认情况下为<code>false</code>；它不常用，需要时启用。  
          
    
- solr.resource.loader.enabled  

 
        “solr” 资源加载程序是默认注册的唯一模板加载程序。模板是由 SolrResourceLoader 从<code>velocity/</code>子目录下可见的资源提供的。VelocityResponseWriter 本身有一些内置的模板（在它 JAR 文件中的<code>velocity/</code>），这些模板可以通过这个加载程序自动使用。当相同的模板名称处于 conf/velocity/ 或使用<code>template.base.dir</code>选项时，可以覆盖这些内置模板。
              
          
    
- tools  

 
        可以将外部“工具”指定为字符串名称/值（工具名称/类别名称）对的列表。Velocity 上下文中的工具只是 Java 对象。工具类是使用无参数构造函数（或者一个单一的 SolrCore-arg 构造函数，如果存在的话）构造的，并以指定名称添加到 Velocity 上下文中。
              
          
    
        
            自定义注册的工具可以重写具有相同名称的内置上下文对象，除了<code>$request</code>，<code>$response</code>，<code>$page</code>和<code>$debug</code>（这些工具被设计成不能被重写）。  
          
    


### VelocityResponseWriter请求参数

- v.template  

  
        指定要呈现的模板的名称。  
    
- v.layout  

    
        指定一个模板名称，用作围绕主<code>v.template</code>指定模板的布局。
              
          
    
        
            主模板呈现为包含在布局渲染中的字符串值<code>$content</code>。  
          
    
- v.layout.enabled  

  
        确定主模板是否应该有围绕它的布局。默认是<code>true</code>，但也需要指定<code>v.layout</code>。
              
          
    
- v.contentType  

    
        指定 HTTP 响应中使用的内容类型。如果没有指定，默认取决于是否指定<code>v.json</code>。
              
          
 
        
            默认情况下不包含<code>v.json=wrf</code>：<code>text/html;charset=UTF-8</code>。  
          
        
            默认为<code>v.json=wrf</code>：<code>application/json;charset=UTF-8</code>。  
          
    
- v.json  

  
        指定一个函数名称来包装呈现为 JSON 的响应。如果指定，则响应中使用的内容类型将为“application / json; charset = UTF-8”，除非被<code>v.contentType</code>覆盖。
              
          
    
        
            输出将采用以下格式（带<code>v.json=wrf</code>）：  
          
        
```
wrf("result":"&lt;Velocity generated response string, with quotes and backslashes escaped&gt;")
```

          
    
- v.locale  

  
        使用<code>$resource</code>工具和其他 LocaleConfig 实现工具的语言环境。默认语言环境是<code>Locale.ROOT</code>。本地化资源从名为<code>resources[_locale-code].properties</code>的标准 Java 资源包中加载
              
          
   
        
            可以通过提供由 SolrResourceLoader 在速度子下的资源包可见的 JAR 文件来添加资源包。资源包不能在<code>conf/</code>下加载，因为只有 SolrResourceLoader 的类加载程序方面可以在这里使用。  
          
    
- v.template.template_name  

        当启用 “params” 资源加载程序时，可以将模板指定为 Solr 请求的一部分。  
    


### VelocityResponseWriter上下文对象

<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">上下文参考</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>request</code>
                  
            </td>
            <td>
                <p style="text-align: center;">SolrQueryRequest javadocs  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>response</code>
                  
            </td>
            <td>
                <p style="text-align: left;">QueryResponse；大多数情况下，但在某些情况下，QueryResponse 不喜欢请求处理程序的输出（例如 AnalysisRequestHandler 会导致 ClassCastException 解析“响应”），则响应将是 SolrResponseBase 对象。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>esc</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity EscapeTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>date</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity ComparisonDateTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>list</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity ListTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>math</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity MathTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>number</code>
                  
            </td>
            <td>
                <p style="text-align: center;">Velocity NumberTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>sort</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity SortTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>display</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity DisplayTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>resource</code>
                  
            </td>
            <td>
                <p style="text-align: center;">一个 Velocity ResourceTool 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>engine</code>
                  
            </td>
            <td>
                <p style="text-align: center;">当前的 VelocityEngine 实例  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>page</code>
                  
            </td>
            <td>
                <p style="text-align: center;">Solr 的 PageTool 的一个实例（只有当响应是 QueryResponse 时，分页才有意义）  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>debug</code>
                  
            </td>
            <td>
                <p style="text-align: center;">响应的调试部分的快捷方式，如果调试不在，则为null。这对于使用<code>#if($debug)…​#end</code>的模板中的仅调试节非常方便
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>content</code>
                  
            </td>
            <td>
                <p style="text-align: left;">呈现布局（<code>v.layout.enabled=true</code>和<code>v.layout=&lt;template&gt;</code>）时主模板的呈现输出。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">[自定义工具]  
            </td>
            <td>
                <p style="text-align: center;">由 VelocityResponseWriter 注册的可选“工具”列表提供的工具可通过指定的名称获得。  
</td></tr></tbody></table>
