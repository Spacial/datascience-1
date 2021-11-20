# backup

- [Backup and Restore](https://hbase.apache.org/book.html#backuprestore)
- [Configurar backup e replicação para Apache HBase e Apache Phoenix no HDInsight](https://docs.microsoft.com/pt-br/azure/hdinsight/hbase/apache-hbase-backup-replication)

## ajustando a config


```
$ cat hbase-2.4.8/conf/hbase-site.xml 
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://localhost:8020/hbase</value>
  </property>

  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>

  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>localhost</value>
  </property>

  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>

  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>

  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/hadoop/zookeeper</value>
  </property>

  <property>
    <name>hbase.tmp.dir</name>
    <value>./tmp</value>
  </property>

</configuration>
```

## Crie o diretorio de dados no HDFS

```
$ hadoop fs -mkdir /hbase
```

## inicie todos os servicos Hadoop

```
$ start-all.sh 
```

## confira com jps

```
$ jps

1938 SecondaryNameNode
1780 DataNode
2580 Jps
1655 NameNode
2073 ResourceManager
2335 NodeManager

```

