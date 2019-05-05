## Solr支持哪些响应编写器 
<div class="content-intro view-box ">
## 响应编写器

响应编写器生成搜索的格式化的响应。
      
  
Solr 支持各种响应编写器，以确保查询响应可以被适当的语言或应用程序解析。  
该 wt 参数选择要使用的响应编写器。下文中介绍了该 wt 参数最常用的设置，并提供了更多详细讨论这些内容的部分。  

    - CSV
    - GeoJSON
    - javabin
    - JSON
    - PHP
    - PHPS
    - python
          
    
    - ruby
          
    
    - smile
          
    
    - velocity
          
    
    - XLSX
    - XML
    - XSLT


## JSON响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#json-response-writer"/>

默认的 Solr 响应编写器是 JsonResponseWriter，它在 JavaScript 对象表示法 (JSON) 中格式化输出，它是 RFC 4627 中指定的轻量级数据交换格式。如果您在请求中没有设置 wt 参数，则默认情况下将获得 JSON。
      
  
下面是一个简单的查询（q=id:VS1GB400C3）的示例响应：  
```
{
  "responseHeader":{
    "zkConnected":true,
    "status":0,
    "QTime":7,
    "params":{
      "q":"id:VS1GB400C3"}},
  "response":{"numFound":1,"start":0,"maxScore":2.3025851,"docs":[
      {
        "id":"VS1GB400C3",
        "name":["CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail"],
        "manu":["Corsair Microsystems Inc."],
        "manu_id_s":"corsair",
        "cat":["electronics",
          "memory"],
        "price":[74.99],
        "popularity":[7],
        "inStock":[true],
        "store":["37.7752,-100.0232"],
        "manufacturedate_dt":"2006-02-13T15:26:37Z",
        "payloads":["electronics|4.0 memory|2.0"],
        "_version_":1549728120626479104}]
  }}
```

JSON 编写器的默认 MIME 类型是 application/json，但是这可以在 solrconfig.xml 中覆盖，例如在 "techproducts" 配置中的此示例中：  
```
&lt;queryResponseWriter name="json" class="solr.JSONResponseWriter"&gt;
  &lt;!-- For the purposes of the tutorial, JSON response are written as
       plain text so that it's easy to read in *any* browser.
       If you are building applications that consume JSON, just remove
       this override to get the default "application/json" mime type.
    --&gt;
  &lt;str name="content-type"&gt;text/plain&lt;/str&gt;
&lt;/queryResponseWriter&gt;
```


### JSON 特定的参数<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#json-specific-parameters"/>


#### json.nl 参数<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#json-nl"/>

此参数控制 NamedLists 的输出格式，其中顺序比按名称访问更重要。NamedList 当前用于字段分面数据。
      
  
该 json.nl 参数采用以下值：  

    - flat
          
        默认值。NamedList 被表示为平面数组，交替的名称和值。  
        
            如果输入为<code>NamedList("a"=1, "bar"="foo", null=3, null=null)</code>，输出将是<code>["a",1, "bar","foo", null,3, null,null]</code>。  
          
    
    - map
          
        NamedList 被表示为一个 JSON 对象。虽然这是最简单的映射，但 NamedList 可以具有可选键、重复键和保留顺序。为 NamedList 使用 JSON 对象（本质上是一个 map 或者 hash）会导致一些信息的丢失。  
        
            如果输入是<code>NamedList("a"=1, "bar"="foo", null=3, null=null)</code>，则输出将是<code>{"a":1, "bar":"foo", "":3, "":null}</code>。  
          
    
    - arrarr
          
        NamedList 被表示为两个元素数组的数组。  
        
            如果输入为<code>NamedList("a"=1, "bar"="foo", null=3, null=null)</code>，则输出将是<code>[["a",1], ["bar","foo"], [null,3], [null,null]]</code>。  
          
    
    - arrmap
          
        NamedList 被表示为一个 JSON 对象数组。  
        
            如果输入为<code>NamedList("a"=1, "bar"="foo", null=3, null=null)</code>，则输出将是<code>[{"a":1}, {"b":2}, 3, null]</code>。  
          
    
    - arrntv
          
        NamedList 被表示为名称类型值 JSON 对象的数组。  
        
            如果输入为<code>NamedList("a"=1, "bar"="foo", null=3, null=null)</code>，则输出将是<code>[{"name":"a","type":"int","value":1}, {"name":"bar","type":"str","value":"foo"}, {"name":null,"type":"int","value":3}, {"name":null,"type":"null","value":null}]</code>。  
          
    


#### json.wrf 参数

