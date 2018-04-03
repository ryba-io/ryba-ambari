
# Hadoop YARN Timeline Server Stop

Stop the HDFS Namenode service. You can also stop the server manually with one of
the following two commands:

```
service hadoop-yarn-timelineserver stop
su -l yarn -c "/usr/hdp/current/hadoop-yarn-timelineserver/sbin/yarn-daemon.sh --config /etc/hadoop/conf stop timelineserver"
```

The file storing the PID is "/var/run/hadoop-yarn/yarn/yarn-yarn-timelineserver.pid".

    module.exports = header: 'YARN ATS Ambari Stop', handler: ->
    
## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop Service

Stop the TIMELINESERVER YARN's service component. 
```
  curl ...
```

You can also stop de service manually
```
ps aef | grep timelineserver
kill 0 pid
```

The file storing the PID is "/var/run/hadoop-yarn/yarn/yarn-yarn-nodemanager.pid".

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'APP_TIMELINE_SERVER'
        hostname: options.fqdn
    # module.exports.push header: 'Clean Logs', handler: ->
    #   {clean_logs, yarn}
    #   return unless clean_logs
    #   @system.execute
    #     cmd: 'rm #{yarn.log_dir}/*/*-nodemanager-*'
    #     code_skipped: 1
