
# Oozie Server Start

Run the command `./bin/ryba start -m ryba/oozie/server` to start the Oozie
server using Ryba.

By default, the pid of the running server is stored in
"/var/run/oozie/oozie.pid".

Start the Oozie server. You can also start the server manually with the
following command:

```
service oozie start
su -l oozie -c "/usr/hdp/current/oozie-server/bin/oozied.sh start"
```

Note, there is no need to clean a zombie pid file before starting the server.

    module.exports = header: 'Oozie Server Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

Wait for all the dependencies.

      # @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      # @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      # @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.hadoop_conf_dir
      # @call 'ryba-ambari-takeover/hbase/master/wait', once: true, options.wait_hbase_master
      # @call 'ryba-ambari-takeover/hive/hcatalog/wait', once: true, options.wait_hive_hcatalog
      # @call 'ryba-ambari-takeover/hive/server2/wait', once: true, options.wait_hive_server2
      # @call 'ryba-ambari-takeover/hive/webhcat/wait', once: true, options.wait_hive_webhcat

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'OOZIE_SERVER'
        hostname: options.fqdn