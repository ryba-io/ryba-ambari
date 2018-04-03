
# YARN NodeManager Stop

    module.exports = header: 'YARN NM Ambari Ambari Stop', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop Service

Stop the NODEMANAGER YARN's service component. 
```
  curl ...
```

You can also stop de service manually
```
ps aef | grep nodemanager
kill 0 pid
```

The file storing the PID is "/var/run/hadoop-yarn/yarn/yarn-yarn-nodemanager.pid".

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'NODEMANAGER'
        hostname: options.fqdn

## Stop Clean Logs

Remove the "\*-nodemanager-\*" log files if the property "ryba.clean_logs" is
activated.

      @system.execute
        header: 'YARN NM Ambari Clean Logs'
        if: options.clean_logs
        cmd: 'rm #{options.log_dir}/*/*-nodemanager-*'
        code_skipped: 1
