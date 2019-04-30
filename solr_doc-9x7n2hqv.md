## Solr更新请求处理器 
<div class="content-intro view-box ">Solr收到的每个更新请求都通过一个称为更新请求处理器或URP的一系列插件运行。
      
  
例如，可以将字段添加到正在索引的文档中，这是很有用的；改变特定字段的值; 或者如果传入文档不符合某些标准，则删除更新。实际上，Solr中出现了大量的特性被实现为更新处理器，因此有必要了解这些插件是如何工作的以及它们是如何配置的。  

## URP分析和生命周期

更新请求处理器是作为一个或多个更新处理器链的一部分创建的。Solr创建了一个默认的更新请求处理器链，其中包含一些更新请求处理器，这些处理器支持基本的Solr功能。除非用户选择配置和指定不同的自定义更新请求处理器链，否则此默认链用于处理每个更新请求。
      
  
描述更新请求处理器最简单的方法是查看抽象类UpdateRequestProcessor的Javadoc 。每个UpdateRequestProcessor必须有一个相应的factory类，它扩展了UpdateRequestProcessorFactory。这个factory类被Solr用来创建这个插件的一个新实例。这样的设计提供了以下两个好处：  
1 <li>更新请求处理器不需要是线程安全的，因为它只被一个请求线程使用，并且一旦请求完成就被销毁。</li>2 <li>factory类可以接受配置参数并维护请求之间可能需要的任何状态。工厂类必须是线程安全的。</li>
每个更新请求处理器链都是在加载Solr核心期间被构建，并被缓存直到核心被卸载。在链中指定的每个 UpdateRequestProcessorFactory 也会以 solrconfig. xml 中指定的配置进行实例化和初始化。  
当Solr收到更新请求时，它会查找用于此请求的更新链。在链中指定的每个UpdateRequestProcessor的新实例都使用相应的factory创建。更新请求被解析成相应的UpdateCommand对象，这些对象通过链接运行。每个UpdateRequestProcessor实例负责调用链中的下一个插件。它可以通过不调用下一个处理器来选择使链路短路，甚至通过抛出异常来中止进一步的处理。  
注意：单个更新请求可能包含多个新文档或删除的批次，因此，对于每个单独的更新，UpdateRequestProcessor 的相应 processXXX 方法将被多次调用。但是，保证单个线程将依次调用这些方法。  

## 更新请求处理器配置

更新请求处理器链可以通过在solrconfig.xml中直接创建整个链，或通过在solrconfig.xml中创建单独的更新处理器来创建，然后通过请求参数指定所有处理器，在运行时动态创建链。
      
  
但是，在我们了解如何配置更新处理器链之前，我们必须了解默认更新处理器链，因为它提供了大多数自定义请求处理器链中所需的基本功能。  

### 默认更新请求处理器链

如果没有在solrconfig.xml配置更新处理器链，Solr将自动创建一个默认的更新处理器链，将用于所有更新请求。此默认更新处理器链由以下处理器组成（按顺序）：
      
  
1 <li>LogUpdateProcessorFactory - 跟踪在这个请求期间处理的命令并且记录它们</li>2 <li>DistributedUpdateProcessorFactory - 负责将更新请求分发到正确的节点，例如，将请求路由到右侧分片的领导者，并将更新从领导者分发到每个副本。该处理器仅在SolrCloud模式下激活。</li>3 <li>RunUpdateProcessorFactory - 使用内部Solr API执行更新。</li>
它们中的每一个都执行基本的功能，因此任何定制链通常包含所有这些处理器。RunUpdateProcessorFactory通常是任何自定义链中的最后一次更新的处理器。  

### 自定义更新请求处理器链

以下示例演示了如何在solrconfig.xml内部配置自定义链。  
重复数据删除updateRequestProcessorChain示例：  
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

