
# Spark Livy

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        ambari_service: module: 'ryba/ambari/server', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent', required: true
        spark_client: module: 'ryba-ambari-takeover/spark2/client', required: true
        spark_service: module: 'ryba-ambari-takeover/spark2/service', required: true
      configure:
        'ryba-ambari-takeover/spark2/livy/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/spark2/livy/install'
        ]

[Ambari-server]: http://ambari.apache.org
