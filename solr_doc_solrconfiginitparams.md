# InitParams

`<initParams>` 用于定义处理程序(requestHandler)配置之外的请求处理参数。

以下是两个可用到initParams的情况：

- 一些处理程序在代码中隐式定义， 应该有一种方法来添加/追加/重写一些隐式定义的属性。
- 为多个请求处理器定义共同的请求参数。

例如，如果您希望多个搜索处理程序返回相同的字段列表，则可以创建一个`<initParams>`部分，而无需在每个请求处理程序定义中定义相同的一组参数。如果您有一个单一的请求处理程序，该处理程序应该返回不同的字段，那么您可以像往常一样在个别`<requestHandler>`部分定义覆盖参数。

一个`<initParams>`部分镜像了请求处理程序的属性和配置。它可以包含用于默认、附加和不变的部分，与任何请求处理程序相同。

例如，这里是在_default示例中默认定义的`<initParams>`部分：

```xml
<initParams path="/update/**,/query,/select,/tvrh,/elevate,/spell,/browse">
  <lst name="defaults">
    <str name="df">_text_</str>
  </lst>
</initParams>
```

这会将路径部分中指定的所有请求处理程序的默认搜索字段（“df”）设置为“ text ”。如果我们以后想要更改/query请求处理程序以默认搜索其他字段，我们可以通过在`<requestHandler>`section中定义参数来覆盖`<initParams>`中的配置。

语法和语义与`<requestHandler>`类似。以下是属性：

- path

以逗号分隔的路径列表，将使用这些参数。可以在路径中使用通配符来定义嵌套路径，如下所述。

- name

参数名称。如果路径未显式命名，则可以在requestHandler定义中直接使用该名称。如果您为`<initParams>`指定了名称，则可以引用未定义为路径的`<requestHandler>`中的Params。

例如，如果`<initParams>`的name="myParams"，则可以在定义请求处理程序时调用该名称：
`<requestHandler name="/dump1" class="DumpRequestHandler" initParams="myParams"/>`

## initParams中的通配符

`<initParams>`的path可以使用通配符。单个星号（*）表示直接子级。双星号（**）表示所有子孙级路径。

如下配置：

```xml
<initParams name="myParams" path="/myhandler,/root/*,/root1/**">
  <lst name="defaults">
    <str name="fl">_text_</str>
  </lst>
  <lst name="invariants">
    <str name="rows">10</str>
  </lst>
  <lst name="appends">
    <str name="df">title</str>
  </lst>
</initParams>
```

我们已经使用此部分定义了三个路径：

/myhandler 直接路径。  
/root/* 表示/root下的直接子级路径，如：/root/a,/root/b。  
/root1/** 表示/root1下子孙级路径，如：/root1/a,/root1/a/b,/root1/c/d/e。  

当我们定义requestHandler时，通配符将以下列方式工作：

`<requestHandler name="/myhandler" class="SearchHandler"/>`  
/myhandler在定义的`<initParams>`的path中，所以这将应用这些参数

再看下一个requestHandler /root/search5：

`<requestHandler name="/root/search5" class="SearchHandler"/>`

它符合通配符路径/root/*，所以这个请求处理程序将使用这些参数。但是，下面这个则不会，因为/root/search5/test不止一个级别/root：

`<requestHandler name="/root/search5/test" class="SearchHandler"/>`

如果我们想要定义所有级别的嵌套路径，我们应该使用双星号，如示例路径中所示/root1/**：

`<requestHandler name="/root1/search/tests" class="SearchHandler"/>`

/root1无论是否在请求处理程序中明确定义，任何路径都将使用匹配initParams部分中定义的参数。