在上述示例中，一个名为“重复数据删除”的新的更新处理器链是使用SignatureUpdateProcessorFactory、LogUpdateProcessorFactory 和 RunUpdateProcessorFactory在链中创建的。SignatureUpdateProcessorFactory 进一步配置了不同的参数，如“signatureField”，“overwriteDupes”等，此链是一个例子，说明 Solr 如何配置，可以通过使用名称、功能和 cat 字段的值来计算签名，然后将其用作
    "id" 字段来执行文档重复数据消除。正如您可能已经注意到的，这个链没有指定 DistributedUpdateProcessorFactory。因为这个处理器对于Solr正常运行至关重要，所以Solr会自动在任何链中插入DistributedUpdateProcessorFactory，而不包括在 RunUpdateProcessorFactory 之前。
      
  
RunUpdateProcessorFactory：请不要忘记在您在 solrconfig. xml 中定义的任何链的末尾添加 RunUpdateProcessorFactory。否则，由该链处理的更新请求将不会实际影响索引数据。  

### 将各个处理器配置为顶级插件

更新请求处理器也可以独立于 solrconfig. xml 中的链进行配置。
      
  
updateProcessor 配置：  
```
&lt;updateProcessor class="solr.processor.SignatureUpdateProcessorFactory" name="signature"&gt;
  &lt;bool name="enabled"&gt;true&lt;/bool&gt;
  &lt;str name="signatureField"&gt;id&lt;/str&gt;
  &lt;bool name="overwriteDupes"&gt;false&lt;/bool&gt;
  &lt;str name="fields"&gt;name,features,cat&lt;/str&gt;
  &lt;str name="signatureClass"&gt;solr.processor.Lookup3Signature&lt;/str&gt;
&lt;/updateProcessor&gt;
&lt;updateProcessor class="solr.RemoveBlankFieldUpdateProcessorFactory" name="remove_blanks"/&gt;
```

在这种情况下，SignatureUpdateProcessorFactory的一个实例被配置为名称“signature” ，而 RemoveBlankFieldUpdateProcessorFactory 则用名称 "remove_blanks" 定义。一旦在solrconfig.xml中指定了上述内容，我们可以在solrconfig.xml更新请求处理器链中引用它们，如下所示：
      
  
updateRequestProcessorChain 配置：  
```
&lt;updateProcessorChain name="custom" processor="remove_blanks,signature"&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateProcessorChain&gt;
```


## 更新SolrCloud中的处理器

在一个独立的Solr节点中，每次更新都会在链中的所有更新处理器中运行一次。但是SolrCloud中更新请求处理器的行为值得特别考虑。
      
  
关键的SolrCloud功能是请求的路由和分配。对于更新请求，此路由由 DistributedUpdateRequestProcessor 实现，并且由于它的重要功能，这个处理器被Solr给予一个特殊的状态。  
在SolrCloud模式，在 DistributedUpdateProcessor 之前链中的所有处理器都在从客户端接收更新的第一个节点上运行，而不管该节点作为前导或副本的状态如何。DistributedUpdateProcessor随后将更新转发给相应的碎片领导者（或在影响多个文档（如通过查询或提交来删除）更新的情况下，向多个领导发送）。碎片领导使用事务日志来应用原子更新和开放式并发，然后将更新转发给所有碎片副本。领导者和每个副本运行链中列出的 DistributedUpdateProcessor
    之后的所有处理器。  
