
# Hive Beeline (Server2 Client)

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog', required: true
        hive_server2: module: 'ryba-ambari-takeover/hive/server2', required: true
        spark_thrift_server: module: 'ryba/spark/thrift_server'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hiveserver2'
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
      configure:
        'ryba-ambari-takeover/hive/beeline/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hive/beeline/install'
          'ryba-ambari-takeover/hive/beeline/check'
        ]
        'check':
          'ryba-ambari-takeover/hive/beeline/check'
        'deploy':
          'ryba-ambari-takeover/hive/beeline/install'
