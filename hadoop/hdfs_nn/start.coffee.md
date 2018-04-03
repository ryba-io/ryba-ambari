
# Hadoop HDFS NameNode Start

Start the NameNode service as well as its ZKFC daemon. In HA mode, all
JournalNodes shall be previously started.

    module.exports = header: 'HDFS NN Ambari Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

      @call 'ryba-ambari-takeover/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hadoop/hdfs_jn/wait', once: true, options.wait_hdfs_jn

## Service

You can also start the server manually with the following two commands:

```
curl 
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'NAMENODE'
        hostname: options.fqdn