例如，考虑我们在上面一节中看到的“重复数据删除”链。假设存在一个3节点的SolrCloud集群，其中节点A承载shard1的引导，节点B承载shard2的引导，节点C承载shard2的副本。假设更新请求被发送到节点A，节点A将更新转发到节点B（因为更新属于shard2），然后将更新分发到其副本节点C。让我们看看在每个节点处发生了什么：  

    - 节点A：通过 SignatureUpdateProcessor（它计算签名并将其放入“ID”字段）运行更新，然后是LogUpdateProcessor，再然后DistributedUpdateProcessor。该处理器确定更新实际上属于节点B并被转发到节点B。更新不被进一步处理。这是必需的，因为下一个处理器 RunUpdateProcessor 将针对本地 shard1 索引执行更新，这将导致 shard1 和 shard2 上的重复数据。
    - 节点B：接收更新，并且看到它被另一个节点转发。更新是直接发送到DistributedUpdateProcessor的，因为它已经通过节点A上的SignatureUpdateProcessor，并且再次执行相同的签名计算将是多余的。DistributedUpdateProcessor确定更新确实属于该节点，并将其分发到节点C上的副本，然后将该更新转发到链中的 RunUpdateProcessor。
    - 节点C：接收更新，并且看到它是由其领导者分发的。该更新直接发送到 DistributedUpdateProcessor，它执行一些一致性检查，并将更新在链中进一步转发到 RunUpdateProcessor。

综上所述：
      
  
1 <li>DistributedUpdateProcessor 之前的所有处理器只在接收更新请求的第一个节点上运行，不管它是转发节点（例如，上述示例中的节点A）还是一个引导（例如节点B）。我们称之为“预处理器”或“处理器”。</li>2 <li>DistributedUpdateProcessor 后的所有处理器只在引线和副本节点上运行。它们不在转发节点上执行。这样的处理器被称为“后处理器”。</li>
在前一节中，我们看到了updateRequestProcessorChain 配置了processor="remove_blanks, signature"。这意味着这样的处理器是 #1 类型的，只能在转发节点上运行。同样，我们可以通过指定属性“后处理器”来将它们配置为＃2类型，如下所示：
      
  
后处理器（post-processors）配置：  
```
&lt;updateProcessorChain name="custom" processor="signature" post-processor="remove_blanks"&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
&lt;/updateProcessorChain&gt;
```

然而，只在转发节点上执行处理器是一种很好的方法，它通过负载平衡器随机发送请求，从而在SolrCloud集群上分配昂贵的计算（如重复数据删除）。否则，会在领导节点和复制节点上重复进行昂贵的计算。
      
  
不能在恢复的复制副本上调用自定义更新链post-processors：当副本处于恢复状态时，入站更新请求将缓冲到事务日志中。成功完成恢复后，将重播那些缓冲的更新请求。但是，在编写这篇文章时，从不为缓冲的更新请求调用自定义更新链post-processors，见SOLR-8030。要解决此问题，直到SOLR-8030被修复，请避免在自定义更新链中指定post-processors。  

### 原子更新处理器工厂

如果AtomicUpdateProcessorFactory是在 DistributedUpdateProcessor 之前的更新链中，则传入的文档链将是部分文档。
      
  
由于DistributedUpdateProcessor负责将原子更新处理成领导者节点上的完整文档，这意味着仅在转发节点上执行的预处理器只能对部分文档进行操作。如果您有一个必须处理完整文档的处理器，那么唯一的选择就是将其指定为post-processors。  

## 使用自定义链


### update.chain请求参数

该update.chain参数可用于任何更新请求中以选择已在 solrconfig. xml 中配置的自定义链。例如，为了选择上一节中描述的“重复数据删除”链，可以发出以下请求：
      
  
使用update.chain：  
```
curl "http://localhost:8983/solr/gettingstarted/update/json?update.chain=dedupe&amp;commit=true" -H 'Content-type: application/json' -d '
[
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  },
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  }
]'
```

上面应该重复删除两个相同的文件，并且只索引其中的一个。  

### 处理器和post-processors请求参数

我们可以使用处理器和post-processor请求参数动态地构建自定义更新请求处理器链。可以将多个处理器指定为这两个参数的逗号分隔值。例如：
      
  
执行在solrconfig.xml中配置的处理器，比如：(pre)-processors  
```
curl "http://localhost:8983/solr/gettingstarted/update/json?processor=remove_blanks,signature&amp;commit=true" -H 'Content-type: application/json' -d '
[
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  },
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  }
]'
```

