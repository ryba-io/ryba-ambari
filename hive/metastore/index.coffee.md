
# Hive Metastore

Hive Metastore is a middleware for persisting and accessing Hadoop metadata.
Apache Impala, Spark, Drill, Presto, and other systems all use Hive’s metastore. 
Some, like Impala and Presto can use it as their own metadata system with the
rest of Hive not present.

Metastore’s table abstraction presents users with a relational view of data in the Hadoop
distributed file system (HDFS) and ensures that users need not worry about where or in what
format their data is stored — RCFile format, text files, SequenceFiles, or ORC files.

    module.exports =
      deps:
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
      configure:
        'ryba-ambari-takeover/hive/metastore/configure'
      commands:
        'install':
          'ryba-ambari-takeover/hive/metastore/install'
        'backup':
          'ryba-ambari-takeover/hive/metastore/backup'
