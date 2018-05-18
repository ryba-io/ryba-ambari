
# HAdoop HDFS zkfc

Wait transactions to be synced

    module.exports = header: 'HDFS ZKFC Ambari TakeOver', handler: (options) ->
      @service.stop
        header: 'Stop'
        name: 'hadoop-hdfs-zkfc'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-hdfs-zkfc.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-hdfs-zkfc'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload'
        code_skipped: 1
