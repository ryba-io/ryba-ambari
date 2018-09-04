
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'HDFS DN Ambari TakeOver', handler: ({options}) ->
      @service.stop
        header: 'Stop'
        name: 'hadoop-hdfs-datanode'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-hdfs-datanode.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-hdfs-datanode'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1
