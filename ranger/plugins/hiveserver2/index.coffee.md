# Ranger HiveServer2 Plugin

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog', required: true
        hive_server2: module: 'ryba-ambari-takeover/hive/server2', local: true, required: true
        hive: module: 'ryba-ambari-takeover/hive/service'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs'
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hiveserver2'
      configure:
        'ryba-ambari-takeover/ranger/plugins/hiveserver2/configure'
