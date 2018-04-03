
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
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
        log4j: module: 'ryba/log4j', local: true
      configure:
        'ryba-ambari-takeover/hive/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hive/service/install'
        ]

