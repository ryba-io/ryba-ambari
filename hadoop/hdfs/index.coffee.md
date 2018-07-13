
# Hadoop takeover

This modules aims at installing HDFS service with ambari.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        ambari_agent: module: 'ryba-ambari-takeover/agent', required: true
      configure: 'ryba-ambari-takeover/hadoop/hdfs/configure'
      commands:
        install: 'ryba-ambari-takeover/hadoop/hdfs/install'
        prepare: 'ryba-ambari-takeover/hadoop/hdfs/prepare'
        deploy: 'ryba-ambari-takeover/hadoop/hdfs/install'
