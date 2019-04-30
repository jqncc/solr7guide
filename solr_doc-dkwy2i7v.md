## Solr基于规则的授权插件 
<div class="content-intro view-box ">Solr允许配置角色来控制用户对系统的访问。  
  
这是通过分配给用户的基于规则的权限定义来完成的。这些角色是完全可自定义的，并且提供了限制对特定集合、请求处理程序、请求参数和请求方法的访问的能力。  
如果您创建了角色，则可以将这些角色用于任何身份验证插件或自定义身份验证插件。您只需确保使用您的身份验证系统提供的适当用户 ID 来配置角色到用户（role-to-user）的映射。  
一旦通过API定义，角色就存储在security.json中。  

## 启用授权插件

必须在 security.json 中启用该插件。此文件以及将其放置到系统中的位置在“使用security.json启用插件”一节中进行了详细介绍。  
  
这个文件有两个部分，即authentication（身份验证）部分和authorization（授权）部分。该authentication部分存储有关用于身份验证的类的信息。  
该authorization部分与基本身份验证无关，但是是一个单独的授权插件，旨在支持细粒度（fine-grained）的用户访问控制。创建security.json时，可以将权限添加到文件中，也可以使用下面描述的授权API在需要时添加它们。  
此示例security.json显示了基本身份验证插件如何与此授权插件配合使用：  
```
{
"authentication":{
   "class":"solr.BasicAuthPlugin", 【1】
   "blockUnknown": true, 【2】
   "credentials":{"solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c="} 【3】
},
"authorization":{
   "class":"solr.RuleBasedAuthorizationPlugin", 【4】
   "permissions":[{"name":"security-edit",
      "role":"admin"}], 【5】
   "user-role":{"solr":"admin"} 【6】
}}
```
此示例中定义了以下几项内容（对于上述代码中的数字）：  
  
<p/>1 <li>"solr" 用户已被定义为 "admin" 角色。</li>
## 权限属性

每个角色由一个或多个权限组成，这些权限定义用户被允许执行的操作。每个权限都由几个定义允许的活动的属性组成。有一些预定义的权限不能被修改。  
  
查询权限是为了在security.json显示它们。匹配的第一个权限适用于每个用户，所以最严格的权限应该在列表的顶部。权限顺序可以通过授权API的参数进行控制，如下所述。  

### 预定义的权限

有几个预先定义的权限。它们具有固定的默认值，不能修改，并且无法添加新属性。要使用这些属性，只需定义一个包含此权限的角色，然后将用户分配给该角色。  
  
预定义的权限是：  

    - security-edit：允许此权限编辑安全性配置，这意味着security.json允许通过API 修改的任何更新操作。
    - security-read：允许此权限读取安全配置​​，这意味着security.json允许通过API 读取设置的任何操作。
    - schema-edit：允许此权限使用架构API编辑集合的架构。请注意，这允许所有集合的架构编辑权限。如果编辑权限只应用于特定集合，则需要创建自定义权限。
    - schema-read：允许此权限使用架构API读取集合的架构。请注意，这允许所有集合的模式读取权限。如果读取权限只应用于特定集合，则需要创建自定义权限。
    - config-edit：允许此权限使用Config API，Request Parameters API和其他修改configoverlay.json的API来编辑集合的配置。请注意，这允许所有集合的配置编辑权限。如果编辑权限只应用于特定集合，则需要创建自定义权限。
    - core-admin-read：读取核心管理API上的操作
    - core-admin-edit：可以改变系统状态的核心管理命令。
    - config-read：允许此权限使用Config API，Request Parameters API和其他修改configoverlay.json的API来读取集合的配置。请注意，这允许所有集合的配置读取权限。如果读取权限只应用于特定集合，则需要创建自定义权限。
    - collection-admin-edit：允许此权限使用Collections API编辑集合的配置。请注意，这允许所有集合的配置编辑权限。如果编辑权限只应用于特定集合，则需要创建自定义权限。具体来说，允许以下的集合API操作：CREATE，RELOAD，SPLITSHARD，CREATESHARD，DELETESHARD，CREATEALIAS，DELETEALIAS，DELETE，DELETEREPLICA，ADDREPLICA，CLUSTERPROP，MIGRATE，ADDROLE，REMOVEROLE，ADDREPLICAPROP，DELETEREPLICAPROP，BALANCESHARDUNIQUE，REBALANCELEADERS
    - collection-admin-read：允许此权限使用Collections API读取集合的配置。请注意，这允许所有集合的配置读取权限。如果读取权限只应用于特定集合，则需要创建自定义权限。具体来说，允许以下的集合API操作：LIST，OVERSEERSTATUS，CLUSTERSTATUS，REQUESTSTATUS
    - update：允许此权限对任何集合执行任何更新操作。这包括发送索引文件（使用更新请求处理程序）。默认情况下，这适用于所有集合（collection:"*"）。
    - read：允许此权限对任何集合执行任何读取操作。这包括使用搜索处理程序（使用请求处理程序）进行查询，例如：/select，/get，/browse，/tvrh，/terms，/clustering，/elevate，/export，/spell，/clustering和/sql。默认情况下，这适用于所有集合（collection:"*"）。
    - all：Solr的任何请求。

