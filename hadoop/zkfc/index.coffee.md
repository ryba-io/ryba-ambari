
# Hadoop ZKFC

The [ZKFailoverController (ZKFC)](https://hadoop.apache.org/docs/r2.3.0/hadoop-yarn/hadoop-yarn-site/HDFSHighAvailabilityWithQJM.html) is a new component which is a ZooKeeper client which also monitors and manages the state of the NameNode.
 Each of the machines which runs a NameNode also runs a ZKFC, and that ZKFC is responsible for Health monitoring, ZooKeeper session management, ZooKeeper-based election.


    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hdfs_nn_local: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        zkfc: module: 'ryba-ambari-takeover/hadoop/zkfc'
      configure:
        'ryba-ambari-takeover/hadoop/zkfc/configure'
      plugin: ({options}) ->
        @after
          type: ['ambari', 'hosts', 'component_start']
          name: 'NAMENODE'
        , ->
          @call 'ryba-ambari-takeover/hadoop/zkfc/install', options
          @call 'ryba-ambari-takeover/hadoop/zkfc/start', options
      commands:
        'install': [
          'ryba-ambari-takeover/hadoop/zkfc/install'
          'ryba-ambari-takeover/hadoop/zkfc/start'
          'ryba-ambari-takeover/hadoop/zkfc/check'
        ]
        'start': 'ryba-ambari-takeover/hadoop/zkfc/start'
        'stop': 'ryba-ambari-takeover/hadoop/zkfc/stop'
        'check': 'ryba-ambari-takeover/hadoop/zkfc/check'
        'status': 'ryba-ambari-takeover/hadoop/zkfc/status'
        'takeover': [
          'ryba-ambari-takeover/hadoop/zkfc/wait'
          'ryba-ambari-takeover/hadoop/zkfc/install'
          'ryba-ambari-takeover/hadoop/zkfc/takeover'
          'ryba-ambari-takeover/hadoop/zkfc/start'
          'ryba-ambari-takeover/hadoop/zkfc/wait'
          'ryba-ambari-takeover/hadoop/zkfc/check'
        ]

