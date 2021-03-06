
# Apache Spark JOB History Server

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        tez: module: 'ryba/tez', local: true
        ganglia_collector: module: 'ryba/retired/ganglia/collector'
        graphite: module: 'ryba/graphite/carbon'
        spark: module: 'ryba-ambari-takeover/spark2/service', required: true
        spark_local: module: 'ryba-ambari-takeover/spark2/service', required: true, local: true
      configure:
        'ryba-ambari-takeover/spark2/history/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/spark2/history/install'
          'ryba-ambari-takeover/spark2/history/start'
          'ryba-ambari-takeover/spark2/history/check'
        ]
        'start':
          'ryba-ambari-takeover/spark2/service/start'
        'stop':
          'ryba-ambari-takeover/spark2/service/stop'
        'check':
          'ryba-ambari-takeover/spark2/service/check'

[tips]: https://www.altiscale.com/hadoop-blog/spark-on-hadoop/
