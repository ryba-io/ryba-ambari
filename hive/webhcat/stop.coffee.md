
# WebHCat Stop

Run the command `./bin/ryba stop -m ryba-ambari-takeover/hive/webhcat` to stop the WebHCat
server using Ryba.

    module.exports = header: 'WebHCat Stop', handler: ({options}) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service

Stop the WebHCat server. You can also stop the server manually with one of the
following two commands:

```
su -l hive -c "/usr/hdp/current/hive-webhcat/sbin/webhcat_server.sh stop"
```

The file storing the PID is "/var/run/webhcat/webhcat.pid".

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'WEBHCAT_SERVER'
        hostname: options.fqdn

## Clean Logs

Remove the "webhcat-console*" log files if the property "clean_logs" is
activated.

      @system.execute
        header: 'Clean Console Logs'
        if: options.clean_logs
        cmd: "rm #{options.log_dir}/webhcat-console*"
        code_skipped: 1

Remove the "webhcat.*" log files if the property "clean_logs" is
activated.

      @system.execute
        header: 'Clean Logs'
        if: options.clean_logs
        cmd: "rm #{options.log_dir}/webhcat.*"
        code_skipped: 1
