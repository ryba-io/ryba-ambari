
# Ranger Policy Manager

Apache Ranger offers a centralized security framework to manage fine-grained
access control over Hadoop data access components like Apache Hive and Apache HBase.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        mysql_client: module: 'masson/commons/mysql/client', local: true
        mariadb_client: module: 'masson/commons/mariadb/client', local: true, auto: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        solr_client: module: 'ryba/solr/client', local: true
        hdp: module: 'ryba/hdp'
      configure:
        'ryba-ambari-takeover/ranger/hdfadmin/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/ranger/hdfadmin/install'
          'ryba-ambari-takeover/ranger/hdfadmin/start'
          'ryba-ambari-takeover/ranger/hdfadmin/setup'
        ]
        'start':
          'ryba-ambari-takeover/ranger/hdfadmin/start'
        'status':
          'ryba-ambari-takeover/ranger/hdfadmin/status'
        'stop':
          'ryba-ambari-takeover/ranger/hdfadmin/stop'
        'takeover': [
          'ryba-ambari-takeover/ranger/hdfadmin/wait'
          'ryba-ambari-takeover/ranger/hdfadmin/install'
          'ryba-ambari-takeover/ranger/hdfadmin/takeover'
          'ryba-ambari-takeover/ranger/hdfadmin/start'
          'ryba-ambari-takeover/ranger/hdfadmin/wait'
        ]
