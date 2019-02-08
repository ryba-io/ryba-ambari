
# WebHCat

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) is a REST API for HCatalog. (REST stands for "representational state transfer", a style of API based on HTTP verbs).  The original name of WebHCat was Templeton.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        db_admin: module: 'ryba/commons/db_admin'
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog', required: true
        hive_client: module: 'ryba-ambari-takeover/hive/client', local: true, auto: true, implicit: true
        hive_webhcat: module: 'ryba-ambari-takeover/hive/webhcat'
        sqoop: module: 'ryba/sqoop'
        log4j: module: 'ryba/log4j', local: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
      configure:
        'ryba-ambari-takeover/hive/webhcat/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hive/webhcat/check'

        'install': [
          'ryba-ambari-takeover/hive/webhcat/install'
        ]
        'start':
          'ryba-ambari-takeover/hive/webhcat/start'
        'status':
          'ryba-ambari-takeover/hive/webhcat/status'
        'stop':
          'ryba-ambari-takeover/hive/webhcat/stop'
