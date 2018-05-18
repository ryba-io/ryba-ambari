
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'YARN NM Ambari TakeOver', handler: (options) ->
      @service.stop
        header: 'Stop'
        name: 'hadoop-yarn-nodemanager'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-yarn-nodemanager.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-yarn-nodemanager'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload'
        code_skipped: 1
