# Ranger HiveServer2 Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        atlas: module: 'ryba-ambari-takeover/atlas/service', required: true
        atlas_server: module: 'ryba-ambari-takeover/atlas/server', local: true, required: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs'
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client'
      configure:
        'ryba-ambari-takeover/ranger/plugins/atlas/configure'