将solrconfig.xml中配置的处理器作为处理前和处理后执行：  
```
curl "http://localhost:8983/solr/gettingstarted/update/json?processor=remove_blanks&amp;post-processor=signature&amp;commit=true" -H 'Content-type: application/json' -d '
[
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  },
  {
    "name" : "The Lightning Thief",
    "features" : "This is just a test",
    "cat" : ["book","hardcover"]
  }
]'
```

在第一个例子中，Solr将动态地创建一个具有“signature”和“remove_blanks”的链作为预处理器，仅在转发节点上执行，而在第二个例子中，“remove_blanks”将作为预处理器并且“signature”将作为post-processor在领导者和副本上执行。
      
  

### 将自定义链配置为默认值

我们还可以指定一个自定义链，默认情况下用于发送到特定更新处理程序的所有请求，而不是在每个请求的请求参数中指定名称。
      
  
这可以通过添加“update.chain”或“processor”和“post-processor”作为给定路径的默认参数，可以通过 initParams 或通过在所有请求处理程序支持的 "default s" 部分中添加来完成。  
以下initParam是在无模式配置中定义的，它将自定义更新链应用于以“/update/”开头的所有请求处理程序。  
initParams示例：  
```
&lt;initParams path="/update/**"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;add-unknown-fields-to-the-schema&lt;/str&gt;
  &lt;/lst&gt;
&lt;/initParams&gt;
```

或者，可以使用下面的示例中显示的“defaults”来实现类似的效果：  
defaults示例：  
```
&lt;requestHandler name="/update/extract" startup="lazy" class="solr.extraction.ExtractingRequestHandler" &gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;add-unknown-fields-to-the-schema&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
```


## 更新请求处理器工厂

以下是对当前可用更新请求处理器的简要说明。一个UpdateRequestProcessorFactory可以根据需要集成到 solrconfig. xml 中的更新链中。强烈要求您检查这些类的Javadocs；这些描述是大部分取自Javadocs摘录的片段。  

### 一般使用 UpdateProcessorFactory

- AddSchemaFieldsUpdateProcessorFactory  

  
        如果输入文档包含一个或多个与模式中的任何字段或动态字段不匹配的字段，则该处理器将动态地向该模式添加字段。  
    
- AtomicUpdateProcessorFactory  

    
        该处理器将传统的字段值文档转换为原子更新文档。  
    
- ClassificationUpdateProcessorFactory  

    
        这个处理器使用Lucene的分类模块来提供简单的文档分类。有关如何使用此处理器的更多详细信息，请参阅https://wiki.apache.org/solr/SolrClassification。  
    
- CloneFieldUpdateProcessorFactory  

   
        将在任何匹配的源字段中找到的值克隆到已配置的dest字段中。  
    
- DefaultValueUpdateProcessorFactory  

   
        一个简单的处理器，它将一个默认值添加到任何在fieldName中没有值的文档。  
    
- DocBasedVersionConstraintsProcessorFactory  

    
        该Factory生成一个UpdateProcessor，该UpdateProcessor帮助使用版本字段的已配置名称，基于每个文档的版本号对文档实施版本限制。  
    
- DocExpirationUpdateProcessorFactory  

    
        更新处理器工厂，用于管理文档的自动“expiration”。  
    
- FieldNameMutatingUpdateProcessorFactory  

    
        通过用配置的<code>replacement</code>替换与配置的<code>pattern</code>匹配的所有匹配项来修改字段名称。  
    
- IgnoreCommitOptimizeUpdateProcessorFactory  

    
        允许您在SolrCloud模式下运行时忽略提交或优化来自客户端应用程序的请求，有关详细信息，请参阅：SolrCloud中的碎片和索引数据  
   
- RegexpBoostProcessorFactory  

    
        一个处理器将匹配“inputField”的内容与在“boostFilename”中找到的正则表达式匹配，如果它匹配，则将从文件中返回相应的boost值，并将其作为double值输出到“boostField”。
              
          
    
- SignatureUpdateProcessorFactory  

    
        使用一组定义的字段为文档生成哈希“signature”。用于索引“similar”文档的一个副本。  
    
