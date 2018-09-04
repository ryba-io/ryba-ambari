
# HBase Start

Start the HBase Master server.

    module.exports = header: 'Ambari HBase Master Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for Kerberos, ZooKeeper and HDFS to be started.

      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.hdfs_conf_dir, hdfs_krb5_user: options.hdfs_krb5_user

## Service

You can also start the server manually with one of the following two commands:

```
service hbase-master start
systemctl start hbase-master
su -l hbase -c "/usr/hdp/current/hbase-master/bin/hbase-daemon.sh --config /etc/hbase-master/conf start master"
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_MASTER'
        hostname: options.fqdn

