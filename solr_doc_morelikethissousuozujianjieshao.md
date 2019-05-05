## MoreLikeThis搜索组件介绍 
<div class="content-intro view-box ">MoreLikeThis 搜索组件使用户能够查询与结果列表中的文档类似的文档。
      
  
它通过使用原始文档中的术语在索引中查找类似的文档来做到这一点。
      
  
您可以通过三种方法使用 MoreLikeThis 搜索组件。第一个，也是最常见的，就是将它用作请求处理程序。在这种情况下，您可以根据需要将文本发送到 MoreLikeThis 请求处理程序（如用户单击“类似文档”链接时）。  
第二种方法是将其用作搜索组件。这是不太理想的，因为它对每个返回的文档执行 MoreLikeThis 分析，这可能会降低搜索结果。  
最后的方法是将其用作请求处理程序，但使用外部提供的文本。这种情况，也被称为 MoreLikeThisHandler，将根据输入文档的文本提供索引中有关类似文档的信息。  

## MoreLikeThis<span style="font-size: 14px;">是</span>如何工作的<a href="http://lucene.apache.org/solr/guide/7_0/morelikethis.html#how-morelikethis-works"/>

MoreLikeThis 基于文档中的术语构造 Lucene 查询。它通过从定义的字段列表中提取 term 来实现这一点（请参阅下面的 mlt. fl 参数）。为了获得最佳结果，这些字段应该已经在 schema.xml 中存储了词条（term）向量。例如：  
```
&lt;field name="cat" ... termVectors="true" /&gt;
```

如果 term 向量不存储，MoreLikeThis 将从存储的字段中生成 term。为了使 MoreLikeThis 正常工作，还必须存储 uniqueKey。
      
  
下一个阶段使用 MoreLikeThis 参数定义的阈值来过滤来自原始文档的 term。最后，使用这些 term 运行查询，并返回已定义的任何其他查询参数（请参阅下面的 mlt. qf 参数）和新的文档集。  

## MoreLikeThis搜索组件参数<a href="http://lucene.apache.org/solr/guide/7_0/morelikethis.html#common-parameters-for-morelikethis"/>

以下总结了 Lucene/Solr 支持的 MoreLikeThis 参数。这些参数可以与三种可能的 MoreLikeThis 方法中的任何一种一起使用。  
- mlt.fl 参数  

   
        指定用于相似性的字段。如果可能，这些应该已经存储<code>termVectors</code>。  
    
- mlt.mintf 参数  

    
        指定最小 term 频率（Minimum Term Frequency），即在源文档中将忽略哪些 term 的频率。  
    
- mlt.mindf 参数  

    
        指定“最小文档频率”（Minimum Document Frequency），这是词被忽略的频率，至少在很多文档中不会出现。  
    
- mlt.maxdf 参数  

   
        指定“最大文档频率”，这是在多个文档中出现的词被忽略的频率。  
    
- mlt.minwl 参数  

    
        设置单词将被忽略的最小字长。  
    
- mlt.maxwl 参数  

        设置单词将被忽略的最大字长。  
    
- mlt.maxqt 参数  

    
        设置将包含在任何生成的查询中的查询条件的最大数量。  
    
- mlt.maxntp 参数  

        设置未在 TermVector 支持下存储的每个示例文档字段中要解析的最大标记数。  
    
- mlt.boost 参数  

        指定查询是否被有趣的术语“相关性”提升。它可以是“true”或“false”。  
    
- mlt.qf 参数  

        使用与 [DisMax 查询解析器](https://www.w3cschool.cn/solr_doc/solr_doc-vpyf2gn1.html)所使用的格式相同的查询字段及其 boost。这些字段也必须在 mlt. fl 中指定。
              
          
    


## MoreLikeThisComponent的参数<a href="http://lucene.apache.org/solr/guide/7_0/morelikethis.html#parameters-for-the-morelikethiscomponent"/>

使用 MoreLikeThis 作为搜索组件返回响应集中每个文档的类似文档。除了通用参数之外，还可以使用这些附加选项：  
- mlt  

    
        如果设置为<code>true</code>，激活<code>MoreLikeThis</code>组件并使得 Solr 返回<code>MoreLikeThis</code>结果。  
    
- mlt.count  

 
        指定要为每个结果返回的相似文档的数量。默认值是5。  
    


## MoreLikeThisHandler的参数<a href="http://lucene.apache.org/solr/guide/7_0/morelikethis.html#parameters-for-the-morelikethishandler"/>

以下总结了可通过 MoreLikeThisHandler 访问的参数。它支持使用通用查询参数进行分面、分页和过滤，但不能很好地处理备用查询解析器。
      
  
- mlt.match.include  

 
        指定响应是否应包含匹配的文档。如果设置为 false，则响应看起来像正常的选择响应。  
    
- mlt.match.offset  

        指定在主查询搜索结果中的偏移量，以定位 MoreLikeThis 查询应在其上操作的文档。默认情况下，查询对 q 参数的第一个结果进行操作。
              
          
    
- mlt.interestingTerms  

    
        控制<code>MoreLikeThis</code>组件如何为查询提供 "interesting" 术语（顶部TF / IDF术语）。支持三个设置。设置列表中列出了这些 term。设置没有列出任何 term。设置细节列出了 term 以及每个 term 使用的 boost 值。除非<code>mlt.boost=true</code>，否则所有条件都会有<code>boost=1.0</code>。  
    


## <span style="font-family: inherit;">MoreLikeThis</span>查询解析器

mlt 查询解析器提供了一种机制，用于检索类似于给定文档的文档，如处理程序。有关 mlt 查询解析器用法的更多信息，可在其他解析器部分找到。  
