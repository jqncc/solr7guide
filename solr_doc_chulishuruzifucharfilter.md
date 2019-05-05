## Solr处理输入字符：CharFilter 
<div class="content-intro view-box ">
## 什么是 CharFilter
CharFilter 是预处理输入字符的组件。  
CharFilter可以像 Token Filters 一样链接在一个 Tokenizer 前面。  
CharFilters 可以添加、更改或删除字符，同时保留原始字符偏移以支持突出显示等功能。  
本节将介绍如下几个过滤器：  
- solr.MappingCharFilterFactory
- solr.HTMLStripCharFilterFactory
- solr.ICUNormalizer2CharFilterFactory
- solr.PatternReplaceCharFilterFactory
### solr.MappingCharFilterFactory
此过滤器创建 org.apache.lucene.analysis.MappingCharFilter，可用于将一个字符串更改为另一个（例如，用于标准化 é 为 e。）。  
  
此过滤器需要指定 mapping 参数，该参数是包含要执行的映射的文件的路径和名称。  
例：  
```
&lt;analyzer&gt;
  &lt;charFilter class="solr.MappingCharFilterFactory" mapping="mapping-FoldToASCII.txt"/&gt;
  &lt;tokenizer ...&gt;
  [...]
&lt;/analyzer&gt;
```
映射文件语法：  
  
