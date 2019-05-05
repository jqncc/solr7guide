## Solr用例的字段属性 
<div class="content-intro view-box ">以下是 Solr 中常见的用例的字段属性的总结，以及字段或字段类型应该支持该情况的属性。在表中输入的 true 或 false 表示该选项必须设置为给定的值才能正常工作。如果没有提供条目，则表示该属性的设置对案例没有影响。  
  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
                        <col/>
                            <col/>
                                <col/>
                                    <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">用例</th>
            <th style="text-align: center;">索引</th>
            <th style="text-align: center;">存储</th>
            <th style="text-align: center;">多值</th>
            <th style="text-align: center;">omitNorms</th>
            <th style="text-align: center;">termVectors</th>
            <th style="text-align: center;">termPositions</th>
            <th style="text-align: center;">docValues</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center; ">在字段中搜索  
            </td>
            <td>
                <p style="text-align: center; ">true  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
            <td/>
            <td/>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">检索内容  
            </td>
            <td/>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>8</sup>
                  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
            <td><p style="text-align: center; "><span style="background-color: transparent; text-align: left;">true</span><sup>8</sup>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">用作唯一键  
            </td>
            <td>
                <p style="text-align: center; ">true  
  
            </td>
            <td/>
            <td>
                <p style="text-align: center; ">false  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">排序领域  
            </td>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>7</sup>
                  
            </td>
            <td/>
            <td>
                <p style="text-align: center; ">false  
            </td>
            <td>
                <p style="text-align: center; ">true<sup>1</sup>
                  
            </td>
            <td/>
            <td/>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>7</sup>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">highlighting  
  
            </td>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>4</sup>
                  
            </td>
            <td>
                <p style="text-align: center; ">true  
  
            </td>
            <td/>
            <td/>
            <td>
                <p style="text-align: center; ">true<sup>2</sup>
                  
            </td>
            <td><p style="text-align: center; "><span style="background-color: transparent; text-align: left;">true</span><sup>3</sup>
                  
            </td>
            <td/>
        </tr>
        <tr>
            <td><p style="text-align: center;"><span style="background-color: transparent; text-align: left;">faceting</span><sup>5</sup>
                  
            </td>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>7</sup>
                  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
            <td/>
            <td><p style="text-align: center; "><span style="background-color: transparent;">true</span><sup>7</sup>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: left;">添加多个值，维护秩序  
            </td>
            <td/>
            <td/>
            <td>
                <p style="text-align: center; ">false  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
        </tr>
        <tr>
            <td>
                字段长度影响文档分数  
            </td>
            <td/>
            <td/>
            <td/>
            <td>
                <p style="text-align: center; ">false  
            </td>
            <td/>
            <td/>
            <td/>
        </tr>
        <tr>
            <td><p style="text-align: center;"><span style="background-color: transparent; text-align: left;">MoreLikeThis</span><span style="background-color: transparent; text-align: left;"> </span><sup>5</sup>
                  
            </td>
            <td/>
            <td/>
            <td/>
            <td/>
            <td><p style="text-align: center; "><span style="background-color: transparent; text-align: left;">true</span><sup>6</sup>
                  
            </td>
            <td/>
            <td/>
        </tr>
    </tbody>
</table>
笔记（对上述表格中带有角标的项进行解释）：  
1 <li>建议但不是必要的。</li>2 <li>将被使用，如果存在，但没有必要。</li>3 <li>如果 termVectors = true。</li>4 <li>必须为字段定义标记器，但不需要对其进行索引。</li>5 <li>在了解分析器、标志器和过滤器中描述。</li>6 <li>术语向量在这里不是强制性的。如果不是 true，则分析存储的字段。所以推荐使用术语向量，但只有在 stored=false 时才需要。</li>7 <li>对于大多数字段类型，indexed 或者 docValues 都必须是 true，但都不是必需的。在许多情况下，DocValues 可以更有效率。对于[Int/Long/Float/Double/Date]PointFields，docValues=true 是必需的。</li>8 <li>默认情况下将使用存储的内容，但也可以使用 docValues。请参阅 DocValues。</li>
