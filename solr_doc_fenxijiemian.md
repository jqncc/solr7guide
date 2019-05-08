# Solr分析界面

通过 Solr 的“分析(Analysis)”界面，您可以根据模式中的字段、字段类型和动态字段配置来检查数据的处理方式。您可以分析在索引期间或在查询处理过程中如何处理内容，以及如何单独或同时查看结果。理想情况下，您需要一致地处理内容，并且此屏幕允许您验证字段类型或字段分析链中的设置。  
  
在顶部的一个或两个框中输入内容，然后选择用于分析的字段或字段类型定义。  
![solr analysis_normal](http://lucene.apache.org/solr/guide/7_0/images/analysis-screen/analysis_normal.png)  
如果单击 “详细输出” 复选框，则您会看到更多信息，其中包括有关输入转换的更多详细信息（例如：转换为小写字母，带额外字符等），其中包括每个阶段的原始字节、类型和详细的位置信息。显示的信息将根据字段或字段类型的设置而变化。该过程的每个步骤都显示在一个单独的部分中，包含了应用在步骤中的标记器或过滤器的缩写。悬停或单击缩写，您将看到标记器或过滤器的名称和路径。

![solr analysis_verbose](http://lucene.apache.org/solr/guide/7_0/images/analysis-screen/analysis_verbose.png)  
在上面的示例截图中，将几个转换应用于输入“Running is a sport”。“is” 和 “a” 这两个词已经被删除，“running” 这个词已更改为其基本形式 "run"。这是因为我们在这个场景中使用了字段类型 text_en，它被配置为删除停止单词（通常不提供大量上下文的小单词）和 "stem" 条件（如果可能的话）以找到更多可能的匹配（这是特别的有助于复数形式的单词）。如果单击“分析字段名称/字段类型（Analyze Fieldname/Field Type）”下拉菜单旁边的问号，将会打开 "模式浏览器" 窗口，显示指定字段的设置。  

“理解分析器（ Understanding Analyzers）”，“标记器（Tokenizers）” 和 “过滤器（Filters）”章节详细描述了每个选项的内容以及它如何转换您的数据以及“运行您的分析程序（ Running Your Analyzer ）”部分有特定的示例来使用分析界面。