- StatelessScriptUpdateProcessorFactory  

   
        更新请求处理器工厂，允许使用以脚本实现的更新处理器。  
    
- TimestampUpdateProcessorFactory  

 
        更新处理器，将新生成的日期值“NOW”添加到正在添加的文档中，该文档在指定的字段中没有值。  
    
- URLClassifyProcessorFactory  

    
        更新处理器，检查URL并输出到具有该URL特征的其他各个字段，包括长度，路径级别数量，是否是顶级URL（levels==0），是否看起来像登录/索引页面， URL的标准表示（例如，index.html），URL的域和路径部分等。  
    
- UUIDUpdateProcessorFactory  

   
        更新处理器，将新生成的UUID值添加到正在添加的文档中，该文档在指定的字段中没有值。  
    


### FieldMutatingUpdateProcessorFactory派生Factory

这些Factory都提供了修改文档中的字段的功能。当使用这些Factory时，请参考FieldMutatingUpdateProcessorFactory的javadocs，了解它们都支持配置哪些字段被修改的常用选项。  
- ConcatFieldUpdateProcessorFactory  

    
        使用可配置的分隔符连接多个值，以匹配指定条件的字段。  
   
- CountFieldValuesUpdateProcessorFactory  

  
        用与该字段的值的数量相同的值替换与指定的条件匹配的字段的值的任何列表。  
   
- FieldLengthUpdateProcessorFactory  

   
        使用这些CharSequences的长度（作为整数）替换在指定条件中找到的所有CharSequence值。  
    
- FirstFieldValueUpdateProcessorFactory  

    
        只保留符合指定条件的字段的第一个值。  
    
- HTMLStripFieldUpdateProcessorFactory  

  
        在符合指定条件的字段中找到的任何CharSequence值中剥离所有HTML标记。  
   
- IgnoreFieldUpdateProcessorFactory  

    
        忽略并删除正在添加到索引的任何文档中符合指定条件的字段。  
    
- LastFieldValueUpdateProcessorFactory  

    
        只保留匹配指定条件的字段的最后一个值。  
    
- MaxFieldValueUpdateProcessorFactory  

 
        更新处理器，只保留找到多个值的任何选定字段的最大值。  
    
- MinFieldValueUpdateProcessorFactory  

   
        更新处理器，只保留找到多个值的任何选定字段的最小值。  
    
- ParseBooleanFieldUpdateProcessorFactory  

    
        尝试将只有CharSequence类型值的选定字段变为布尔值。  
   
- ParseDateFieldUpdateProcessorFactory  

    
        尝试将仅具有CharSequence类型值的选定字段变为Solr日期值。  
    
- ParseNumericFieldUpdateProcessorFactory派生类  

    
        
            <ul><li>ParseDoubleFieldUpdateProcessorFactory  

                
                    尝试将具有仅CharSequence类型值的选定字段变为Double值。  
                
- ParseFloatFieldUpdateProcessorFactory  

                
                    尝试将具有仅CharSequence类型的值的选定字段变为Float值。  
                
- ParseIntFieldUpdateProcessorFactory  

                
                    尝试将仅具有CharSequence类型值的选定字段变为Integer值。  
               
- ParseLongFieldUpdateProcessorFactory  

                
                    尝试将具有仅CharSequence类型的值的选定字段变为Long值。  
                
            
          
    </li><li>PreAnalyzedUpdateProcessorFactory  

    
        更新处理器，用配置的格式解析器分析使用PreAnalyzedField添加的任何文档的配置字段。  
    </li><li>RegexReplaceProcessorFactory  

    
        更新后的处理器，将配置的正则表达式应用于在所选字段中找到的任何CharSequence值，并用配置的替换字符串替换任何匹配项。  
   </li><li>RemoveBlankFieldUpdateProcessorFactory  

    
        删除找到的长度为0的CharSequence的任何值（即：空字符串）。  
    </li><li>TrimFieldUpdateProcessorFactory  

    
        从匹配指定条件的字段中找到的任何CharSequence值修剪前导和尾随空白。  
    </li><li>TruncateFieldUpdateProcessorFactory  

    
        将匹配指定条件的字段中的所有CharSequence值截断为最大字符长度。  
    </li><li>UniqFieldsUpdateProcessorFactory  

    
        删除在符合指定条件的字段中找到的重复值。  
    </li>
