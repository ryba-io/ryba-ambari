
# Hive HCatalog Stop

Stop the Hive HCatalog server.

The file storing the PID is "/var/run/hive-server2/hive-server2.pid".

    module.exports = header: 'Ambari Hive HCatalog Stop', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service

You can also stop the server manually with one of
the following two commands:

```
su -l hive -c "kill `cat /var/run/hive/hcat.pid`"
```

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_METASTORE'
        hostname: options.fqdn

## Clean Logs

Remove the "*" log file if the property "clean_logs" is
activated.

      @system.execute
        header: 'Clean Logs'
        if: options.clean_logs
        cmd: "rm #{options.log_dir}/*"
        code_skipped: 1
