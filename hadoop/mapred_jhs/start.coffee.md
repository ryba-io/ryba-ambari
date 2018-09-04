
# MapReduce JobHistoryServer (JHS) Start

Start the MapReduce Job History Server.

It is recommended but not required to start the JHS server before the Resource
Manager. If started after after, the ResourceManager will print a message in the
log file complaining it cant reach the JSH server (default port is "10020").

    module.exports = header: 'Mapreduce Ambari JHS Start', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

Wait for the DataNode and NameNode to be started to fetch all history.

      @call once: true, 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', options.wait_hdfs_nn, conf_dir: options.hadoop_conf_dir, hdfs_krb5_user: options.hdfs_krb5_user

## Service

You can also start the server manually with the following command:

```
curl 
```

## Service Start

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        hostname: options.fqdn
