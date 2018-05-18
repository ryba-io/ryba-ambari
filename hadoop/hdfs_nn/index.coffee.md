
# Hadoop HDFS NameNode

NameNode’s primary responsibility is storing the HDFS namespace. This means things
like the directory tree, file permissions, and the mapping of files to block
IDs. It tracks where across the cluster the file data is kept on the DataNodes. It
does not store the data of these files itself. It’s important that this metadata
(and all changes to it) are safely persisted to stable storage for fault tolerance.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_jn: module: 'ryba-ambari-takeover/hadoop/hdfs_jn'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        ranger_admin: module: 'ryba/ranger/admin', single: true
        metrics: module: 'ryba/metrics', local: true
        log4j: module: 'ryba/log4j', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hadoop/hdfs_nn/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hadoop/hdfs_nn/install'
          'ryba-ambari-takeover/hadoop/hdfs_nn/start'
          'ryba-ambari-takeover/hadoop/hdfs_nn/layout'
          'ryba-ambari-takeover/hadoop/hdfs_nn/check'
        ]
        'start': 'ryba-ambari-takeover/hadoop/hdfs_nn/start'
        'stop': 'ryba-ambari-takeover/hadoop/hdfs_nn/stop'
        'check': 'ryba-ambari-takeover/hadoop/hdfs_nn/check'
        'status': 'ryba-ambari-takeover/hadoop/hdfs_nn/status'
        'takeover': [
          'ryba-ambari-takeover/hadoop/hdfs_nn/wait'
          'ryba-ambari-takeover/hadoop/hdfs_nn/install'
          'ryba-ambari-takeover/hadoop/hdfs_nn/takeover'
          'ryba-ambari-takeover/hadoop/hdfs_nn/start'
          'ryba-ambari-takeover/hadoop/hdfs_nn/wait'
          'ryba-ambari-takeover/hadoop/hdfs_nn/check'
        ]

[keys]: https://github.com/apache/hadoop-common/blob/trunk/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/DFSConfigKeys.java
