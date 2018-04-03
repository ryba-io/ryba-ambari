
# Hive Server2 Start

The Hive HCatalog require the database server to be started. The Hive Server2
require the HDFS Namenode to be started. Both of them will need to functionnal
HDFS server to answer queries.

    module.exports = header: 'Ambari Hive Server2 Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos, Zookeeper, Hadoop and Hive HCatalog.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba/zookeeper/server/wait', once: true, options.wait_zookeeper_server

## Service

Start the Hive Server2. You can also start the server manually with one of the
following two commands:

```
su hive -l -s /bin/bash -c 'cat /var/run/hive/hive-server.pid 1>/tmp/tmpFJSTfP 2>/tmp/tmpSKLpIL'
su -l hive -c 'nohup /usr/hdp/current/hive-server2/bin/hiveserver2 >/var/log/hive/hiveserver2.out 2>/var/log/hive/hiveserver2.log & echo $! >/var/run/hive-server2/hive-server2.pid'
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_SERVER'
        hostname: options.fqdn