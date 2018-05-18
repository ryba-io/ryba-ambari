
# Zookeeper Server

Setting up a ZooKeeper server in standalone mode or in replicated mode.

A replicated group of servers in the same application is called a quorum, and in
replicated mode, all servers in the quorum have copies of the same configuration
file. The file is similar to the one used in standalone mode, but with a few
differences.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        hdp: module: 'ryba/hdp', local: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        log4j: module: 'ryba/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure: 'ryba-ambari-takeover/zookeeper/server/configure'
      commands:
        'check':
          'ryba-ambari-takeover/zookeeper/server/check'
        'start':
          'ryba-ambari-takeover/zookeeper/server/start'
        'install': [
          'ryba-ambari-takeover/zookeeper/server/install'
          'ryba-ambari-takeover/zookeeper/server/start'
          'ryba-ambari-takeover/zookeeper/server/check'
        ]
        'takeover': [
          'ryba-ambari-takeover/zookeeper/server/wait'
          'ryba-ambari-takeover/zookeeper/server/install'
          'ryba-ambari-takeover/zookeeper/server/takeover'
          'ryba-ambari-takeover/zookeeper/server/start'
          'ryba-ambari-takeover/zookeeper/server/wait'
          'ryba-ambari-takeover/zookeeper/server/check'
        ]
