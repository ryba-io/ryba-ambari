
# Atlas Service

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        atlas: module: 'ryba-ambari-takeover/atlas/service'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs', local: true, required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/atlas/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/atlas/service/install'
        ]