- 以哈希标记（#）开头的注释行以及空白行将被忽略。
- 每个非注释，非空白行由以下形式的映射组成： "source" =&gt; "target"，即：双引号的源字符串、可选的空格、箭头（=&gt;）、可选的空格、双引号的目标字符串。
- 不允许对映射行使用尾随注释。
- 源字符串必须至少包含一个字符，但目标字符串可能为空。
- 以下字符转义序列在源和目标字符串中被识别：<table class=""><thead><tr><th style="text-align: center;">转义序列</th><th style="text-align: center;">结果字符（[ECMA-48](http://www.ecma-international.org/publications/standards/Ecma-048.htm)别名）</th><th style="text-align: center;">Unicode字符</th><th style="text-align: center;">示例映射线</th></tr></thead><tbody><tr><td><p style="text-align: center;"><code>\\</code>  
</td><td><p style="text-align: center;"><code>\</code>  
</td><td><p style="text-align: center;">U + 005C  
</td><td><p style="text-align: center;"><code>"\\" =&gt; "/"</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\"</code>  
</td><td><p style="text-align: center;"><code>"</code>  
</td><td><p style="text-align: center;">U + 0022  
</td><td><p style="text-align: center;"><code>"\"and\"" =&gt; "'and'"</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\b</code>  
</td><td><p style="text-align: center;">退格（BS）  
</td><td><p style="text-align: center;">U + 0008  
</td><td><p style="text-align: center;"><code>"\b" =&gt; " "</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\t</code>  
</td><td><p style="text-align: center;">标签（HT）  
</td><td><p style="text-align: center;">U + 0009  
</td><td><p style="text-align: center;"><code>"\t" =&gt; ","</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\n</code>  
</td><td><p style="text-align: center;">换行（LF）  
</td><td><p style="text-align: center;">U + 000A  
</td><td><p style="text-align: center;"><code>"\n" =&gt; "&lt;br&gt;"</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\f</code>  
</td><td><p style="text-align: center;">换页（FF）  
</td><td><p style="text-align: center;">U + 000C  
</td><td><p style="text-align: center;"><code>"\f" =&gt; "\n"</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\r</code>  
</td><td><p style="text-align: center;">回车（CR）  
</td><td><p style="text-align: center;">U + 000D  
</td><td><p style="text-align: center;"><code>"\r" =&gt; "/carriage-return/"</code>  
</td></tr><tr><td><p style="text-align: center;"><code>\uXXXX</code>  
</td><td>由4个十六进制数字引用的 Unicode 字符  
</td><td><p style="text-align: center;">U + XXXX  
</td><td><p style="text-align: center; "><code>"\uFEFF" =&gt; ""</code>  
</td></tr></tbody></table>任何其他字符之后的反斜杠被解释为如果字符不存在反斜线。后面跟着其他字符的反斜杠被解释为没有反斜杠的字符。
### solr.HTMLStripCharFilterFactory
这个过滤器创建 org.apache.solr.analysis.HTMLStripCharFilter。此 CharFilter 从输入流中剥离 HTML 并将结果传递给另一个 CharFilter 或 Tokenizer。  
这个过滤器：  
- 删除HTML / XML标记，同时保留其他内容。
- 删除标签中的属性并支持可选的属性引用。
- 删除 XML 处理指令，例如：&lt;？foo bar？&gt;
- 删除 XML 注释。
- 删除以 &lt;！&gt; 开头的 XML 元素。
- 删除 &lt;script&gt; 和 &lt;style&gt; 元素的内容。
- 处理这些元素中的 XML 注释（正常的注释处理不会总是有效的）。
- 替换数字字符实体引用，如 A 或  与相应的字符。
- 终止 ';' 如果输入末尾的实体引用是可选的；否则终止 ';' 是强制性的，以避免 “Alpha＆Omega Corp” 之类的错误匹配。
- 用相应的字符替换所有命名的字符实体引用。
-   被替换为空格而不是 0xa0 字符。
- 换行符代替块级元素。
- &lt;CDATA&gt; 部分被识别。
- 内嵌标签，如 &lt;b&gt;，&lt;i&gt; 或 &lt;span&gt; 将被删除。
- 大写字符实体类似 quot、gt、lt 和 amp被认为和小写处理。
<blockquote>Tip：输入不一定是一个 HTML 文档。过滤器只删除看起来像 HTML 的构造。如果输入不包含任何看起来像 HTML 的内容，则筛选器不会删除任何输入。  
</blockquote>下表介绍了 HTML 剥离的示例：  
<table class=""><colgroup><col/><col/></colgroup><thead><tr><th style="text-align: center;">输入</th><th style="text-align: center;">输出</th></tr></thead><tbody><tr><td><code>my &lt;a href="www.foo.bar"&gt;link&lt;/a&gt;</code>  
</td><td><p style="text-align: center;">my link  
  
</td></tr><tr><td><code>&lt;br&gt;hello&lt;!--comment--&gt;</code>  
</td><td><p style="text-align: center; "><span style="text-align: left;">hello</span>  
  
</td></tr><tr><td><code>hello&lt;script&gt;&lt;!-- f('&lt;!--internal--&gt;&lt;/script&gt;'); --&gt;&lt;/script&gt;</code>  
</td><td><p style="text-align: center;">hello  
  
</td></tr><tr><td><code>if a&lt;b then print a;</code>  
</td><td><p style="text-align: center;">如果 a &lt; b，则输出 a;  
</td></tr><tr><td><code>hello &lt;td height=22 nowrap align="left"&gt;</code>  
</td><td><p style="text-align: center;">hello  
  
</td></tr><tr><td><code>a&lt;b A Alpha&amp;Omega</code> Ω  
</td><td><p style="text-align: center;">a &lt;b A Alpha＆ΩΩ  
</td></tr></tbody></table>例：  
```
&lt;analyzer&gt;
  &lt;charFilter class="solr.HTMLStripCharFilterFactory"/&gt;
  &lt;tokenizer ...&gt;
  [...]
&lt;/analyzer&gt;
```
### solr.ICUNormalizer2CharFilterFactory
该过滤器使用 ICU4J 执行预标记化 Unicode 标准化。  
参数：  
**name**
一个Unicode范式，一<code>nfc</code>，<code>nfkc</code>，<code>nfkc_cf</code>。默认是<code>nfkc_cf</code>。  
**mode**
无论是<code>compose</code>或<code>decompose</code>。默认是<code>compose</code>。使用<code>decompose</code>带<code>name="nfc"</code>或<code>name="nfkc"</code>分别获得NFD或NFKD。  
**filter**
一个 UnicodeSet 模式。集外的代码点始终保持不变。默认是<code>[]</code>（空集，没有过滤 - 所有的码点都受到规范化）。  

例：  
```
&lt;analyzer&gt;
  &lt;charFilter class="solr.ICUNormalizer2CharFilterFactory"/&gt;
  &lt;tokenizer ...&gt;
  [...]
&lt;/analyzer&gt;
```
### solr.PatternReplaceCharFilterFactory
此过滤器使用正则表达式替换或更改字符模式。  
参数：  
**pattern**
正则表达式模式应用于传入的文本。  
**replacement**
用来替换匹配模式的文本。  

你可以在 schema.xml 中像这样配置这个过滤器：  
```
&lt;analyzer&gt;
  &lt;charFilter class="solr.PatternReplaceCharFilterFactory"
             pattern="([nN][oO]\.)\s*(\d+)" replacement="$1$2"/&gt;
  &lt;tokenizer ...&gt;
  [...]
&lt;/analyzer&gt;
```
下面的表格给出了基于正则表达式的模式替换的例子：  
<table class=""><colgroup><col/><col/><col/><col/><col/></colgroup><thead><tr><th style="text-align: center;">输入</th><th style="text-align: center;">模式</th><th style="text-align: center;">替代</th><th style="text-align: center;">输出</th><th style="text-align: center;">描述</th></tr></thead><tbody><tr><td><p style="text-align: center;">see-ing looking  
  
</td><td><p style="text-align: center;"><code>(\w+)(ing)</code>  
</td><td><p style="text-align: center;"><code>$1</code>  
</td><td><p style="text-align: center;">see-ing look  
  
</td><td><p style="text-align: center;">从词尾删除 “ing”  
</td></tr><tr><td><p style="text-align: center;">see-ing looking  
  
</td><td><p style="text-align: center;"><code>(\w+)ing</code>  
</td><td><p style="text-align: center;"><code>$1</code>  
</td><td><p style="text-align: center;">see-ing look  
  
</td><td><p style="text-align: center;">同上，第二个括号可以省略  
</td></tr><tr><td><p style="text-align: center;">No.1 NO. no. 543  
  
</td><td><p style="text-align: center;"><code>[nN][oO]\.\s*(\d+)</code>  
</td><td><p style="text-align: center;"><code>#$1</code>  
</td><td><p style="text-align: center;">#1 NO. #543  
  
</td><td><p style="text-align: center;">替换一些字符串文字  
</td></tr><tr><td><p style="text-align: center;">abc=1234=5678  
  
</td><td><p style="text-align: center;"><code>(\w+)=(\d+)=(\d+)</code>  
</td><td><p style="text-align: center;"><code>$3=$1=$2</code>  
</td><td><p style="text-align: center;">5678=abc=1234  
  
</td><td><p style="text-align: center;">改变组的顺序  
</td></tr></tbody></table>
