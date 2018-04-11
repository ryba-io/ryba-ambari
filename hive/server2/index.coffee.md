
# Hive Server2

HiveServer2 (HS2) is a server interface that enables remote clients to execute
queries against Hive and retrieve the results. The current implementation, based
on Thrift RPC, is an improved version of HiveServer and supports multi-client
concurrency and authentication. It is designed to provide better support for
open API clients like JDBC and ODBC.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        tez: module: 'ryba-ambari-takeover/tez', local: true, auto: true, implicit: true
        hive_metastore: module: 'ryba-ambari-takeover/hive/metastore', local: true, auto: true, implicit: true
        hive_hcatalog_local: module: 'ryba-ambari-takeover/hive/hcatalog', local: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog', required: true
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        hive_client: module: 'ryba-ambari-takeover/hive/client'
        hbase_thrift: module: 'ryba-ambari-takeover/hbase/thrift'
        hbase_client: module: 'ryba-ambari-takeover/hbase/client', local: true
        phoenix_client: module: 'ryba/ambari/phoenix/client'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        log4j: module: 'ryba/log4j', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hive/server2/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hive/server2/install'
          'ryba-ambari-takeover/hive/server2/start'
          'ryba-ambari-takeover/hive/server2/check'
        ]
        'start':
          'ryba-ambari-takeover/hive/server2/start'
        'check':
          'ryba-ambari-takeover/hive/server2/check'
        'status':
          'ryba-ambari-takeover/hive/server2/status'
        'stop':
          'ryba-ambari-takeover/hive/server2/stop'
        'backup':
          'ryba-ambari-takeover/hive/server2/backup'
