## MBean请求处理程序 
<div class="content-intro view-box ">MBean请求处理程序提供对管理 UI 的插件/统计（Plugin/Stats）页上提供的信息的编程访问。
      
  
MBean请求处理程序接受以下参数：  
- key  

 
        通过对象键来限制结果。  
    
- cat  

   
        按类别名称限制结果。  
    
- stats  

    
        指定是否使用结果返回统计信息。您可以在每个字段的基础上覆盖<code>stats</code>参数。默认是<code>false</code>。  
    
- wt  

   
        输出格式。这与查询中的<code>wt</code>参数操作相同。默认是<code>json</code>。  
    


## MBeanRequestHandler示例

以下示例假设您正在运行Solr的techproducts示例配置：  
```
bin/solr start -e techproducts
```

仅返回有关CACHE类别的信息：  
```
http://localhost:8983/solr/techproducts/admin/mbeans?cat=CACHE
```

要仅返回有关CACHE类别的信息和统计信息，请使用XML格式：  
```
http://localhost:8983/solr/techproducts/admin/mbeans?stats=true&amp;cat=CACHE&amp;wt=xml
```

返回所有内容的信息, 除 fieldCache 之外的所有内容的统计：  
```
http://localhost:8983/solr/techproducts/admin/mbeans?stats=true&amp;f.fieldCache.stats=false
```

仅返回 fieldCache 的信息和统计：  
```
http://localhost:8983/solr/techproducts/admin/mbeans?key=fieldCache&amp;stats=true
```