## 授权API

### 授权API端点
/admin/authorization：使用一组命令来创建权限，将权限映射到角色，并将角色映射到用户。  

### 管理权限

三个命令控制管理权限：  

    - set-permission：创建新权限，覆盖现有权限定义或为角色分配预定义的权限。
    - update-permission：更新现有权限定义的一些属性。
    - delete-permission：删除权限定义。

如果权限不在上面的预定义权限列表中，则需要创建权限。  
可以使用几个属性来定义您的自定义权限。  

**name**
    
        权限的名称。只有在预定义权限的情况下才需要。  
    
**collection**
    权限将应用到的集合。  
        
            当允许的路径是特定于集合的路径时（例如设置权限以允许使用Schema API时），省略集合属性将允许为所有集合定义的路径或方法。但是，如果路径是非集合特定的路径（如Collections API），则集合值必须为<code>null</code>。默认值是*（所有集合）。  
  
**path**
    
        请求处理程序名称，如<code>/update</code>或者 <code>/select</code>。支持通配符，以适应所有路径（如，<code>/update/*</code>）。  
    
**method**
    
        允许此权限的HTTP方法。您可以只允许GET请求，或者有一个允许PUT和POST请求的角色。该属性允许的方法值是GET，POST，PUT，DELETE和HEAD。  
    
**params**
    
        请求参数的名称和值。如果所有的请求参数都要匹配，那么这个属性可以省略，但是如果定义的话只会限制访问提供的值。  
        
            例如，此属性可用于限制角色允许使用Collections API执行的操作。如果只允许角色执行LIST或CLUSTERSTATUS请求，则可以按如下方式进行定义：  
```
{"params": {
   "action": ["LIST", "CLUSTERSTATUS"]
  }
}
```
  
        
            参数的值可以是简单的字符串，也可以是正则表达式。使用前缀<code>REGEX:</code>来使用正则表达式匹配而不是字符串标识匹配  
          
        
            如果命令LIST和CLUSTERSTATUS不区分大小写，上面的例子应该如下：  
```
{"params": {
   "action": ["REGEX:(?i)LIST", "REGEX:(?i)CLUSTERSTATUS"]
  }
}
```
  
    
**before**
    
        该属性允许排序权限。这个属性的值是这个新权限应该放在<code>security.json</code>之前的权限的索引。该索引按照它们创建的顺序自动分配。  
**role**
    
        授予此权限的角色的名称。此名称将用于将用户标识映射到角色以授予这些权限。该值可以是通配符，如（<code>*</code>），这意味着任何用户都可以，但是没有用户是不可以的。  
    

下面创建一个名为“collection-mgr”的新权限，允许创建和列出集合。权限将被放置在“读取”权限之前。还要注意，我们已经将“集合”定义为 null，这是因为对集合API的请求从来都不是特定于集合的。  
```
curl --user solr:SolrRocks -H 'Content-type:application/json' -d '{
  "set-permission": {"collection": null,
                     "path":"/admin/collections",
                     "params":{"action":["LIST", "CREATE"]},
                     "before": 3,
                     "role": "admin"}
}' http://localhost:8983/solr/admin/authorization
```
将所有集合的更新权限应用于称为dev的角色，dev并将权限读取到名为guest的角色：  
```
curl --user solr:SolrRocks -H 'Content-type:application/json' -d '{
  "set-permission": {"name": "update", "role":"dev"},
  "set-permission": {"name": "read", "role":"guest"}
}' http://localhost:8983/solr/admin/authorization
```

### 更新或删除权限

权限可以通过列表中的索引进行访问。使用/admin/authorizationAPI查看现有权限及其索引。  
以下示例更新'role'索引3处的权限属性：  
```
curl --user solr:SolrRocks -H 'Content-type:application/json' -d '{
  "update-permission": {"index": 3,
                       "role": ["admin", "dev"]}
}' http://localhost:8983/solr/admin/authorization
```
以下示例删除索引3处的权限：  
```
curl --user solr:SolrRocks -H 'Content-type:application/json' -d '{
  "delete-permission": 3
}' http://localhost:8983/solr/admin/authorization
```

### 将角色映射到用户
单个命令允许将角色映射到用户：  

    - set-user-role：将用户映射到权限。

要删除用户的权限，您应该将角色设置为null。没有命令来删除用户角色。  
提供给命令的值只是一个用户ID和一个或多个用户应具有的角色。  
例如，以下内容将“admin”和“dev”角色授予给用户“solr”，并从用户ID“harry”中删除所有角色：  
```
curl -u solr:SolrRocks -H 'Content-type:application/json' -d '{
   "set-user-role" : {"solr": ["admin","dev"],
                      "harry": null}
}' http://localhost:8983/solr/admin/authorization
```
