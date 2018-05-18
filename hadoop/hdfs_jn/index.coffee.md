
# Hadoop HDFS JournalNode

This module configure the JournalNode following the 
[HDFS High Availability Using the Quorum Journal Manager](https://hadoop.apache.org/docs/r2.3.0/hadoop-yarn/hadoop-yarn-site/HDFSHighAvailabilityWithQJM.html) official 
recommandations.

In order for the Standby node to keep its state synchronized with the Active 
node, both nodes communicate with a group of separate daemons called 
"JournalNodes" (JNs). When any namespace modification is performed by the Active 
node, it durably logs a record of the modification to a majority of these JNs. 
The Standby node is capable of reading the edits from the JNs, and is constantly 
watching them for changes to the edit log.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_jn: module: 'ryba-ambari-takeover/hadoop/hdfs_jn'
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        metrics: module: 'ryba/metrics', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hadoop/hdfs_jn/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hadoop/hdfs_jn/install'
          'ryba-ambari-takeover/hadoop/hdfs_jn/start'
          'ryba-ambari-takeover/hadoop/hdfs_jn/check'
        ]
        'takeover': [
          'ryba-ambari-takeover/hadoop/hdfs_jn/wait'
          'ryba-ambari-takeover/hadoop/hdfs_jn/install'
          'ryba-ambari-takeover/hadoop/hdfs_jn/takeover'
          'ryba-ambari-takeover/hadoop/hdfs_jn/start'
          'ryba-ambari-takeover/hadoop/hdfs_jn/wait'
          'ryba-ambari-takeover/hadoop/hdfs_jn/check'
        ]
        'start': 'ryba-ambari-takeover/hadoop/hdfs_jn/start'
        'stop': 'ryba-ambari-takeover/hadoop/hdfs_jn/stop'
        'check': 'ryba-ambari-takeover/hadoop/hdfs_jn/check'
        'status': 'ryba-ambari-takeover/hadoop/hdfs_jn/status'
[qjm]: http://hadoop.apache.org/docs/r2.3.0/hadoop-yarn/hadoop-yarn-site/HDFSHighAvailabilityWithQJM.html#Architecture
