
# Hadoop ZKFC Stop

    module.exports = header: 'HDFS ZKFC Ambari Stop', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Stop

Stop the ZKFC deamon. You can also stop the server manually with one of
the following two commands:

```
service hadoop-hdfs-zkfc stop
su -l hdfs -c "/usr/hdp/current/hadoop-client/sbin/hadoop-daemon.sh --config /etc/hadoop/conf --script hdfs stop zkfc"
```

The file storing the PID is "/var/run/hadoop-hdfs/hadoop-hdfs-zkfc.pid".

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZKFC'
        hostname: options.fqdn


      @system.execute
        header: 'Clean Logs'
        if: options.clean_logs
        cmd: 'rm /var/log/hadoop-hdfs/*-zkfc-*'
        code_skipped: 1
