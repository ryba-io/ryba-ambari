
# Kafka Broker

Apache Kafka is publish-subscribe messaging rethought as a distributed commit
log. It is fast, scalable, durable and distributed by design.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hdp: module: 'ryba/hdp', local: true
        hdf: module: 'ryba/hdf', local: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        kafka_broker: module: 'ryba-ambari-takeover/kafka/broker'
        kafka_service: module: 'ryba-ambari-takeover/kafka/service', required: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
      configure:
        'ryba-ambari-takeover/kafka/broker/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/kafka/broker/install'
          'ryba-ambari-takeover/kafka/broker/start'
          'ryba-ambari-takeover/kafka/broker/check'
        ]
        'check':
          'ryba-ambari-takeover/kafka/broker/check'
        'start':
          'ryba-ambari-takeover/kafka/broker/start'
        'stop':
          'ryba-ambari-takeover/kafka/broker/stop'
        'status':
          'ryba-ambari-takeover/kafka/broker/status'