</ul>

### 更新可作为插件加载的处理器Factory

这些处理器作为“contribs”包含在Solr发行版中，并且需要在运行时加载额外的jar。有关详细信息，请参阅与每个contrib关联的README文件：  
- 该<code>langid</code>contrib 提供  

    
        
            <ul><li>LangDetectLanguageIdentifierUpdateProcessorFactory  

                
                    使用http://code.google.com/p/language-detection标识一组输入字段的语言。  
                
- TikaLanguageIdentifierUpdateProcessorFactory  

               
                    使用Tika的LanguageIdentifier标识一组输入字段的语言。  
                
            
          
    </li><li>该<code>uima</code>contrib请提供  

   
        
            - UIMAUpdateRequestProcessorFactory  

               
                    使用UIMA提取的信息更新要编入索引的文档。  
                
            
          
    </li>
</ul>

### 更新处理器工厂，您不应该修改或删除

这些是为完整性而列出的，但是是Solr基础设施的一部分，特别是SolrCloud。除了确保在修改更新请求处理程序（或您创建的任何副本）时不要删除它们，您将很少需要更改这些处理程序。
      
  
- DistributedUpdateProcessorFactory  

  
        用于将更新分发到所有必需的节点。  
        
            <ul><li>NoOpDistributingUpdateProcessorFactory  

                
                    一个<code>DistributingUpdateProcessorFactory</code>的替代的No-Op实现，总是返回null。专为希望绕过分布式更新并使用自己的自定义更新逻辑的用户而设计。  
                
            
          
    </li><li>LogUpdateProcessorFactory  

    
        记录处理器。这将跟踪所有已经通过链的命令，并在finish（）上打印它们。  
    </li><li>RunUpdateProcessorFactory  

    
        使用底层的UpdateHandler执行更新命令。几乎所有的处理器链都应该以一个<code>RunUpdateProcessorFactory</code>实例结束，除非用户在另一个自定义<code>UpdateRequestProcessorFactory</code>中显式执行更新命令。  
    </li>
</ul>

### 更新可以在运行时使用的处理器

这些更新处理器不需要任何配置，就是您的solrconfig.xml。当它们的名字被添加到processor参数时，它们会自动初始化。通过附加多个处理器名称可以使用多个处理器（用逗号分隔）
      
  

#### TemplateUpdateProcessorFactory

该TemplateUpdateProcessorFactory可用于新的字段添加到基于模板的格式的文档的。  
使用参数processor=template来使用它。模板参数template.field（多值）定义要添加的字段和模式。模板可能包含引用文档中其他字段的占位符。您可以在一个请求中使用多个Template.field参数。  
例如：  
```
processor=template&amp;template.field=fullName:Mr. {firstName} {lastName}
```

上面的例子会在文档中添加一个新的字段fullName。这些字段firstName 和 lastName是从文档字段提供的。如果其中任何一个丢失，则该部分将被替换为空字符串。如果这些字段是多值的，则只使用第一个值。  

#### AtomicUpdateProcessorFactory

处理器的名字是atomic。使用它将您的正常update操作转换为原子更新操作。当您使用诸如/update/csv或/update/json/docs不支持原子操作语法的端点时，这一点特别有用。  
例如：  
```
processor=atomic&amp;atomic.field1=add&amp;atomic.field2=set&amp;atomic.field3=inc&amp;atomic.field4=remove&amp;atomic.field4=remove
```

以上参数转换为正常update操作  

    - field1到原子add操作
    - field2到原子set操作
    - field3到原子inc操作
    - field4到原子remove操作

