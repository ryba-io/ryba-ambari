
# Hadoop HDFS Client

[Clients][hdfs_client] contact NameNode for file metadata or file modifications
and perform actual file I/O directly with the DataNodes.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true, recommended: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        # hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        log4j: module: 'ryba/log4j'
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
      configure:
        'ryba-ambari-takeover/hadoop/hdfs_client/configure'
      commands:
        'check': 'ryba-ambari-takeover/hadoop/hdfs_client/check'

[hdfs_client]: http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html
