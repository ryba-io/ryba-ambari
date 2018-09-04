# Ranger Kafka Plugin

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        kafka_broker: module: 'ryba-ambari-takeover/kafka/broker', local: true, required: true
        kafka_service: module: 'ryba-ambari-takeover/kafka/service', required: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs', required: true
        ranger_kafka: module: 'ryba-ambari-takeover/ranger/plugins/kafka'
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/kafka/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'KAFKA_BROKER'
        , ->
          @call 'ryba-ambari-takeover/ranger/plugins/kafka/install', options
        # @after 'ryba/kafka/broker/install', ->
        #   @call 'ryba/ranger/plugins/kafka/install', options
