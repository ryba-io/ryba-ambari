
# Hadoop HDFS JournalNode Stop

Stop the JournalNode service through Ambari.

TODO: put curl command

```
service hadoop-hdfs-journalnode stop
su -l hdfs -c "/usr/hdp/current/hadoop-hdfs-journalnode/../hadoop/sbin/hadoop-daemon.sh --config /etc/hadoop/conf --script hdfs stop journalnode"
```

The file storing the PID is "/var/run/hadoop-hdfs/hadoop-hdfs-journalnode.pid".

    module.exports = header: 'HDFS JN Ambari Stop', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Service Stop

      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HDFS_JOURNALNODE'
        hostname: options.fqdn


Clean up the log files related to the JournalNode

      @system.execute
        header: 'Clean Logs'
        if: options.clean_logs
        cmd: 'rm /var/log/hadoop-hdfs/*-journalnode-*'
        code_skipped: 1
