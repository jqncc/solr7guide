## Solr包含的字段类型 
<div class="content-intro view-box ">本节列出了 Solr 中可用的字段类型。org.apache.solr.schema 软件包包括以下列出的所有类。  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">类</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">BinaryField  
            </td>
            <td>
                二进制数据。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">BoolField  
            </td>
            <td>
                包含 true 或 false。第一个字符中的值：<code>1</code>，<code>t</code>或<code>T</code>被解释为<code>true</code>；第一个字符中的任何其他值都被解释为<code>false</code>。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">Collat​​ionField  
            </td>
            <td>
                支持排序和范围查询的 Unicode 排序规则。ICUCollat​​ionField 是一个更好的选择，如果你可以使用 ICU4J。有关更多信息，请参阅 Unicode 归类部分。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">CurrencyField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 CurrencyFieldType。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">CurrencyFieldType  
            </td>
            <td>
                支持货币和汇率。有关更多信息，请参阅使用货币和汇率部分。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">DateRangeField  
</td><td>支持索引日期范围，还包括时间点实例（单毫秒（<span style="background-color: transparent;">single-millisecond</span><span style="background-color: transparent;"> </span><span style="background-color: transparent;">）持续时间）。有关使用此字段类型的更多详细信息，请参阅使用日期部分。请考虑使用这种字段类型，即使它只是用于日期实例，特别是当查询通常在 UTC 年/月/日/小时等边界时。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center; ">DatePointField  
</td><td>日期字段。代表精确到毫秒的时间点，使用基于“维度点”的数据结构进行编码，可以非常有效地搜索特定值或值的范围。有关支持的语法的更多详细信息，请参阅使用日期部分。对于单值字段，<span style="background-color: transparent;">必须使用 docValues = "true" 来启用排序。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">DoublePointField  
            </td>
            <td>
                <span style="background-color: transparent;">双字段（64 位 IEEE 浮点）。该类使用基于 “Dimensional Points” 的数据结构对double 值进行编码，从而可以非常有效地搜索特定的值或值的范围。对于单值字段，</span><span style="background-color: transparent;">必须使用 docValues = "true" 来启用排序。</span>  
  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">ExternalFileField  
            </td>
            <td>
                从磁盘上的文件中提取值。有关更多信息，请参阅使用外部文件和进程一节。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">EnumField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 EnumFieldType。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">EnumFieldType  
</td><td>允许定义枚举的一组值，这些值可能不易按字母或数字顺序（例如，严重性等级列表）排序。这个字段类型需要一个配置文件，它列出了字段值的正确顺序。有关更多信息，请参阅使用枚举字段一节。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">FloatPointField  
            </td>
            <td>
                浮点字段（32 位 IEEE 浮点）。该类使用基于“维度点”的数据结构对浮点值进行编码，可以非常有效地搜索特定的值或值的范围。对于单值字段，<span style="background-color: transparent;">必须使用 docValues = "true" 来启用排序。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">ICUCollat​​ionField  
            </td>
            <td>
                支持排序和范围查询的 Unicode 排序规则。有关更多信息，请参阅 Unicode 归类部分。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">IntPointField  
            </td>
            <td>
                整数字段（32位有符号整数）。该类使用基于“Dimensional Points”的数据结构对int 值进行编码，可以非常有效地搜索特定值或值的范围。对于单值字段，<span style="background-color: transparent;">必须使用 docValues = "true" 来启用排序。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">LatLonPointSpatialField  
            </td>
            <td>
                <span style="background-color: transparent;">纬度/经度坐标对；可能多值多点。通常用逗号指定为 “lat，lon” 顺序。有关更多信息，请参阅空间搜索部分。</span>  
  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">LatLonType  
            </td>
            <td>
                <strong>已弃用</strong>。请考虑使用 LatLonPointSpatialField 来代替。一个单值的纬度/经度坐标对。通常用逗号指定为 “lat，lon” 顺序。有关更多信息，请参阅空间搜索部分。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">LongPointField  
            </td>
            <td>
                长字段（64 位有符号整数）。该类使用基于 “Dimensional Points” 的数据结构对foo 值进行编码，从而可以非常有效地搜索特定值或值的范围。对于单值字段，<span style="background-color: transparent;">必须使用 docValues = "true" 来启用排序。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">PointType  
            </td>
            <td>
                一个单值的 n 维点。它既用于排序不是经纬度的空间数据，也用于一些更罕见的用例。（注：<span style="background-color: transparent;">这与基于 "Point" 的数值字段无关</span><span style="background-color: transparent;">）。请参阅</span><span style="background-color: transparent;">空间搜索以获取更多信息。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">PreAnalyzedField  
            </td>
            <td>
                提供一种发送到 Solr 序列化标记流的方法，可选地具有独立存储的字段值，并且在没有任何额外的文本处理的情况下存储和索引这些信息。  
                PreAnalyzedField 的配置和用法在“使用外部文件和进程”一节中有介绍。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">RandomSortField  
            </td>
            <td>
                不包含值。对此字段类型进行排序的查询将以随机顺序返回结果。使用动态字段来使用此功能。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">SpatialRecursivePrefixTreeFieldType  
            </td>
            <td>
                （简称 RPT）接受纬度逗号经度字符串或 WKT 格式的其他形状。请参阅空间搜索以获取更多信息。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">StrField  
            </td>
            <td>
                字符串（UTF-8 编码的字符串或 Unicode）。字符串用于小型字段，不以任何方式标记或分析。他们有一个不到 32K 的硬限制。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TextField  
            </td>
            <td>
                文本，通常是多个单词或标记。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieDateField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 DatePointField。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieDoubleField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 DoublePointField。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieFloatField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 FloatPointField。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieIntField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 IntPointField。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieLongField  
            </td>
            <td>
                <strong>已弃用</strong>。改用 LongPointField。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">TrieField  
            </td>
            <td>
                <strong>已弃用</strong>。这个字段用一个 type 参数来定义要使用的 Trie * 字段的特定类；改为使用适当的“<span style="background-color: transparent;">Point Field</span><span style="background-color: transparent;">”类型。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">UUIDField  
            </td>
            <td>
                通用唯一标识符（UUID）。<span style="background-color: transparent;">通过 NEW 值， Solr 将创建一个新的 UUID。</span>  
                <strong>注意</strong>：NEW 在使用 SolrCloud 时，配置一个默认值为 UUIDField 的实例对于大多数用户是不可取的（因为结果将是每个文档的每个副本将得到一个唯一的 UUID值。建议使用 UUIDUpdateProcessorFactory 在添加文档时生成 UUID 值。  
</td></tr></tbody></table>
