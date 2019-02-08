
# HBase Master

[HMaster](http://hbase.apache.org/book.html#_master) is the implementation of the Master Server.
The Master server is responsible for monitoring all RegionServer instances in the cluster, and is the interface for all metadata changes.
In a distributed cluster, the Master typically runs on the NameNode.
J Mohamed Zahoor goes into some more detail on the Master Architecture in this blog posting, [HBase HMaster Architecture](http://blog.zahoor.in/2012/08/hbase-hmaster-architecture/)

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service'
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs', local: true, required: true
      configure:
        'ryba-ambari-takeover/hbase/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hbase/service/install'
        ]
        'deploy': [
          'ryba-ambari-takeover/hbase/service/install'
        ]