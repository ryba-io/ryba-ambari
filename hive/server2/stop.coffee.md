
# Hive Server2 Stop

Run the command `./bin/ryba stop -m ryba-ambari-takeover/hive/server2` to stop the Hive Server2
server using Ryba.

    module.exports = header: 'Ambari Hive Server2 Stop', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## System

You can also stop the server manually with one of the following two commands:

```
su -l hive -c "kill `cat /var/run/hive/hive-server2.pid`"
```
      
      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_SERVER'
        hostname: options.fqdn


## Clean Logs

Remove the "*" log file if the property "clean_logs" is
activated.

      @system.execute
        header: 'Stop Clean Logs'
        if: -> options.clean_logs
        cmd: "rm #{options.log_dir}/*"
        code_skipped: 1
