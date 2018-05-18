
# Zookeeper Server Start

Start the ZooKeeper server. You can also start the server manually with the
following two commands:

```
service zookeeper-server start
su - zookeeper -c "export ZOOCFGDIR=/usr/hdp/current/zookeeper-server/conf; export ZOOCFG=/etc/zookeeper/conf/zoo.cfg; source /usr/hdp/current/zookeeper-server/conf/zookeeper-env.sh; /usr/hdp/current/zookeeper-server/bin/zkServer.sh start"
```

    module.exports = header: 'ZooKeeper Server Start', handler: (options) ->

## Regitry

      @registry.register ['ambari', 'hosts', 'component_start'], "ryba-ambari-actions/lib/hosts/component_start"

Wait for Kerberos to be started.
      
      @call 'masson/core/krb5_client/wait', once:true, options.wait_krb5_client

## Start the service.

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_SERVER'
        hostname: options.fqdn

