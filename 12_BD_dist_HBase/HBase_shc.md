# Usar o Apache Spark para ler e gravar dados do Apache HBase

Usando:
```
 https://docs.microsoft.com/pt-br/azure/hdinsight/hdinsight-using-spark-query-hbase 
```

## criando os dados 


(hadoop)$ hbase shell:
```
create 'Contacts', 'Personal', 'Office'

put 'Contacts', '1000', 'Personal:Name', 'John Dole'
put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'

```

## instalação do shc

```
$ pwd
/home/hadoop

$ git clone https://github.com/hortonworks-spark/shc
Cloning into 'shc'...
remote: Enumerating objects: 3835, done.
remote: Total 3835 (delta 0), reused 0 (delta 0), pack-reused 3835
Receiving objects: 100% (3835/3835), 587.17 KiB | 1.53 MiB/s, done.
Resolving deltas: 100% (1204/1204), done.

$ git checkout branch-2.4
Branch 'branch-2.4' set up to track remote branch 'branch-2.4' from 'origin'.
Switched to a new branch 'branch-2.4'

c$ mvn clean package -DskipTests 
[INFO] Scanning for projects...
Downloading from central: https://repo1.maven.org/maven2/org/apache/apache/14/apache-14.pom
Downloaded from central: https://repo1.maven.org/maven2/org/apache/apache/14/apache-14.pom (15 kB at 8.7 kB/s)
          

            .... depois de uma eternidade...


[INFO] --- maven-site-plugin:3.3:attach-descriptor (attach-descriptor) @ shc-examples ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for HBase Spark Connector Project Parent POM 1.1.3-2.4-s_2.11:
[INFO] 
[INFO] HBase Spark Connector Project Parent POM ........... SUCCESS [02:02 min]
[INFO] HBase Spark Connector Project Core ................. SUCCESS [05:45 min]
[INFO] HBase Spark Connector Project Examples ............. SUCCESS [ 12.836 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  08:03 min
[INFO] Finished at: 2021-11-19T20:40:37-03:00
[INFO] ------------------------------------------------------------------------

$ spark-shell --jars /home/hadoop/shc/core/target/shc-core-1.1.3-2.4-s_2.11.jar, | /home/hadoop/hbase-2.4.8/
```

## Importando o uso

- No spark shell:
```
## importando libs
import org.apache.spark.sql.{SQLContext, _}
import org.apache.spark.sql.execution.datasources.hbase._
import org.apache.spark.{SparkConf, SparkContext}
import spark.sqlContext.implicits._

#Criar o catálogo da tabela:
def catalog = s"""{
    |"table":{"namespace":"default", "name":"Contacts"},
    |"rowkey":"key",
    |"columns":{
    |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
    |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
    |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
    |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
    |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
    |}
|}""".stripMargin

# Definir o método pra acessar:
def withCatalog(cat: String): DataFrame = {
    spark.sqlContext
    .read
    .options(Map(HBaseTableCatalog.tableCatalog->cat))
    .format("org.apache.spark.sql.execution.datasources.hbase")
    .load()
 }
 
 #criando o dataframe:
 val df = withCatalog(catalog)

```

## visualizando os dados

```
> df.show()
+------+--------------------+--------------+------------+--------------+        
|rowkey|       officeAddress|   officePhone|personalName| personalPhone|
+------+--------------------+--------------+------------+--------------+
|  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
|  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
+------+--------------------+--------------+------------+--------------+
```

##  Criando uma view

```
df.createTempView("contacts")

## validando

> spark.sqlContext.sql("select personalName, officeAddress from contacts").show
+------------+--------------------+
|personalName|       officeAddress|
+------------+--------------------+
|   John Dole|1111 San Gabriel Dr.|
| Calvin Raji|5415 San Gabriel Dr.|
+------------+--------------------+

```

