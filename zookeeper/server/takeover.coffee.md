
# HAdoop HDFS JournalNode

Wait transactions to be synced

    module.exports = header: 'Zookeeper Server Ambari TakeOver', handler: ({options}) ->
      @service.stop
        header: 'Stop'
        name: 'zookeeper-server'
      @system.remove
        header: 'Remove initd file'
        if_os: name: ['redhat','centos'], version: '7'
        target: '/usr/lib/systemd/system/zookeeper-server.service'
        code_skipped: 1
      @system.remove
        if_os: name: ['redhat','centos'], version: '6'
        header: 'Remove initd file'
        target: '/etc/init.d/zookeeper-server'
        code_skipped: 1
      @system.execute
        if_os: name: ['redhat','centos'], version: '7'
        header: 'Daemon reload'
        cmd: 'systemctl daemon-reload;systemctl reset-failed'
        code_skipped: 1
      @system.remove
        header: 'Remove old keytab'
        target: '/etc/security/keytabs/zookeeper.service.keytab'
        code_skipped: 1