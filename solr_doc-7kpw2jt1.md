## Solr使用Python 
<div class="content-intro view-box ">
## 使用Python
Solr包含一个专门用于Python的输出格式，但JSON输出更健壮一些。  
  
### 简单的Python
进行查询是一件简单的事情。首先，告诉Python你将需要建立HTTP连接。  
```
from urllib2 import *
```
现在打开一个到服务器的连接并获得响应。该wt查询参数告诉Solr以Python可以理解的格式返回结果。  
```
connection = urlopen('http://localhost:8983/solr/collection_name/select?q=cheese&amp;wt=python')
response = eval(connection.read())
```
现在解释响应只是把你需要的信息抽出来。  
```
print response['response']['numFound'], "documents found."
# Print the name of each document.
for document in response['response']['docs']:
  print "  Name =", document['name']
```
### 带有JSON的Python
JSON是一种更健壮的响应格式，但是您需要添加一个Python包才能使用它。在命令行中，像这样安装simplejson软件包：  
  
```
sudo easy_install simplejson
```
一旦完成，提出查询几乎与以前一样。但是请注意，wt查询参数现在是json（如果不指定wt参数，则它也是默认参数），并且响应现在由 simplejson. load () 来消除。  
```
from urllib2 import *
import simplejson
connection = urlopen('http://localhost:8983/solr/collection_name/select?q=cheese&amp;wt=json')
response = simplejson.load(connection)
print response['response']['numFound'], "documents found."
# Print the name of each document.
for document in response['response']['docs']:
  print "  Name =", document['name']
```
