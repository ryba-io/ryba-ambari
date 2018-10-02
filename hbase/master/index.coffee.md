
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
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        hbase_local: module: 'ryba-ambari-takeover/hbase/service', required: true, local: true, implicit: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        # hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', required: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        ranger_admin: module: 'ryba/ranger/admin', single: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'
        ganglia_collector: module: 'ryba/retired/ganglia/collector', single: true
        metrics: module: 'ryba/metrics', local: true
        log4j: module: 'ryba/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hbase/master/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hbase/master/check'
        'install': [
          'ryba-ambari-takeover/hbase/master/install'
          'ryba-ambari-takeover/hbase/master/start'
          'ryba-ambari-takeover/hbase/master/layout'
          'ryba-ambari-takeover/hbase/master/check'

        ]
        'start':
          'ryba-ambari-takeover/hbase/master/start'
        'stop':
          'ryba-ambari-takeover/hbase/master/stop'
