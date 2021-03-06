
# Ambari Client

[Ambari-agent][Ambari-agent-install] on hosts enables the ambari server to be
aware of the  hosts where Hadoop will be deployed. The Ambari Server must be
installed before the agent registration.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        java: module: 'masson/commons/java', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core'
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        hdfs_jn: module: 'ryba-ambari-takeover/hadoop/hdfs_jn'
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs_zkfc: module: 'ryba-ambari-takeover/hadoop/zkfc'
        httpfs: module: 'ryba-ambari-takeover/hadoop/httpfs'
        mapreduce: module: 'ryba-ambari-takeover/hadoop/mapreduce'
        mapred_client: module: 'ryba-ambari-takeover/hadoop/mapred_client'
        mapred_jhs: module: 'ryba-ambari-takeover/hadoop/mapred_jhs'
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn'
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        hbase_service: module: 'ryba-ambari-takeover/hbase/service'
        hbase_client: module: 'ryba-ambari-takeover/hbase/client'
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver'
        hbase_rest: module: 'ryba-ambari-takeover/hbase/rest'
        hive: module: 'ryba-ambari-takeover/hive/service'
        hive_client: module: 'ryba-ambari-takeover/hive/client'
        hive_beeline: module: 'ryba-ambari-takeover/hive/beeline'
        hive_metastore: module: 'ryba-ambari-takeover/hive/metastore'
        hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        webhcat: module: 'ryba-ambari-takeover/hive/webhcat'
        kafka_service: module: 'ryba-ambari-takeover/kafka/service'
        kafka_broker: module: 'ryba-ambari-takeover/kafka/broker'
        knox_service: module: 'ryba-ambari-takeover/knox/service'
        knox_server: module: 'ryba-ambari-takeover/knox/server'
        oozie_service: module: 'ryba-ambari-takeover/oozie/service'
        oozie_client: module: 'ryba-ambari-takeover/oozie/client'
        oozie_server: module: 'ryba-ambari-takeover/oozie/server'
        phoenix_client: module: 'ryba-ambari-takeover/phoenix/client'
        phoenix_queryserver: module: 'ryba-ambari-takeover/phoenix/queryserver'
        ranger_hdpadmin: module: 'ryba-ambari-takeover/ranger/hdpadmin'
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs'
        ranger_yarn: module: 'ryba-ambari-takeover/ranger/plugins/yarn'
        ranger_hbase: module: 'ryba-ambari-takeover/ranger/plugins/hbase'
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hiveserver2'
        ranger_knox: module: 'ryba-ambari-takeover/ranger/plugins/knox'
        ranger_kafka: module: 'ryba-ambari-takeover/ranger/plugins/kafka'
        ranger_atlas: module: 'ryba-ambari-takeover/ranger/plugins/atlas'
        ambari_infra_service: module: 'ryba-ambari-takeover/ambari_infra/service'
        ambari_infra_instance: module: 'ryba-ambari-takeover/ambari_infra/instance'
        ambari_metrics_service: module: 'ryba-ambari-takeover/metrics/service'
        ambari_metrics_collector: module: 'ryba-ambari-takeover/metrics/collector'
        ambari_metrics_monitor: module: 'ryba-ambari-takeover/metrics/monitor'
        ambari_metrics_grafana: module: 'ryba-ambari-takeover/metrics/grafana'
        spark_service: module: 'ryba-ambari-takeover/spark/service'
        spark_client: module: 'ryba-ambari-takeover/spark/client'
        spark_hs: module: 'ryba-ambari-takeover/spark/history'
        spark_client_2: module: 'ryba-ambari-takeover/spark2/client'
        spark_hs_2: module: 'ryba-ambari-takeover/spark2/history_server'
        sqoop: module: 'ryba-ambari-takeover/sqoop'
        tez: module: 'ryba-ambari-takeover/tez'
        zookeeper_client: module: 'ryba-ambari-takeover/zookeeper/client'
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        zeppelin_service: module: 'ryba-ambari-takeover/zeppelin/service'
        zeppelin_master: module: 'ryba-ambari-takeover/zeppelin/master'
        logsearch_service: module: 'ryba-ambari-takeover/logsearch/service'
        logsearch_server: module: 'ryba-ambari-takeover/logsearch/server'
        logsearch_feeder: module: 'ryba-ambari-takeover/logsearch/feeder'
        smartsense_server: 'ryba-ambari-takeover/smartsense/server'
        smartsense_agent: 'ryba-ambari-takeover/smartsense/agent'
        smartsense_service: 'ryba-ambari-takeover/smartsense/service'
        smartsense_explorer: module: 'ryba-ambari-takeover/smartsense/explorer'
      configure:
        'ryba-ambari-takeover/agent/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/agent/install'
        ]

[Ambari-agent-install]: https://cwiki.apache.org/confluence/display/AMBARI/Installing+ambari-agent+on+target+hosts
