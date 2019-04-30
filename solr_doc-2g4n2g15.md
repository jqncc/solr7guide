## Solr界面管理：流界面 
<div class="content-intro view-box ">Solr 流界面允许您输入<span style="font-size: 14px;">流式表达式</span><span style="font-size: 14px;">并查看结果。它与</span><span style="font-size: 14px;">查询屏幕</span><span style="font-size: 14px;">非常相似，不同的是输入框位于顶部，所有选项必须在表达式中声明。</span>  
  
该界面会将所有内容插入到流式表达式本身，因此您不需要输入带有主机名、端口、集合等的完整 URI。只需在<code>expr=</code> part 之后输入表达式，URL 就会根据需要动态构建。  
在输入框下，执行按钮将运行表达式。“with explanation” 选项将显示已执行的流式表达的各个部分。在此之下，将显示流式结果。也可以在浏览器中查看输出的 URL。  
<p style="text-align: center; ">
    ![solr](https://7n.w3cschool.cn/attachments/image/20171111/1510381514593380.png)  
