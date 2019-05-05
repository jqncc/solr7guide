## 使用ConfigSets API 
<div class="content-intro view-box ">ConfigSets API使您可以创建、删除和以其他方式管理ConfigSets。
      
  
要将使用此API创建的ConfigSet用作集合的配置，请使用Collections API。  
此API只能与在SolrCloud模式下运行的Solr一起使用。如果您没有在SolrCloud模式下运行Solr，但仍希望使用共享配置，请参阅配置集。  

## 配置API入口点

所有API调用的基本URL是http://&lt;hostname&gt;:&lt;port&gt;/solr。  

    - /admin/configs?action=CREATE：根据现有的ConfigSet创建一个ConfigSet
    - /admin/configs?action=DELETE：删除一个ConfigSet
    - /admin/configs?action=LIST：列出所有ConfigSets
    - /admin/configs?action=UPLOAD：上传一个ConfigSet

## 创建一个ConfigSet

/admin/configs?action=CREATE&amp;name=name&amp;baseConfigSet=baseConfigSet  
根据现有的ConfigSet创建一个ConfigSet。  

### 创建ConfigSet参数

创建ConfigSet时支持以下参数。  
- name  
    
        要创建的ConfigSet。该参数是必需的。  
    
- baseConfigSet  
    
        将ConfigSet复制为基础。该参数是必需的。  
    
- configSetProp.name = value  
  
        从基地的任何ConfigSet属性覆盖。  
    

### 创建ConfigSet响应

ConfigSet响应将包括请求的状态。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 创建配置集示例

<b>配置集示例的输入</b>
  
创建一个名为'myConfigSet'的ConfigSet，它基于'predefinedTemplate'ConfigSet，将不可变属性重写为false。  
```
http://localhost:8983/solr/admin/configs?action=CREATE&amp;name=myConfigSet&amp;baseConfigSet=predefinedTemplate&amp;configSetProp.immutable=false
```
<b>该配置集的输出</b>
  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;323&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```

## 删除配置集

/admin/configs?action=DELETE&amp;name=name  
删除配置集  

### 删除ConfigSet参数

- name  
    
        要删除的ConfigSet。该参数是必需的。  
    

### 删除ConfigSet响应

输出将包含请求的状态。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 删除配置集示例

输入如下：  
删除配置集'myConfigSet'  
```
http://localhost:8983/solr/admin/configs?action=DELETE&amp;name=myConfigSet
```
输出如下：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;0&lt;/int&gt;
    &lt;int name="QTime"&gt;170&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```

## 列出配置集

/admin/configs?action=LIST  
获取群集中ConfigSet的名称。  

### 列出配置集示例

输入如下：  
```
http://localhost:8983/solr/admin/configs?action=LIST
```
得到输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":203},
  "configSets":["myConfigSet1",
    "myConfig2"]}
```

## 上传一个配置集

/admin/configs?action=UPLOAD&amp;name=name  
上传一个ConfigSet，作为一个压缩文件发送。请注意，如果启用了身份验证，并且该上传操作作为已验证的请求执行，则会以“trusted”模式上传ConfigSet。没有身份验证，ConfigSet以“untrusted”模式上传。使用“untrusted”的ConfigSet创建集合时，以下功能将不起作用：  

    - 不会初始化RunExecutableListener，如果在ConfigSet中指定的话。
    - DataImportHandler的ScriptTransformer不会初始化，如果在ConfigSet中指定的话。
    - XSLT变换器（tr参数）不能在请求处理时使用。
    - 如果在ConfigSet中指定，StatelessScriptUpdateProcessor不会初始化。

### 上传ConfigSet参数

- name  
    
        上传完成后要创建的ConfigSet。该参数是必需的。  
    

请求的主体应该包含一个压缩配置集。  

### 上传ConfigSet响应

输出将包含请求的状态。如果状态不是“success”，则会显示错误消息，说明请求失败的原因。  

### 上传ConfigSet示例

从压缩文件myconfigset.zip创建一个名为“myConfigSet”的配置集。该压缩文件必须从conf目录中创建（即solrconfig.xml必须是zip文件中的顶层条目）。这里是一个关于如何创建zip文件并上传的例子。  
```
$ (cd solr/server/solr/configsets/sample_techproducts_configs/conf &amp;&amp; zip -r - *) &gt; myconfigset.zip
$ curl -X POST --header "Content-Type:application/octet-stream" --data-binary @myconfigset.zip "http://localhost:8983/solr/admin/configs?action=UPLOAD&amp;name=myConfigSet"
```
使用unix管道可以实现同样的效果，不需要创建一个中间的zip文件，如下所示：  
```
$ (cd server/solr/configsets/sample_techproducts_configs/conf &amp;&amp; zip -r - *) | curl -X POST --header "Content-Type:application/octet-stream" --data-binary @- "http://localhost:8983/solr/admin/configs?action=UPLOAD&amp;name=myConfigSet"
```
