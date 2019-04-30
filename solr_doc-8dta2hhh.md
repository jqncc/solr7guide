## Solr JDBC驱动程序：Python/Jython 
<div class="content-intro view-box ">
## Python/Jython

Solr的JDBC驱动程序支持Python和Jython。  

## Python

Python 支持使用 JayDeBeApi 库访问 JDBC。必须将CLASSPATH变量配置为包含 solr-solrj jar 和支持的 solrj-lib jar。  
  

### JayDeBeApi<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-python-jython.html#jaydebeapi"/>

run.sh：  
```
#!/usr/bin/env bash
# Java 8 must already be installed
pip install JayDeBeApi
export CLASSPATH="$(echo $(ls /opt/solr/dist/solr-solrj* /opt/solr/dist/solrj-lib/*) | tr ' ' ':')"
python solr_jaydebeapi.py
```
solr_jaydebeapi.py：  
```
#!/usr/bin/env python
# https://pypi.python.org/pypi/JayDeBeApi/
import jaydebeapi
import sys
if __name__ == '__main__':
  jdbc_url = "jdbc:solr://localhost:9983?collection=test"
  driverName = "org.apache.solr.client.solrj.io.sql.DriverImpl"
  statement = "select fielda, fieldb, fieldc, fieldd_s, fielde_i from test limit 10"
  conn = jaydebeapi.connect(driverName, jdbc_url)
  curs = conn.cursor()
  curs.execute(statement)
  print(curs.fetchall())
  conn.close()
  sys.exit(0)
```

## Jython<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-python-jython.html#jython"/>

Jython支持通过Java接口或zxJDBC库本地访问JDBC。CLASSPATH变量必须配置为包含solr-solrj jar和支持的solrj-lib jar。  
run.sh：  
```
#!/usr/bin/env bash
# Java 8 and Jython must already be installed
export CLASSPATH="$(echo $(ls /opt/solr/dist/solr-solrj* /opt/solr/dist/solrj-lib/*) | tr ' ' ':')"
jython [solr_java_native.py | solr_zxjdbc.py]
```

### Java Native<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-python-jython.html#java-native"/>

solr_java_native.py：  
```
#!/usr/bin/env jython
# http://www.jython.org/jythonbook/en/1.0/DatabasesAndJython.html
# https://wiki.python.org/jython/DatabaseExamples#SQLite_using_JDBC
import sys
from java.lang import Class
from java.sql  import DriverManager, SQLException
if __name__ == '__main__':
  jdbc_url = "jdbc:solr://localhost:9983?collection=test"
  driverName = "org.apache.solr.client.solrj.io.sql.DriverImpl"
  statement = "select fielda, fieldb, fieldc, fieldd_s, fielde_i from test limit 10"
  dbConn = DriverManager.getConnection(jdbc_url)
  stmt = dbConn.createStatement()
  resultSet = stmt.executeQuery(statement)
  while resultSet.next():
    print(resultSet.getString("fielda"))
  resultSet.close()
  stmt.close()
  dbConn.close()
  sys.exit(0)
```

### zxJDBC<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-python-jython.html#zxjdbc"/>

solr_zxjdbc.py：  
```
#!/usr/bin/env jython
# http://www.jython.org/jythonbook/en/1.0/DatabasesAndJython.html
# https://wiki.python.org/jython/DatabaseExamples#SQLite_using_ziclix
import sys
from com.ziclix.python.sql import zxJDBC
if __name__ == '__main__':
  jdbc_url = "jdbc:solr://localhost:9983?collection=test"
  driverName = "org.apache.solr.client.solrj.io.sql.DriverImpl"
  statement = "select fielda, fieldb, fieldc, fieldd_s, fielde_i from test limit 10"
  with zxJDBC.connect(jdbc_url, None, None, driverName) as conn:
    with conn:
      with conn.cursor() as c:
        c.execute(statement)
        print(c.fetchall())
  sys.exit(0)
```
