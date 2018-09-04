
# Hadoop HDFS NameNode Stop


    module.exports = header: 'HDFS NN Ambari Stop', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop Service

Stop the HDFS Namenode service with ambari REST Api. You can also stop the server manually with one of
the following two commands:

```
ps aef | grep namenode
kill 0 pid
```

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'NAMENODE'
        hostname: options.fqdn


## Stop Clean Logs

Remove the "\*-namenode-\*" log files if the property "ryba.clean_logs" is
activated.

      @system.execute
        header: 'Clean Logs'
        cmd: "rm #{options.log_dir}/*-namenode-*"
        code_skipped: 1
        if: options.clean_logs
