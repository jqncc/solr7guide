## Solr配置集 
<div class="content-intro view-box ">在多核Solr实例中，您可能会发现要共享多个不同内核之间的配置。您可以使用命名的configsets来实现这个功能，这些配置文件本质上是存储在可配置的configset基本目录下的共享配置目录。  
  
要创建一个configset，只需在configset的基本目录下添加一个新的目录即可。这个configset将会被这个目录的名字所标识。然后将您想要共享的config目录复制到此。结构应该是像下面这样的：  
```
/&lt;configSetBaseDir&gt;
    /configset1
        /conf
            /managed-schema
            /solrconfig.xml
    /configset2
        /conf
            /managed-schema
            /solrconfig.xml
```
默认的基本目录是：$SOLR_HOME/configsets，可以在solr.xml中配置。  
使用一个configset创建一个新的核心，将configSet作为核心属性之一。例如，如果您通过核心管理API执行此操作：  
http://localhost:8983/admin/cores?action=CREATE&amp;name=mycore&amp;instanceDir=path/to/instance&amp;configSet=configset2  
