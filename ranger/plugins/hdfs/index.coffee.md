
# Ranger HDFS Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
      configure:
        'ryba-ambari-takeover/ranger/plugins/hdfs/configure'
