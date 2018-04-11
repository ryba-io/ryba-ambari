
# Hive HCatalog

[HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog+UsingHCat) 
is a table and storage management layer for Hadoop that enables users with different 
data processing tools — Pig, MapReduce — to more easily read and write data on the grid.

HCatalog’s table abstraction presents users with a relational view of data in the Hadoop
distributed file system (HDFS) and ensures that users need not worry about where or in what
format their data is stored — RCFile format, text files, SequenceFiles, or ORC files.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        mapred_client: module: 'ryba-ambari-takeover/hadoop/mapred_client', local: true, auto: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        tez: module: 'ryba-ambari-takeover/tez', local: true, auto: true, implicit: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
        hive_metastore: module: 'ryba-ambari-takeover/hive/metastore', local: true, auto: true, implicit: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        hbase_client: module: 'ryba-ambari-takeover/hbase/client', local: true, recommanded: true
        log4j: module: 'ryba/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/hive/hcatalog/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hive/hcatalog/install'
          'ryba-ambari-takeover/hive/hcatalog/start'
          'ryba-ambari-takeover/hive/hcatalog/check'
        ]
        'check':
          'ryba-ambari-takeover/hive/hcatalog/check'
        'start':
          'ryba-ambari-takeover/hive/hcatalog/start'
        'status':
          'ryba-ambari-takeover/hive/hcatalog/status'
        'stop':
          'ryba-ambari-takeover/hive/hcatalog/stop'
        'report': [
          'masson/bootstrap/report'
          'ryba-ambari-takeover/hive/hcatalog/report'
        ]
        'backup':
          'ryba-ambari-takeover/hive/hcatalog/backup'
