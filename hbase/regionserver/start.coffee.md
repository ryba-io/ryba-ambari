
# HBase RegionServer Start

Start the RegionServer server. You can also start the server manually with one of the
following two commands:

```
service hbase-regionserver start
su -l hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh --config /etc/hbase-regionserver/conf start regionserver"
```

    module.exports = header: 'HBase RegionServer Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

Wait for Kerberos, ZooKeeper, HDFS and HBase Master to be started.

      @call 'masson/core/krb5_client/wait', once: true,  options.wait_krb5_client
      @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true,  options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hadoop/hdfs_dn/wait', once: true
      # @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true,  options.wait_hdfs_nn, conf_dir: options.hdfs_conf_dir
      @call 'ryba-ambari-takeover/hbase/master/wait', once: true, options.wait_hbase_master

Start the service.


      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_REGIONSERVER'
        hostname: options.fqdn
