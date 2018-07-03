
# Apache Spark JOB Thrift Server

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        tez: module: 'ryba/tez', local: true
        ganglia_collector: module: 'ryba/retired/ganglia/collector'
        graphite: module: 'ryba/graphite/carbon'
        spark: module: 'ryba-ambari-takeover/spark/service', required: true
        spark_local: module: 'ryba-ambari-takeover/spark/service', required: true, local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/spark/thrift/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/spark/thrift/install'
          'ryba-ambari-takeover/spark/thrift/start'
          'ryba-ambari-takeover/spark/thrift/check'
        ]
        'check':
          'ryba-ambari-takeover/spark/thrift/check'

[tips]: https://www.altiscale.com/hadoop-blog/spark-on-hadoop/
