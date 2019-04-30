## 使用AsciiDoc文件 
<div class="content-intro view-box "><h2>AsciiDoc语法表  
</h2>AsciiDoc语法的权威手册在“Asciidoctor用户手册”中。但是，为了帮助人们开始，这里提供一个简单的语法表。  
  
### AsciiDoc与Asciidoctor语法
我们使用Asciidoctor项目中的工具来构建Ref Guide的HTML和PDF版本。Asciidoctor是原来的AsciiDoc项目的Ruby端口，几年前这个项目大部分都被抛弃了。  
  
虽然这两者之间的大部分语法都是相同的，但AsciiDoc支持AsciiDoc中不存在的许多约定。虽然Asciidoctor项目试图提供与旧项目的后向兼容性，但这可能永远不会是真的。出于这个原因，强烈建议只使用Asciidoctor用户手册作为任何在这里没有描述的语法的参考。  
### 基本的AsciiDoc语法

##### 加粗（Bold）
将星号放在文字上以使其粗体。  
更多信息：http : //asciidoctor.org/docs/user-manual/#bold-and-italic  
##### 斜体（Italics）
在字符串的任一侧使用下划线将文本变为斜体。  
更多信息：http : //asciidoctor.org/docs/user-manual/#bold-and-italic  
##### 标题（Headings）
等号（=）用于表示标题级别。每个等号都是一个级别。每个页面只能有一个顶层。  
级别应适当嵌套。在构建过程中，验证发生的目的是确保级别3之前是级别2，级别4之前是级别3等。包括不按顺序的标题级别（例如级别3，级别5）不会失败建立，但会产生一个错误。  
更多信息：http://asciidoctor.org/docs/user-manual/#sections  
##### 代码示例（Code Examples）
使用反引号`来表示应该是等宽的文本，例如段落主体中的代码或类名称。  
更多信息：http://asciidoctor.org/docs/user-manual/#mono  
更长的代码示例可以用source块与文本分开。这些允许定义正在使用的语法，以便代码正确地突出显示。  
示例源块：  
```
[source,xml]
&lt;field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /&gt;
```
如果您的代码块将包含换行符，请在整个块之前和之后放置4个连字符（----）。  
更多信息：http://asciidoctor.org/docs/user-manual/#source-code-blocks  
###### 源块语法高亮（Source Block Syntax Highlighting）
PDF和HTML输出使用Pygments为代码示例添加语法高亮显示。这是通过在source之后添加代码块的语言来完成的，如上面的示例源代码块所示（在这种情况下为 xml）。  
  
Pygments有很多可用的词法分析器。你可以在http://pygments.org/docs/lexers看到完整的列表。使用其中一个有效的短名称来获得该语言的语法高亮显示。  
理想情况下，我们将有一个适当的词法分析器用于所有的源代码块，但这是不可能的。如有疑问，请选择text，或将其保留为空。  
### 块标题（Block Titles）
标题可以通过用句号（.）初始化标题来添加到大多数块（图像，源块，表格等）。例如，要为上面的源代码块添加一个标题：  
  
  
```
.Example ID field
[source,xml]
&lt;field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /&gt;
```
### 链接（Links）

#### 链接到互联网上的网站
将内容转换为HTML或PDF时，Asciidoctor将自动呈现许多链接类型（例如http:和mailto:），而无需任何其他语法。  
但是，您可以通过添加URI后跟方括号来为链接添加名称：  
```
http://lucene.apache.org/solr[Solr Website]
```
#### 链接到其他页面/部分
预先警告，链接到其他页面可能会有点难。根据您要创建的链接类型以及您要链接的位置，规则略有不同。  
  
构建过程包括对内部或页面间链接的验证，所以如果您可以在本地构建文档，则可以使用它来验证是否正确构建了链接（或者在提交后注意Jenkins构建）。  
通过以下所有示例，您可以添加文本以显示链接标题，方法是在节参考后面跟随显示文本添加逗号，如下所示：  
```
&lt;&lt;schema-api.adoc#modify-the-schema,Modify the Schema&gt;&gt;
```
#### 链接到同一页上的部分
如果要链接到同一页面上的定位点（或节标题），则可以简单地在要链接的定位点/标题/节标题周围使用双角括号（&lt;&lt; &gt;&gt;）。任何部分标题（以等号开头的标题）在转换过程中自动成为锚点，可用于深层链接。  
  
**示例**
如果我在页面上显示如下（从<code>defining-fields.adoc</code>）的部分：  
```
== Field Properties
Field definitions can have the following properties:
```
要从同一<code>defining-fields.adoc</code>页面的另一部分链接到本节，我只需要将部分标题放在双尖括号中，如下所示：  
```
See also the &lt;&lt;Field Properties&gt;&gt; section.
```
  
节标题将被用作显示文本; 自定义在章节标题后面添加逗号，然后显示用于显示的文本。  
  

更多信息：http : //asciidoctor.org/docs/user-manual/#internal-cross-references  
#### 链接到具有锚点ID的部分
链接到任何部分（在同一页面或另一个页面）时，还必须注意可能正在使用的任何预定义的节点（这些节点将位于双括号内[[ ]]）。当页面被转换时，这些将是你的链接需要指向的引用。  
  
**示例**
以<code>configsets-api.adoc</code>这个例子为例：  
```
[[configsets-create]]
== Create a ConfigSet
```
要链接到本节，有两种方法取决于您从哪里连接：  
  
- 从同一页面，只需使用节点名称：<code>&lt;&lt;configsets-create&gt;&gt;</code>。  
- 在另一个页面上，使用页面名称和节点名称：<code>&lt;&lt;configsets-api.adoc#configsets-create&gt;&gt;</code>。  
  

#### 链接到另一个页面
要链接到另一个页面或另一个页面上的一个部分，您必须引用完整的文件名，并引用您要链接到的部分。  
  
不幸的是，当你想将阅读器引用到另一个页面而没有深度链接到一个部分时，你不能简单地将其他文件名称放在尖括号中，并称之为一天。这是由于PDF转换 - 一旦所有的页面被合并成一个大页面的一个大的PDF页面，缺乏具体的引用，导致页面间链接失败。  
所以，你必须总是链接到一个特定的部分。如果您只想引用另一个页面的顶部，则可以使用page-shortname每个页面顶部的属性作为节点引用。  
**示例**
该文件<code>upgrading-solr.adoc</code>在顶部有一个一个<code>page-shortname</code>看起来如下：  
```
= Upgrading Solr
:page-shortname: upgrading-solr
:page-permalink: upgrading-solr.html
```
要构建一个链接到这个页面，我们需要引用文件名（<code>upgrading-solr.adoc</code>），然后使用<code>page-shortname</code>作为我们的节点引用。如：  
```
For more information about upgrades, see &lt;&lt;upgrading-solr.adoc#upgrading-solr&gt;&gt;
```
  

#### 链接到另一页上的部分
链接到某个节与链接到页面顶部的概念相同，只需要格外小心地在链接引用中正确设置的节点ID。  
  
当您链接到另一页上的部分时，您必须将标题转换为在转换过程中创建部分ID的格式。这些是改变部分的规则：  
- 所有的字符都是小写的：Using security.json with Solr 变 using security.json with solr
- 所有非alpha字符都被删除，除了连字符（所有句点，逗号，＆符号，括号等被删除）：using security.json with solr 变 using security json with solr
- 所有的空格都用连字符替换：using security json with solr 变 using-security-json-with-solr
**示例**
该文件<code>schema-api.adoc</code>有一个“修改架构”部分，如下所示：  
```
== Modify the Schema
`POST /_collection_/schema`
```
要从其他页面链接到此部分，您需要创建一个如下所示的链接：  
  
- 带有section（<code>schema-api.adoc</code>）的页面的文件名，  
- 那么哈希符号（<code>#</code>），  
- 那么转换后的部分标题（<code>modify-the-schema</code>），  
- 然后显示一个逗号和任何链接标题。  
  
  
  
上下文中的链接将如下所示：  
```
For more information, see the section &lt;&lt;schema-api.adoc#modify-the-schema,Modify the Schema&gt;&gt;
```
  

更多信息：http : //asciidoctor.org/docs/user-manual/#inter-document-cross-references  
### 有序和无序列表
AsciiDoc支持三种类型的列表：  
- 无序列表
- 有序列表
- 标记的列表
每种类型的列表可以与其他类型混合使用。所以，如果有必要的话，你可以在一个标签列表中有一个有序列表。  
#### 无序列表
简单项目符号列表需要每行以星号（*）开头。它应该是该行的第一个字符，后面跟着一个空格。  
这些列表也需要从中分离出来  
更多信息：http : //asciidoctor.org/docs/user-manual/#unordered-lists  
#### 有序列表
编号列表需要每一行以句点（.）开始。它应该是该行的第一个字符，后面跟着一个空格。  
这种风格比手动编号列表更受欢迎。  
更多信息：http : //asciidoctor.org/docs/user-manual/#ordered-lists  
#### 标记的列表
这些问题和答案列表或词汇表定义。每行应该以列表项开始，然后是双冒号（::），然后是空格或换行。  
  
标记的列表可以通过添加额外的冒号（例如:::等）来嵌套。  
如果您的内容将跨越多个段落或包含源代码块等，您将需要添加一个加号（+）来保持阅读器的各个部分。  
我们更喜欢这些参数的列表样式，因为它允许更多的自由度来显示每个参数的详细信息。例如，它自动支持内部的有序或无序列表，并且您可以包含多个段落和源块，而不必尝试将它们塞入一个较小的表单元格中。  
更多信息：http : //asciidoctor.org/docs/user-manual/#labeled-list  
### 图像（Images）
有两种方法可以包含图像：内联或块。  
  
内联图像是文字在图像周围流动的位置。块图像是自己的行上出现的，从页面上的任何其他文本出发。  
两种方法都使用图像文件名之前的image标签，但在image定义后冒号的数量是内联还是块。内嵌图像使用一个冒号（image:），而块图像使用两个冒号（image::）。  
块图像自动包括一个标题标签和一个数字（如Figure 1）。如果一个块图像包含一个标题，它将作为标题的文本包含在内。  
可选属性允许您设置替代文本，图像的大小，如果它应该是一个链接，浮动和对齐。  
更多信息：http : //asciidoctor.org/docs/user-manual/#images  
### 表（Tables）
表格可能很复杂，但要制作一个适合大多数需求的基本表格非常简单。  
#### 基本表
表格的基本结构与Markdown类似，管道（|）分隔行之间的列：  
```
|===
| col 1 row 1 | col 2 row 1|
| col 1 row 2 | col 2 row 2|
|===
```
注意|===在开始和结束时的使用。对于不完全需要的基本表格，但它确实有助于划分表格的开始和结束，以防意外地在行之间引入（或者更喜欢）空格。  
#### 标题行
要为表添加标题，只需要header在表的开始处设置属性：  
```
[options="header"]
|===
| header col 1 | header col 2|
| col 1 row 1 | col 2 row 1|
| col 1 row 2 | col 2 row 2|
|===
```
#### 定义列样式
如果您需要为列中的所有行定义特定样式，可以使用属性来完成。  
这个例子将把所有行的所有内容放在中间：  
```
[cols="2*^" options="header"]
|===
| header col 1 | header col 2|
| col 1 row 1 | col 2 row 1|
| col 1 row 2 | col 2 row 2|
|===
```
对齐或任何其他样式只能应用于特定的列。例如，这只会将表格的最后一列居中：  
```
[cols="2*,^" options="header"]
|===
| header col 1 | header col 2|
| col 1 row 1 | col 2 row 1|
| col 1 row 2 | col 2 row 2|
|===
```
更多格式化的例子：  
- 列：http : //asciidoctor.org/docs/user-manual/#cols-format
- 单元格：http : //asciidoctor.org/docs/user-manual/#cell
#### 更多的选择
表格还可以被赋予页脚行、边框和标题。您可以确定列的宽度或整个表的宽度。  
  
也可以使用CSV或DSV来代替在管道中格式化数据。  
更多信息：http : //asciidoctor.org/docs/user-manual/#tables  
### 警告（注意，警告）
AsciiDoc支持几种类型的标注框，称为“警告”：  
- NOTE  
- TIP  
- IMPORTANT  
- CAUTION  
- WARNING  
用一个冒号（如NOTE:）开始一个段落就足够了。当它被转换成HTML或PDF时，这些部分将被正确格式化 - 从主文本缩进并显示一个图标内联。  
  
您可以通过将警告标题添加到警告区块来为警告添加标题。警告块的结构是这样的：  
```
.Title of Note
[NOTE]
====
Text of note
====
```
在这个例子中，警告的类型被包含在方括号（[NOTE]）中，并且标题以句点为前缀。四个等号表示注释文本的起点和终点（可以包括新行，列表，代码示例等）。  
更多信息：http : //asciidoctor.org/docs/user-manual/#admonition  
## 使用AsciiDoc文件的工具

### AsciiDoc与Asciidoctor
Solr 参考指南以 AsciiDoc 格式编写的。这种格式通常被认为是Markdown的扩展，因为它支持目录，更好的表格支持和其他特性，使得它更适合编写技术文档。  
  
我们正在使用AsciiDoc语法的一个版本以及来自一个名为Asciidoctor的开源项目的工具。这提供了对AsciiDoc语法的全面支持，但是用Ruby编写的代码取代了原来的Python处理器。有一个Java实现，称为AsciidoctorJ。原始AsciiDoc项目的进一步扩展包括对基于字体的图标和UI元素的支持。  
### Doc预览
这允许您在接近于HTML输出时看到您的文档。  
  
以下信息来自http://asciidoctor.org/docs/editing-asciidoc-with-live-preview。  
- Atom有AsciiDoc预览，它为您提供了一个在您键入时更新的面板。还有一些其他的插件来支持AsciiDoc格式和自动完成。
- Chrome，Firefox和Opera有一个实时预览浏览器插件，允许你在浏览器中打开你的AsciiDoc页面。它也将随着你的输入而更新。
- 还有一个Intellij IDEA插件来支持AsciiDoc格式。
