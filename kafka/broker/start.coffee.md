
# Kafka Broker Start

Start the Kafka Broker.

    module.exports = header: 'Kafka Broker Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos and ZooKeeper.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba/zookeeper/server/wait', once: true, options.wait_zookeeper_server

## service.

You can also start the server manually with the following commands:

```
service kafka-broker start
systemctl start kafka-broker
su - kafka -c '/usr/hdp/current/kafka-broker/bin/kafka start'
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'KAFKA_BROKER'
        hostname: options.fqdn
