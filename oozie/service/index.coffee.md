
# Oozie Service


    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        oozie: module: 'ryba-ambari-takeover/oozie/service', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
        log4j: module: 'ryba/log4j', local: true
      configure:
        'ryba-ambari-takeover/oozie/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/oozie/service/install'
        ]

