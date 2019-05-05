## 在Solr中使用货币和汇率 
<div class="content-intro view-box ">该 currency 的 FieldType 为 Solr / Lucene（查询时间货币兑换和汇率） 提供货币值支持，它支持以下功能：  
  

    - Point 查询
    - Range 查询
    - 函数范围（Function range）查询
    - 排序
    - 按货币代码或符号进行货币分析
    - 对称和不对称的汇率（如果汇率与交换货币有关，则不对称汇率是有用的）

## 配置货币<a href="http://lucene.apache.org/solr/guide/7_0/working-with-currencies-and-exchange-rates.html#configuring-currencies"/>
<blockquote>注意：CurrencyField 已被弃用，以支持 CurrencyFieldType；下面的所有配置示例使用 CurrencyFieldType。  
</blockquote>货币字段类型在 schema.xml 中定义。以下是这种类型的默认配置。  
```
&lt;fieldType name="currency" class="solr.CurrencyFieldType"
           amountLongSuffix="_l_ns" codeStrSuffix="_s_ns"
           defaultCurrency="USD" currencyConfig="currency.xml" /&gt;
```
  
在这个例子中，我们定义了字段类型的名称和类别，并将 defaultCurrency 定义为“USD”，即美元。我们还定义了 currencyConfig 来使用一个名为 “currency.xml” 的文件。这是我们的默认货币与其他货币之间的汇率文件。有一个备用的实施，可以允许定期下载货币数据。请参阅下面的汇率了解更多。  
  
Solr 附带的许多示例架构都包含一个使用此类型的动态字段，如下例所示：  
```
&lt;dynamicField name="*_c"   type="currency" indexed="true"  stored="true"/&gt;
```
这个动态字段将匹配任何以 _c 结束的字段，并使其成为货币类型的字段。  
  
在索引时，货币字段可以索引在本国货币。例如，如果电子商务网站上的产品以欧元列出，则将价格字段标记为“1000，EUR”将适当地对其进行索引。价格应该用逗号与货币分开，价格必须用浮点值（小数点）进行编码。  
在查询处理期间，范围和 Point 查询均受支持。  

### 子字段后缀<a href="http://lucene.apache.org/solr/guide/7_0/working-with-currencies-and-exchange-rates.html#sub-field-suffixes"/>

您必须指定指定 amountLongSuffix 和 codeStrSuffix 的参数，对应于用于原始金额和货币动态子字段的动态字段，例如：  
```
&lt;fieldType name="currency" class="solr.CurrencyFieldType"
           amountLongSuffix="_l_ns" codeStrSuffix="_s_ns"
           defaultCurrency="USD" currencyConfig="currency.xml" /&gt;
```
在上面的例子中，原始金额字段将使用 "*_l_ns" 动态字段，该字段必须存在于架构中，并使用长字段类型，即扩展 LongValueFieldType。货币代码字段将使用 "*_s_ns" 动态字段，该字段必须存在于架构中，并使用字符串字段类型，即一个是或正在扩展的 StrField。  
  
Tip：如果存储动态子字段，则原子更新将不起作用：正如在更新文档部分时提到的，在您使用原子更新时，存储的动态子将导致索引失败。要避免此问题，请在这些动态字段上指定 stored="false"。  

## 汇率<a href="http://lucene.apache.org/solr/guide/7_0/working-with-currencies-and-exchange-rates.html#exchange-rates"/>

您可以通过指定提供者来配置汇率。本地支持两种提供程序类型：FileExchangeRateProvider 或 OpenExchangeRatesOrgProvider。  
  

### FileExchangeRateProvider<a href="http://lucene.apache.org/solr/guide/7_0/working-with-currencies-and-exchange-rates.html#fileexchangerateprovider"/>

该提供程序要求您提供汇率文件。这是默认的，这意味着要使用这个提供程序，只需将指定此类型的定义中的 currencyConfig 的文件路径和名称作为值。  
currency.xmlSolr 中包含一个示例文件，在与 schema.xml 文件相同的目录中找到。下面是这个文件的一个小片段：  
```
&lt;currencyConfig version="1.0"&gt;
  &lt;rates&gt;
    &lt;!-- Updated from http://www.exchangerate.com/ at 2011-09-27 --&gt;
    &lt;rate from="USD" to="ARS" rate="4.333871" comment="ARGENTINA Peso" /&gt;
    &lt;rate from="USD" to="AUD" rate="1.025768" comment="AUSTRALIA Dollar" /&gt;
    &lt;rate from="USD" to="EUR" rate="0.743676" comment="European Euro" /&gt;
    &lt;rate from="USD" to="CAD" rate="1.030815" comment="CANADA Dollar" /&gt;
    &lt;!-- Cross-rates for some common currencies --&gt;
    &lt;rate from="EUR" to="GBP" rate="0.869914" /&gt;
    &lt;rate from="EUR" to="NOK" rate="7.800095" /&gt;
    &lt;rate from="GBP" to="NOK" rate="8.966508" /&gt;
    &lt;!-- Asymmetrical rates --&gt;
    &lt;rate from="EUR" to="USD" rate="0.5" /&gt;
  &lt;/rates&gt;
&lt;/currencyConfig&gt;
```

### OpenExchangeRatesOrgProvider<a href="http://lucene.apache.org/solr/guide/7_0/working-with-currencies-and-exchange-rates.html#openexchangeratesorgprovider"/>

您可以将 Solr 配置为从 OpenExchangeRates.Org 下载汇率，更新率为每小时 170 美元。这些比率只是对称的。  
在这种情况下，需要在字段类型的定义中指定 providerClass，并注册一个 API 密钥。下面是一个示例:  
```
&lt;fieldType name="currency" class="solr.CurrencyFieldType"
           amountLongSuffix="_l_ns" codeStrSuffix="_s_ns"
           providerClass="solr.OpenExchangeRatesOrgProvider"
           refreshInterval="60"
           ratesFileLocation="http://www.openexchangerates.org/api/latest.json?app_id=yourPersonalAppIdKey"/&gt;
```
refreshInterval 是分钟，所以上面的例子将每 60 分钟下载一次最新的费率。刷新间隔可能增加，但不会减少。  