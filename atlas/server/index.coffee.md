# Apache Atlas 

[Atlas][atlas-server] is a scalable and extensible set of core foundational
governance services – enabling enterprises to effectively and efficiently meet
their compliance requirements within Hadoop and allows integration with the whole
enterprise data ecosystem.

Atlas enables Hadoop users to manage more efficiently their data:

- Data Classification
- Centralized auditing
- Search & Lineage
- Scurity & Policy Engine

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true, recommanded: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'
        # hbase_client: module: 'ryba-ambari-takeover/hbase/client', local: true, recommanded: true # Required if hbase_master
        hbase_client: module: 'ryba-ambari-takeover/hbase/client', local: true, auto: true # Required if hbase_master
        kafka_broker: module: 'ryba-ambari-takeover/kafka/broker'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        ranger_kafka: module: 'ryba-ambari-takeover/ranger/plugins/kafka'
        ranger_hbase: module: 'ryba-ambari-takeover/ranger/plugins/hbase'
        # solr_client: module: 'ryba-ambari-takeover/solr/client', local: true
        solr_cloud: module: 'ryba/solr/cloud_docker'
        solr_cloud_docker: module: 'ryba/solr/cloud_docker'
        # ranger_tagsync: module: 'ryba/ranger/tagsync'  # migration: wdavidw 171006, service does not exists
        atlas: module: 'ryba-ambari-takeover/atlas/service', required: true
      configure:
        'ryba-ambari-takeover/atlas/server/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/atlas/server/install'
        ]
        'start':
          'ryba-ambari-takeover/atlas/start'
        'status':
          'ryba-ambari-takeover/atlas/status'
        'check':
          'ryba-ambari-takeover/atlas/check'
        'stop':
          'ryba-ambari-takeover/atlas/stop'

[atlas-apache]: http://atlas.incubator.apache.org
