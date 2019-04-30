## Solr JDBC - R 
<div class="content-intro view-box ">
# Solr JDBC - R
R支持使用[RJDBC](https://www.rforge.net/RJDBC/)库访问JDBC 。  
RJDBC是一个在JDBC基础上实现DBI的包。这允许在R中通过JDBC接口使用任何DBMS。唯一的要求是使用Java和JDBC驱动程序来访问数据库引擎。  
  
## RJDBC<a href="http://lucene.apache.org/solr/guide/7_0/solr-jdbc-r.html#rjdbc"/>
run.sh：  
```
#!/usr/bin/env bash
# Java 8 must already be installed and R configured with `R CMD javareconf`
Rscript -e 'install.packages("RJDBC", dep=TRUE)'
Rscript solr_rjdbc.R
```
solr_rjdbc.R：  
```
# https://www.rforge.net/RJDBC/
library("RJDBC")
solrCP &lt;- c(list.files('/opt/solr/dist/solrj-lib', full.names=TRUE), list.files('/opt/solr/dist', pattern='solrj', full.names=TRUE, recursive = TRUE))
drv &lt;- JDBC("org.apache.solr.client.solrj.io.sql.DriverImpl",
           solrCP,
           identifier.quote="`")
conn &lt;- dbConnect(drv, "jdbc:solr://localhost:9983?collection=test", "user", "pwd")
dbGetQuery(conn, "select fielda, fieldb, fieldc, fieldd_s, fielde_i from test limit 10")
dbDisconnect(conn)
```
