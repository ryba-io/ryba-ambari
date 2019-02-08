
# YARN Ambari Install

This modules aims at installing YARN service with ambari.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn'
      configure: 'ryba-ambari-takeover/hadoop/yarn/configure'
      commands:
        install: 'ryba-ambari-takeover/hadoop/yarn/install'
        prepare: 'ryba-ambari-takeover/hadoop/yarn/prepare'
        deploy: 'ryba-ambari-takeover/hadoop/yarn/install'

