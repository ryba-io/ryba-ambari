# Ranger Knox Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: use: true, module: 'ryba-ambari-takeover/hadoop/core', local: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        knox_server: module: 'ryba-ambari-takeover/knox/server', local: true
        knox: module: 'ryba-ambari-takeover/knox/service'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/knox/configure'
