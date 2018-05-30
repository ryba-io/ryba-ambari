
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'Oozie Server Ambari TakeOver', handler: (options) ->
      @service.stop
        header: 'Stop'
        name: 'oozie'
      @system.remove
        header: 'Remove systemd file'
        target: '/usr/lib/systemd/system/oozie.service'
        code_skipped: 1
      @system.remove
        header: 'Remove initd file'
        target: '/etc/init.d/oozie'
        code_skipped: 1
      @system.execute
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1
