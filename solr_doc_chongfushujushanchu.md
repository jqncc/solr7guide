## Solr重复数据删除 
<div class="content-intro view-box ">如果索引中存在重复或接近重复的文档，则可能需要执行重复数据删除。  
  
防止重复或接近重复的文档进入索引或用签名/指纹标记具有重复字段折叠的文档可以用低冲突或模糊哈希算法有效地实现。Solr 通过 Signature 类本地支持这种类型的重复数据删除技术，并允许轻松添加新的哈希/签名实现。签名可以通过几种方式实现：  

    - MD5Signature：用于精确重复检测的 128 位哈希。
    - Lookup3Signature：用于精确重复检测的 64 位哈希。这比 MD5 快得多，索引也小一些。
    - TextProfileSignature：来自 Apache Nutch 的模糊哈希实现，用于接近重复的检测。这是可调的，但在较长的文本上效果最好。

其他更复杂的模糊/近似哈希算法可以稍后添加。  
注意：在重复数据删除过程中添加将更改 allowDups 设置，使其适用于更新术语 (在本例中为 signatureField)，而不是唯一的字段术语。当然，signatureField 可以是唯一的字段，但通常您希望唯一字段是独一无二的。添加文档时，将自动生成签名并将其附加到指定 signatureField 中的文档。  

## 配置重复数据删除选项

Solr 中有两个地方可以配置重复数据删除：in solrconfig.xml 和 in schema.xml。  

### 在 solrconfig.xml 中

SignatureUpdateProcessorFactory 必须被登记在 solrconfig.xml 中作为更新请求处理器链的一部分，如在这个例子中：  
```
&lt;updateRequestProcessorChain name="dedupe"&gt;
  &lt;processor class="solr.processor.SignatureUpdateProcessorFactory"&gt;
    &lt;bool name="enabled"&gt;true&lt;/bool&gt;
    &lt;str name="signatureField"&gt;id&lt;/str&gt;
    &lt;bool name="overwriteDupes"&gt;false&lt;/bool&gt;
    &lt;str name="fields"&gt;name,features,cat&lt;/str&gt;
    &lt;str name="signatureClass"&gt;solr.processor.Lookup3Signature&lt;/str&gt;
  &lt;/processor&gt;
  &lt;processor class="solr.LogUpdateProcessorFactory" /&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateRequestProcessorChain&gt;
```
这 SignatureUpdateProcessorFactory 需要以下几个属性：  

**signatureClass**
    
        用于生成签名哈希的签名实现。默认是<code>org.apache.solr.update.processor.Lookup3Signature</code>。  
        
            必须指定实现的完整类路径。上面介绍了可用的选项，要使用的关联类路径是：  
          
        
            
                - 
                    <code>org.apache.solr.update.processor.Lookup3Signature</code>
                      
                
                - 
                    <code>org.apache.solr.update.processor.MD5Signature</code>
                      
                
                - 
                    <code>org.apache.solr.update.process.TextProfileSignature</code>
                      
                
            
          
    
<dt>fields  
</dt>
    
        用于在逗号分隔列表中生成签名哈希的字段。默认情况下，将使用文档上的所有字段。  
    
**signatureField**
    
        用于保存指纹/签名的字段的名称。该字段应该被定义<code>schema.xml</code>。默认是<code>signatureField</code>。  
    
<dt>enabled  
</dt>
    
        设置为 false 可禁用重复数据删除处理。默认值是 true。  
    
**overwriteDupes**
    
        如果为 true，则默认情况下，当文档已经与此签名匹配时，它将被覆盖。  
    


### 在 schema.xml 中

如果您使用单独的字段来存储签名，则必须将其编入索引：  
```
&lt;field name="signatureField" type="string" stored="true" indexed="true" multiValued="false" /&gt;
```
请确保更改您的更新处理程序以使用定义的链，如下所示：  
```
&lt;requestHandler name="/update" class="solr.UpdateRequestHandler" &gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;dedupe&lt;/str&gt;
  &lt;/lst&gt;
...
&lt;/requestHandler&gt;
```
上述例子假设你已经定义了请求处理程序的其他部分。  
还可以使用参数 update.chain=dedupe 为每个请求指定更新处理器。  
