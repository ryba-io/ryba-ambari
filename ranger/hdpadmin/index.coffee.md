
# Ranger Policy Manager

Apache Ranger offers a centralized security framework to manage fine-grained
access control over Hadoop data access components like Apache Hive and Apache HBase.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        mysql_client: module: 'masson/commons/mysql/client', local: true
        mariadb_client: module: 'masson/commons/mariadb/client', local: true, auto: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        # remove ryba/solr/cloud from logs destination
        # keep only solr embedded and solr/cloud_docker
        # solr_cloud: module: 'ryba/solr/cloud'
        security: module: 'ryba-ambari-takeover/hadoop/security', required: true, local: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        solr_client: module: 'ryba/solr/client', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        hdp: module: 'ryba/hdp'
      configure:
        'ryba-ambari-takeover/ranger/hdpadmin/configure'
      commands:
        'install': [
          # 'ryba-ambari-takeover/ranger/solr/install'
          # 'ryba-ambari-takeover/ranger/solr/bootstrap'
          'ryba-ambari-takeover/ranger/hdpadmin/install'
          'ryba-ambari-takeover/ranger/hdpadmin/start'
          'ryba-ambari-takeover/ranger/hdpadmin/setup'
        ]
        'start':
          'ryba-ambari-takeover/ranger/hdpadmin/start'
        'status':
          'ryba-ambari-takeover/ranger/hdpadmin/status'
        'stop':
          'ryba-ambari-takeover/ranger/hdpadmin/stop'
