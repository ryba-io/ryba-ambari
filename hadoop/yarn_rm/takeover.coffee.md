
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'YARN RM Ambari TakeOver', handler: ({options}) ->
      @service.stop
        header: 'Stop'
        name: 'hadoop-yarn-resourcemanager'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/hadoop-yarn-resourcemanager.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/hadoop-yarn-resourcemanager'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1
