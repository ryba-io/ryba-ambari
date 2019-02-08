
# Mapreduce Service Ambari Install

This modules aims at installing MAPREDUCE service in Ambari.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', local: true, required: true
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn', local: true, required: true
      configure: 'ryba-ambari-takeover/hadoop/mapreduce/configure'
      commands:
        install: 'ryba-ambari-takeover/hadoop/mapreduce/install'
        deploy: 'ryba-ambari-takeover/hadoop/mapreduce/install'
