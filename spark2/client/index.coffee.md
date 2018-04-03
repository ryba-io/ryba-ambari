
# Apache Sparkn Client Package

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hiveserver2'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        tez: module: 'ryba/tez', local: true
        ganglia_collector: module: 'ryba/retired/ganglia/collector'
        graphite: module: 'ryba/graphite/carbon'
        spark: module: 'ryba-ambari-takeover/spark2/service', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/spark2/client/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/spark2/client/install'
          'ryba-ambari-takeover/spark2/client/check'
        ]
        'check':
          'ryba-ambari-takeover/spark2/client/check'

[tips]: https://www.altiscale.com/hadoop-blog/spark-on-hadoop/
