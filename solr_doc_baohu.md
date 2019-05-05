## 保护Solr 
<div class="content-intro view-box ">在规划如何保护Solr时，您应该考虑哪些可用的功能或方法适合您：  
  

    - 用户的身份验证或授权使用：<ul><li>Kerberos身份验证插件
- 基本身份验证插件
- 基于规则的授权插件
- 自定义身份验证或授权插件
</li>
    <li>启用S​​SL</li>
    <li>如果使用SolrCloud 和 ZooKeeper访问控制</li>
</ul>
注意：没有任何 Solr API（包括管理 UI）被设计为向信任方公开。调整您的防火墙，以便只有受信任的计算机和人员被允许访问。因此，该项目不会将Admin UI XSS问题视为安全漏洞。但是，我们仍然要求您在JIRA中报告这些问题。  

      
  

      
  
