
# MapReduce JobHistoryServer Stop

Stop the MapReduce Job History Password. You can also stop the server manually
with one of the following two commands:

```
service hadoop-mapreduce-historyserver stop
systemctl stop hadoop-mapreduce-historyserver
su -l mapred -c "/usr/hdp/current/hadoop-mapreduce-historyserver/sbin/mr-jobhistory-daemon.sh --config /etc/hadoop-mapreduce-historyserver/conf stop historyserver"
```

The file storing the PID is "/var/run/hadoop-mapreduce/mapred-mapred-historyserver.pid".

    module.exports = header: 'Mapreduce Ambari JHS Stop', handler: ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Service Stop

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HISTORYSERVER'
        hostname: options.fqdn