json.wrf=function 在 JSON 响应中添加一个包装函数，在 AJAX 中用于指定 JavaScript 回调函数的动态脚本标记很有用。  

    - [http://www.xml.com/pub/a/2005/12/21/json-dynamic-script-tag.html](http://www.xml.com/pub/a/2005/12/21/json-dynamic-script-tag.html)
    
    - [http://www.theurer.cc/blog/2005/12/15/web-services-json-dump-your-proxy/](http://www.theurer.cc/blog/2005/12/15/web-services-json-dump-your-proxy/)
    


## 标准的XML响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#standard-xml-response-writer"/>

XML 响应编写器是 Solr 当前包含的最通用和可重用的响应编写器。这是大多数关于 Solr 查询响应的讨论和文档中使用的格式。
      
  
请注意，XSLT 响应编写器可用于将此编写器生成的 XML 转换为其他词汇表或基于文本的格式。  
XML 响应编写器的行为可以由以下查询参数驱动。  
- version 参数  

   
        该<code>version</code>参数确定响应中使用的 XML 协议。强烈建议客户始终指定协议版本，以确保如果升级 Solr 服务器并引入新的默认格式，则收到的响应格式不会意外更改。  
        
            目前唯一支持的版本值是<code>2.2</code>。<code>responseHeader</code>的格式更改为使用与响应的其余部分相同的<code>&lt;lst&gt;</code>结构。  
          
        
            默认值是最新支持的。  
          
    
- stylesheet 参数  

        该<code>stylesheet</code>参数可以用来指导 Solr 在它返回的 xml 响应中包含<code>&lt;?xml-stylesheet type="text/xsl" href="…​"?&gt;</code>声明。
              
          

        
            默认行为是不返回任何样式表声明。  
          
不鼓励使用<code>stylesheet</code>参数，因为目前还没有指定外部样式的方法，而且 Solr 发行版中没有提供样式。这是一个传统的参数，可以在将来的版本中进一步发展。
- indent 参数  

 
        如果<code>indent</code>参数被使用，并且有一个非空的值，那么 Solr 会尝试缩进它的 XML 响应，使它更容易被人读取。  
        
            默认行为不是缩进。  
          
    


## XSLT 响应编写器

XSLT 响应编写器将 XML 样式表应用于输出。它可以用于诸如 RSS 提要的格式化结果等任务。
      
  

### tr参数<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#tr-parameter"/>

XSLT 响应编写器接受一个参数：tr 参数，它标识要使用的 XML 转换。该转换必须在 Solr 的 "/xslt 目录" 中找到。
      
  
响应的内容类型根据 XSLT 转换中的 &lt;xsl:output&gt; 语句设置的，例如：&lt;xsl:output media-type="text/html"/&gt;  

### XSLT配置<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#xslt-configuration"/>

下面的示例从 Solr 分布中的 sample_techproducts_configs 配置集中显示了 XSLT 响应编写器的配置方式。  
```
&lt;!--
  Changes to XSLT transforms are taken into account
  every xsltCacheLifetimeSeconds at most.
--&gt;
&lt;queryResponseWriter name="xslt"
                     class="org.apache.solr.request.XSLTResponseWriter"&gt;
  &lt;int name="xsltCacheLifetimeSeconds"&gt;5&lt;/int&gt;
&lt;/queryResponseWriter&gt;
```

值为5的 xsltCacheLifetimeSeconds 对于开发很有好处，可以快速看到 XSLT 的变化。对于生产，您可能需要更高的价值。  

## 二进制响应编写器

这是 Solr 用于节点间通信以及客户端 - 服务器通信的自定义二进制格式。SolrJ 使用此选项作为索引和查询的默认值。请参阅客户端API获取更多详细信息。  

## GeoJSON 响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#geojson-response-writer"/>

返回 Solr 导致在 GeoJSON 增强了特定于 Solr 的 JSON。要使用此功能，请设置 wt=geojson 和 geojson.field 为空间 Solr 的字段的名称。并非所有空间字段类型都受支持，并且如果使用不受支持的类型，则会出现错误。
      
  

## Python响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#python-response-writer"/>

Solr 有一个可选的 Python 响应格式，通过以下方式扩展其 JSON 输出，以允许 python 解释器安全地评估响应：  

    - true 与 false 变为 True 与 False
    - 在需要的地方使用 Python unicode 字符串
    - ASCII 输出（使用 Unicode 转义）用于减少容易出错的互操作性
    - 换行符被转义
    - null 更改为 None


## PHP响应编写器和PHP序列化响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#php-writer"/>

Solr 有一个 PHP 响应格式，输出一个可以评估的数组（可以是 PHP 代码）。设置 wt 参数设置为 php 调用 PHP 响应编写器。  
用法示例：  
```
$code = file_get_contents('http://localhost:8983/solr/techproducts/select?q=iPod&amp;wt=php');
eval("$result = " . $code . ";");
print_r($result);
```

Solr 还包括一个 PHP 序列化响应编写器，它将输出格式化为一个序列化数组。设置 wt 参数为 phps 调用 PHP 序列化响应编写器。  
用法示例：  
```
$serializedResult = file_get_contents('http://localhost:8983/solr/techproducts/select?q=iPod&amp;wt=phps');
$result = unserialize($serializedResult);
print_r($result);
```


## Ruby响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#ruby-response-writer"/>

Solr 有一个可选的 Ruby 响应格式，可以通过以下方式扩展其 JSON 输出，以便 Ruby 的解释器可以安全地评估响应：
      
  

    - Ruby 的单引号字符串被用来防止可能的字符串攻击。
    - \ 和 ' 是唯一的两个字符逃脱。
    - Unicode 转义不被使用。数据是以原始 UTF-8 编写的。
    - 零用于 null。
    - =&gt; 用作映射中的键/值（key/value）分隔符。

下面是一个简单的例子，说明如何使用 Ruby 响应格式来查询 Solr：  
```
require 'net/http'
h = Net::HTTP.new('localhost', 8983)
hresp, data = h.get('/solr/techproducts/select?q=iPod&amp;wt=ruby', nil)
rsp = eval(data)
puts 'number of matches = ' + rsp['response']['numFound'].to_s
#print out the name field for each returned document
rsp['response']['docs'].each { |doc| puts 'name field = ' + doc['name'\] }
```


## CSV响应编写器

CSV响应编写器返回以逗号分隔的值 (CSV) 格式的文档列表。通常包括在响应中的其他信息（如分面信息）不包括在内。  
CSV响应编写器支持多值字段以及伪字段，并且此CSV格式的输出与 Solr 的 CSV 更新格式兼容。  

### CSV参数<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#csv-parameters"/>

这些参数指定将返回的 CSV 格式。您可以接受默认值或指定您自己的。  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">参数</th>
            <th style="text-align: center;">默认值</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">csv.encapsulator  
            </td>
            <td>
                <p style="text-align: center;"><code>"</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.escape  
            </td>
            <td>
                <p style="text-align: center;">没有  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.separator  
            </td>
            <td>
                <p style="text-align: center;"><code>,</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.header  
            </td>
            <td>
                <p style="text-align: center;">默认为<code>true</code>。如果<code>false</code>，Solr 不打印列标题  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.newline  
            </td>
            <td>
                <p style="text-align: center;"><code>\n</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.null  
            </td>
            <td>
                <p style="text-align: center;">默认为零长度的字符串。当文档没有特定字段的值时使用此参数  
            </td>
        </tr>
    </tbody>
</table>

### 多值字段CSV参数

这些参数指定如何对多值字段进行编码。这些值的每字段重写可以使用 f.&lt;fieldname&gt;.csv.separator=|。
      
  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">参数</th>
            <th style="text-align: center;">默认值</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">csv.mv.encapsulator  
            </td>
            <td>
                <p style="text-align: center;">没有  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.mv.escape  
            </td>
            <td>
                <p style="text-align: center;"><code>\</code>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">csv.mv.separator  
            </td>
            <td>
                <p style="text-align: center;">默认为<code>csv.separator</code>值  
            </td>
        </tr>
    </tbody>
</table>

### CSV编写器示例<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#csv-writer-example"/>

例如 http://localhost:8983/solr/techproducts/select?q=ipod&amp;fl=id,cat,name,popularity,price,score&amp;wt=csv 将返回：  
```
id,cat,name,popularity,price,score
IW-02,"electronics,connector",iPod &amp; iPod Mini USB 2.0 Cable,1,11.5,0.98867977
F8V7067-APL-KIT,"electronics,connector",Belkin Mobile Power Cord for iPod w/ Dock,1,19.95,0.6523595
MA147LL/A,"electronics,music",Apple 60 GB iPod with Video Playback Black,10,399.0,0.2446348
```


## Velocity响应编写器

该 VelocityResponseWriter 通过 Apache 速度模板化来处理 Solr 响应和请求上下文。  
有关详细信息，请参阅“速度响应编写器”部分。  

## Smile响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#smile-response-writer"/>

Smile 格式是一个 JSON 兼容的二进制格式，在这里详细描述：http://wiki.fasterxml.com/SmileFormat。  

## XLSX响应编写器<a href="http://lucene.apache.org/solr/guide/7_0/response-writers.html#xlsx-response-writer"/>

使用此项可以将响应作为. xlsx (Microsoft Excel) 格式的电子表格。它接受 colwidth.&lt;field-name&gt; 和 colname.&lt;field-name&gt; 表单中的参数，帮助您自定义列宽和列名。
      
  
这个响应编写器已经被添加为提取库的一部分，并且只有在服务器类路径中存在提取贡献时才起作用。用 lib 指令定义类路径是不够的。相反，您需要将必要的. jar 手动复制到 Solr web 的 lib 目录中。您可以从您的 $SOLR_INSTALL 目录运行这些命令：  
```
cp contrib/extraction/lib/*.jar server/solr-webapp/webapp/WEB-INF/lib/
cp dist/solr-cell-6.3.0.jar server/solr-webapp/webapp/WEB-INF/lib/
```

一旦库就位后，您可以将 wt=xlsx 添加您的请求，结果将作为 XLSX 表单返回。  
