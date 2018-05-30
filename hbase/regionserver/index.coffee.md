
# HBase RegionServer

[HRegionServer](http://hbase.apache.org/book.html#regionserver.arch) is the
RegionServer implementation.
It is responsible for serving and managing regions. 
In a distributed cluster, a RegionServer runs on a DataNode.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        hbase_local: module: 'ryba-ambari-takeover/hbase/service', required: true, local: true, implicit: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', local: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master', required: true
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver'
        ranger_admin: module: 'ryba/ranger/admin'
        ganglia_collector: module: 'ryba/retired/ganglia/collector'
        log4j: module: 'ryba/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hbase/regionserver/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hbase/regionserver/check'
        'install': [
          'ryba-ambari-takeover/hbase/regionserver/install'
          'ryba-ambari-takeover/hbase/regionserver/start'
          'ryba-ambari-takeover/hbase/regionserver/check'
        ]
        'start':
          'ryba-ambari-takeover/hbase/regionserver/start'
        'status':
          'ryba-ambari-takeover/hbase/regionserver/status'
        'stop':
          'ryba-ambari-takeover/hbase/regionserver/stop'
        'takeover': [
          'ryba-ambari-takeover/hbase/regionserver/takeover'
          'ryba-ambari-takeover/hbase/regionserver/install'
          'ryba-ambari-takeover/hbase/regionserver/start'
          'ryba-ambari-takeover/hbase/regionserver/check'
        ]
          
